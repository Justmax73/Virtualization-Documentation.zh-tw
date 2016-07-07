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
ms.sourcegitcommit: 26a8adb426a7cf859e1a9813da2033e145ead965
ms.openlocfilehash: d17413fc572e59ec21ff513ef5de994c6716aa08

---

# 在巢狀虛擬化的虛擬機器中執行 Hyper-V

巢狀虛擬化功能可讓在 Hyper-V 虛擬機器中執行 Hyper-V。 換言之，使用巢狀虛擬化，Hyper-V 主機本身也能虛擬化。 舉例來說，巢狀虛擬化可讓您在虛擬化的容器主機中執行 Hyper-V 容器；在虛擬化環境中設定 HYPER-V 實驗室；或無須配備個別的硬體，就能測試多部機器。 本文件詳細說明軟硬體先決條件、設定步驟，以及如何疑難排解問題。 如果您是在 Windows Insider Preview 組建 14361 或更新版本上執行 Hyper-V，請參閱 [Nested Virtualization Preview for Windows Insiders: Builds 14361+](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting#nested-virtualization-preview-for-windows-insiders-builds-14361-) (適用於 Windows 測試人員的巢狀虛擬化：組建 14361+)。

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

##適用於 Windows 測試人員的巢狀虛擬化：組建 14361+
幾個月前，我們公佈了 Hyper-V 巢狀虛擬化與組建 10565 的預先預覽版本。 我們很高興這項功能引起眾人的高度期待，並很榮幸能與 Windows 測試人員分享相關更新。

###需使用新的 VM 版本以支援巢狀虛擬化
從組建 14361 開始，啟用巢狀虛擬化的 VM 需為 8.0 版。 因此，如果是在舊版主機上建立並啟用巢狀功能的 VM，就需要更新版本。 

####更新 VM 版本
若要繼續使用巢狀虛擬化，您需要將 VM 版本更新至 8.0 版。 這表示您必須移除儲存狀態，並將 VM 關閉。 下列 PowerShell Cmdlet 可更新 VM 版本︰
```none
Update-VMVersion -Name <VMName>
```
####停用巢狀虛擬化
如果您不想更新 VM，可以停用巢狀虛擬化，以讓 VM 開機︰
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

###VM 8.0 版的新行為 
在此預覽版本中，已啟用巢狀功能之 VM 的運作方式有下列幾點變更︰
-   現在，您可針對已啟用巢狀虛擬化的 VM，建立及套用檢查點。
-   您現在可以儲存並啟動啟用巢狀功能的 VM。
-   啟用巢狀虛擬化的 VM 現可在啟用虛擬式安全性的主機上執行 (包括 Device Guard 和 Credential Guard)。
-   我們已改善現有限制的錯誤訊息。

###功能限制
-   巢狀虛擬化功能是專為在 Hyper-V 虛擬機器內執行 Hyper-V 所設計。 目前不支援協力廠商虛擬化應用程式，因此在 Hyper-V VM 中可能無法使用這些應用程式。
-   動態記憶體與巢狀虛擬化不相容。 當 Hyper-V 在 VM 內部執行時，VM 就無法變更它的執行階段記憶體。 
-   執行階段記憶體大小調整與巢狀虛擬化不相容。 當 Hyper-V 在 VM 內部執行時，VM 的記憶體大小調整就會失敗。 
-   僅有 Intel 系統支援巢狀虛擬化。

###已知問題
組建 14361 中有下列已知問題：第 2 代 VM 無法開機，並發生如下錯誤︰
```none
“Cannot modify property without enabling VirtualizationBasedSecurityOptOut”
```
您可以停用巢狀虛擬化，或選擇取消虛擬式安全性，以暫時修正此問題︰
```none
Set-VMSecurity -VMName <vmname> -VirtualizationBasedSecurityOptOut $true
```

###我們很重視您的意見
歡迎您繼續使用 Windows 意見反應應用程式傳送意見給我們。 如有任何問題，請在我們的說明文件 [GitHub](https://github.com/Microsoft/Virtualization-Documentation) 頁面上提出問題。 



<!--HONumber=Jul16_HO1-->


