---
title: 巢狀虛擬化
description: 巢狀虛擬化
keywords: windows 10, hyper-v
author: johncslack
ms.date: 12/18/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: f819ac04773188525af202d370ba271a2d93e259
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998915"
---
# <a name="run-hyper-v-in-a-virtual-machine-with-nested-virtualization"></a>在巢狀虛擬化的虛擬機器中執行 Hyper-V

巢狀虛擬化功能可讓您在 Hyper-V 虛擬機器 (VM) 中執行 Hyper-V。 這對於虛擬機器中執行 Visual Studio 手機模擬器，或測試通常需要數個主機的組態很實用。

![](./media/HyperVNesting.png)

## <a name="prerequisites"></a>必要條件

* Hyper-V 主機和客體都必須是 Windows Server 2016/Windows 10 年度更新版或更新版本。
* VM 組態版本 8.0 或更新版本。
* 搭載 VT-x 與 EPT 技術的 Intel 處理器 -- 巢狀目前**僅限 Intel**。
* 第二層虛擬機器的虛擬網路有一些差異。 請參閱＜巢狀虛擬機器網路功能＞。


## <a name="configure-nested-virtualization"></a>設定巢狀虛擬化

1. 建立虛擬機器。 如需了解所需的 OS 及 VM 版本，請參閱上列必要條件。
2. 當虛擬機器為「關閉」狀態時，請對實體 HYPER-V 主機執行下列命令。 這可巢狀虛擬化虛擬機器。

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 啟動虛擬機器。
4. 一如在實體主機中，在虛擬機器上安裝 HYPER-V。 如需如何安裝 Hyper-V 的詳細資訊，請參閱 [安裝 Hyper-V](../quick-start/enable-hyper-v.md)。

## <a name="disable-nested-virtualization"></a>停用巢狀虛擬化
您可以使用下列 PowerShell 命令，停用停止使用之虛擬主機的巢狀虛擬化︰
```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## <a name="dynamic-memory-and-runtime-memory-resize"></a>調整動態記憶體與執行階段記憶體的大小
當 HYPER-V 在虛擬機器內執行時，必須先關閉虛擬機器，才能調整其記憶體。 意即，即使啟用了動態記憶體，記憶體量也不會變動。 虛擬機器如有啟用動態記憶體，任何在虛擬機器開啟時調整記憶體量的嘗試都會失敗。 

請注意，單純啟用巢狀虛擬化，對調整動態記憶體或執行階段記憶體的大小沒有影響。 僅當 HYPER-V 在 VM 中執行時，才會發生不相容的問題。

## <a name="networking-options"></a>網路連線選項

您有兩個選項可與巢狀虛擬機器連線： 

1. MAC 位址詐騙
2. NAT 網路功能

### <a name="mac-address-spoofing"></a>MAC 位址詐騙
為使網路封包能由兩部虛擬交換器進行路由，必須在虛擬交換器的第一層 (L1) 啟用改變 MAC 位址。 若要啟用此設定，請使用下列 PowerShell 命令。

``` PowerShell
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="network-address-translation-nat"></a>網路位址轉譯 (NAT)
第二個選項仰賴網路位址轉譯 (NAT)。 當無法改變 MAC 位址 (例如公用雲端環境) 時，就很適合使用此方法。

首先必須在主機虛擬機器 (即「中間」VM) 中建立虛擬 NAT 交換器。 請注意，此處的 IP 位址僅為範例，其會隨不同環境而異︰

``` PowerShell
New-VMSwitch -Name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```

接著將 IP 位址指派給網路介面卡︰

``` PowerShell
Get-NetAdapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```

每部巢狀虛擬機器都必須指派以 IP 位址與閘道。 請注意，閘道 IP 必須指向上一個步驟中的 NAT 配接器。 您也可以指派 DNS 伺服器︰

``` PowerShell
Get-NetAdapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## <a name="how-nested-virtualization-works"></a>巢狀虛擬化的運作方式

現代處理器包含可讓虛擬化更快速且更安全的硬體功能。 Hyper-V 仰賴這些處理器擴充功能來執行虛擬機器 (例如 Intel VT-x 和 AMD-V)。 一般而言，當 Hyper-V 啟動，它會防止其他軟體使用這些處理器功能。  這將導致客體虛擬電腦無法執行 Hyper-V。

巢狀虛擬化為客體虛擬電腦提供這項硬體支援。

下圖顯示非巢狀的 Hyper-V。  Hyper-V Hypervisor 會完全掌控硬體虛擬化功能 (橘色箭號)，且不會向客體作業系統公開它們。

![](./media/HVNoNesting.PNG)

另一方面，下圖顯示已啟用巢狀虛擬化的 Hyper-V。 在此情況下，Hyper-V 會向其虛擬機器公開硬體虛擬化延伸模組。 啟用巢狀後，客體虛擬機器可以安裝自己的 Hypervisor，並執行它自己的客體 VM。

![](./media/HVNesting.png)

## <a name="3rd-party-virtualization-apps"></a>協力廠商虛擬化應用程式

HYPER-V 虛擬機器不支援 HYPER-V 以外的虛擬化應用程式，而且可能會失敗。 這包括所有需要硬體虛擬化延伸模組的軟體。
