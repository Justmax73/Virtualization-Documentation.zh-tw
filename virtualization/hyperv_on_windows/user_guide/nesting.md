---
title: 巢狀虛擬化
description: 巢狀虛擬化
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
---

# 在巢狀虛擬化的虛擬機器中執行 Hyper-V

巢狀虛擬化功能可讓在 Hyper-V 虛擬機器中執行 Hyper-V。 換言之，使用巢狀虛擬化，Hyper-V 主機本身也能虛擬化。 舉例來說，巢狀虛擬化可讓您在虛擬化的容器主機中執行 Hyper-V 容器；在虛擬化環境中設定 HYPER-V 實驗室；或無須配備個別的硬體，就能測試多部機器。 本文件詳細說明軟硬體先決條件、設定步驟，以及如何疑難排解問題。

## 先決條件

- 執行組建 10565 或更新版本的 Windows 測試人員組建 (Windows Server 2016 或 Windows 10)。
- 兩個 Hypervisor (父項與子項) 都必須執行同一版 Windows 組建 (10565 或更新版本)。
- 搭載 Intel VT-x 與 EPT 技術的 Intel 處理器。

## 設定巢狀虛擬化

首先須建立虛擬機器主機。**請勿開啟虛擬機器**。 如需詳細資訊，請參閱[建立虛擬機器](../quick_start/walkthrough_create_vm.md)。

建立好虛擬機器之後，請在實體 Hypervisor 主機上執行下列命令。 這可啟用虛擬機器上的巢狀虛擬化。

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
執行巢狀 HYPER-V 主機時，必須停用虛擬機器上的動態記憶體。 您可以透過虛擬機器的屬性加以設定，也可以使用下列 PowerShell 命令來設定。
```none
Set-VMMemory -VMName <VMName> -DynamicMemoryEnabled $false
```

完成這些步驟後，即可啟動虛擬機器，Hyper-V 也完成安裝。 如需如何安裝 Hyper-V 的詳細資訊，請參閱 [安裝 Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)。

## 網路連線選項
您有兩個選項可與巢狀虛擬機器連線，一是改變 MAC 位址，一是使用 NAT 模式。

### 改變 MAC 位址
為使網路封包能由兩部虛擬交換器進行路由，必須在虛擬交換器的第一層啟用改變 MAC 位址。 若要啟用此設定，請使用下列 PowerShell 命令。

```none
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### 網路位址轉譯
第二個選項仰賴網路位址轉譯 (NAT)。 當無法改變 MAC 位址 (例如公用雲端環境) 時，就很適合使用此方法。

首先必須在主機虛擬機器 (即「中間」VM) 中建立虛擬 NAT 交換器。 請注意，此處的 IP 位址僅為範例，其會隨不同環境而異︰
```none
new-vmswitch -name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```
接著將 IP 位址指派給網路介面卡︰
```none
get-netadapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```
每部巢狀虛擬機器都必須指派以 IP 位址與閘道。 請注意，閘道 IP 必須指向上一個步驟中的 NAT 配接器。 您也可以指派 DNS 伺服器︰
```none
get-netadapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```


## 已知問題

- 啟用 Device Guard 的主機無法向客體公開虛擬化延伸模組。
- 啟用虛擬式安全性 (VBS) 的虛擬機器無法同時啟用巢狀功能。 若要使用巢狀虛擬化，必須先停用 VBS。
- 虛擬機器一旦啟用巢狀虛擬化，下列功能將不再與該 VM 相容。  
  * 調整執行階段記憶體與動態記憶體的大小
  * 檢查點
  * 啟用 Hyper-V 的虛擬機器無法即時移轉。

## 常見問題和疑難排解

我的虛擬機器無法啟動，我該怎麼做？

1. 請確認動態記憶體已「關閉」。
2. 請確定您的 Intel 處理器等級夠快。
3. 在主機電腦上，從提高權限的提示中，執行[此 PowerShell 指令碼](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1)。

## 意見反應

您可以利用 Windows 意見反應應用程式、[虛擬化論壇](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv)或 [GitHub](https://github.com/Microsoft/Virtualization-Documentation) 回報其他問題。


<!--HONumber=Jun16_HO3-->


