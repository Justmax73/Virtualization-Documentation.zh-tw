---
title: "巢狀虛擬化"
description: "巢狀虛擬化"
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
translationtype: Human Translation
ms.sourcegitcommit: 4eb3e733990ca86f28620e23bf408ff71b879173
ms.openlocfilehash: c94b7f2c9c90eaf2834a1d0c54fb1f9b2a8f9669

---

# 在巢狀虛擬化的虛擬機器中執行 Hyper-V

巢狀虛擬化功能可讓在 Hyper-V 虛擬機器中執行 Hyper-V。 換言之，使用巢狀虛擬化，Hyper-V 主機本身也能虛擬化。 舉例來說，巢狀虛擬化可讓您在虛擬化的容器主機中執行 Hyper-V 容器；在虛擬化環境中設定 HYPER-V 實驗室；或無須配備個別的硬體，就能測試多部機器。 本文將詳細說明軟體與硬體的必要條件、設定步驟及限制。 

## 必要條件

- 執行 Windows Server 2016 或 Windows 10 年度更新版的 HYPER-V 主機。
- 執行 Windows Server 2016 或 Windows 10 年度更新版的 HYPER-V VM。
- 設定了 8.0 版或更新版本的 HYPER-V VM。
- 搭載 Intel VT-x 與 EPT 技術的 Intel 處理器。

## 設定巢狀虛擬化

1. 建立虛擬機器。 如需了解所需的 OS 及 VM 版本，請參閱上列必要條件。
2. 當虛擬機器為「關閉」狀態時，請對實體 HYPER-V 主機執行下列命令。 這可巢狀虛擬化虛擬機器。
```none
    Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 啟動虛擬機器。
4. 一如在實體主機中，在虛擬機器上安裝 HYPER-V。 如需如何安裝 Hyper-V 的詳細資訊，請參閱 [安裝 Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)。

## 停用巢狀虛擬化
您可以使用下列 PowerShell 命令，停用停止使用之虛擬主機的巢狀虛擬化︰
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## 調整動態記憶體與執行階段記憶體的大小
當 HYPER-V 在虛擬機器內執行時，必須先關閉虛擬機器，才能調整其記憶體。 意即，即使啟用了動態記憶體，記憶體量也不會變動。 虛擬機器如有啟用動態記憶體，任何在虛擬機器開啟時調整記憶體量的嘗試都會失敗。 

請注意，單純啟用巢狀虛擬化，對調整動態記憶體或執行階段記憶體的大小沒有影響。 僅當 HYPER-V 在 VM 中執行時，才會發生不相容的問題。

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

## 第三方虛擬化 App
HYPER-V 虛擬機器不支援 HYPER-V 以外的虛擬化應用程式，而且可能會失敗。 這包括所有需要硬體虛擬化延伸模組的軟體。



<!--HONumber=Sep16_HO3-->


