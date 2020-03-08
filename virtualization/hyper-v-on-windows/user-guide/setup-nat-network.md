---
title: 設定 NAT 網路
description: 設定 NAT 網路
keywords: windows 10, hyper-v
author: jmesser81
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
ms.openlocfilehash: 1652c3bcb32ddbc4e05e8821d0e646a76a2fd4f0
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853962"
---
# <a name="set-up-a-nat-network"></a>設定 NAT 網路

Windows 10 Hyper-V 可促成虛擬網路的原生網路位址轉譯 (NAT)。

本指南將逐步引導您完成：
* 建立 NAT 網路
* 將現有的虛擬機器連接到新網路
* 確認虛擬機器已正確連接

需求：
* Windows 10 年度更新版或以上版本
* 已啟用 Hyper-V (指示在[這裡](../quick-start/enable-hyper-v.md))

> **注意︰** 目前，每個主機只能建立一個 NAT 網路。 如需 Windows NAT (WinNAT) 實作、功能及限制的其他詳細資料，請參閱 [WinNAT 功能與限制部落格](https://techcommunity.microsoft.com/t5/Virtualization/Windows-NAT-WinNAT-Capabilities-and-limitations/ba-p/382303)

## <a name="nat-overview"></a>NAT 概觀
NAT 讓虛擬機器透過內部的 Hyper-V 虛擬交換器，使用主機電腦的 IP 位址和連接埠存取網路資源。

網路位址轉譯 (NAT) 是設計來節省 IP 位址的網路模式，它會將外部 IP 位址和連接埠對應至較大集合的內部 IP 位址。  基本上，NAT 使用流程表將來自外部 (主機) IP 位址和連接埠號碼的流量，路由傳送到已與網路上的端點 (虛擬機器、電腦、容器等) 建立關聯的正確內部 IP 位址。

此外，NAT 還可藉由將通訊連接埠與唯一外部連接埠對應，以讓多部虛擬機器裝載需要相同 (內部) 通訊連接埠的應用程式。

由於所有這些理由，NAT 網路是非常普遍的容器技術 (請參閱[容器的網路功能](https://docs.microsoft.com/virtualization/windowscontainers/container-networking/architecture))。


## <a name="create-a-nat-virtual-network"></a>建立 NAT 虛擬網路
讓我們逐步解說如何設定新的 NAT 網路。

1. 以系統管理員身分開啟 PowerShell 主控台。  

2. 建立內部交換器。

    ```powershell
    New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
    ```

3. 尋找您剛才建立之虛擬交換器的介面索引。

    您可以藉由執行 `Get-NetAdapter` 來尋找介面索引

    您的輸出看起來應該像這樣︰

    ```console
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    內部交換器會有類似 `vEthernet (SwitchName)` 的名稱，和介面描述 `Hyper-V Virtual Ethernet Adapter`。 記下其 `ifIndex` 以用於下一個步驟。

4. 使用 [New-NetIPAddress](https://docs.microsoft.com/powershell/module/nettcpip/New-NetIPAddress) 設定 NAT 閘道。  

    下面是一般的命令︰
    ```powershell
    New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
    ```

    若要設定閘道，您將需要一些網路的相關資訊︰  
    * **IPAddress** - NAT 閘道 IP 指定要做為 NAT 閘道 IP 的 IPv4 或 IPv6 位址。  
      一般格式為 a.b.c.1 (例如 172.16.0.1)。  最後一個位置不一定要是 .1，不過通常是如此 (根據首碼長度)

      常見的閘道 IP 是 192.168.0.1  

    * **PrefixLength** -Nat 子網前置長度會定義 nat 區域子網大小（子網路遮罩）。
      子網路首碼長度是介於 0 到 32 之間的整數值。

      0 會對應整個網際網路，32 則只允許一個對應的 IP。  常見的值範圍從 24 到 12，視多少 IP 必須附加至 NAT 而定。

      常見的 PrefixLength 是 24 - 這是子網路遮罩 255.255.255.0

    * **InterfaceIndex** -- ifIndex 是虛擬交換器的介面索引 (您已在上一步中判斷出)。

    請執行下列命令，以建立 NAT 閘道：

    ```powershell
    New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
    ```

5. 使用 [New-NetNat](https://docs.microsoft.com/powershell/module/netnat/New-NetNat) 設定 NAT 網路。  

    下面是一般的命令︰

    ```powershell
    New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
    ```

    若要設定閘道，您將需要提供網路和 NAT 閘道的相關資訊︰  
    * **名稱** - NATOutsideName 描述 NAT 網路的名稱。  您將使用它來移除 NAT 網路。

    * **InternalIPInterfaceAddressPrefix** - NAT 子網路首碼說明上述的 NAT 閘道 IP 首碼，以及上述的 NAT 子網路首碼長度。

    一般格式將為 a.b.c.0/NAT 子網路首碼長度

    從上述說明，在本例中，我們將使用 192.168.0.0/24

    針對我們的範例，執行下列命令以設定 NAT 網路︰

    ```powershell
    New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
    ```

恭喜！  您現在有了虛擬 NAT 網路了！  若要新增虛擬機器至 NAT 網路，請遵循[這些指示](#connect-a-virtual-machine)。

## <a name="connect-a-virtual-machine"></a>連接虛擬機器

若要將虛擬機器連接到新的 NAT 網路，請使用 \[VM 設定\] 功能表將您在 [NAT 網路設定](#create-a-nat-virtual-network)一節的第一個步驟中建立的內部交換器，連接到您的虛擬機器。

因為 WinNAT 本身不會將 IP 位址配置和指派到端點 (例如 VM)，所以您必須從 VM 內手動執行此操作，也就是在 NAT 內部首碼的範圍內設定 IP 位址、設定預設閘道 IP 位址，以及設定 DNS 伺服器資訊。 唯一要注意的是端點連結到容器的時機。 在本例中，主機網路服務 (HNS) 會配置並使用主機運算服務 (HCS) 直接將 IP 位址、閘道 IP 和 DNS 資訊指派給容器。


## <a name="configuration-example-attaching-vms-and-containers-to-a-nat-network"></a>設定範例︰將 VM 和容器連結到 NAT 網路

_如果您需要將多個 Vm 和容器連結到單一 NAT，您必須確定 NAT 內部子網首碼夠大，足以包含由不同應用程式或服務所指派的 IP 範圍（例如適用於 Windows 的 Docker 和 Windows 容器）。HNS）。這將需要一個應用層級的 Ip 指派和網路設定，或手動設定，這必須由系統管理員完成，並保證不會在相同的主機上重複使用現有的 IP 指派。_

### <a name="docker-for-windows-linux-vm-and-windows-containers"></a>Docker for Windows (Linux VM) 和 Windows Containers
下方的解決方案會讓 Docker for Windows (執行 Linux 容器的 Linux VM) 和 Windows Containers 使用不同的內部 vSwitch，共用同一個 WinNAT 執行個體。 Linux 和 Windows 容器間的連線都會正常運作。

使用者已透過名為 “VMNAT” 的內部 vSwitch 將 VM 連線到 NAT 網路，現在想要安裝具有 docker 引擎的 Windows Container 功能
```
PS C:\> Get-NetNat “VMNAT”| Remove-NetNat (this will remove the NAT but keep the internal vSwitch).
Install Windows Container Feature
DO NOT START Docker Service (daemon)
Edit the arguments passed to the docker daemon (dockerd) by adding –fixed-cidr=<container prefix> parameter. This tells docker to create a default nat network with the IP subnet <container prefix> (e.g. 192.168.1.0/24) so that HNS can allocate IPs from this prefix.
PS C:\> Start-Service Docker; Stop-Service Docker
PS C:\> Get-NetNat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> Start-Service docker
```
Docker/HNS 會將 Ip 指派給 Windows 容器，而系統管理員會將 Ip 指派給兩者的差異集合中的 Vm。

使用者已安裝執行 docker 引擎的 Windows Container 功能，現在想要將 VM 連線到 NAT 網路
```
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
PS C:\> Get-NetNat | Remove-NetNat (this will remove the NAT but keep the internal vSwitch)
Edit the arguments passed to the docker daemon (dockerd) by adding -b “none” option to the end of docker daemon (dockerd) command to tell docker not to create a default NAT network.
PS C:\> New-ContainerNetwork –name nat –Mode NAT –subnetprefix <container prefix> (create a new NAT and internal vSwitch – HNS will allocate IPs to container endpoints attached to this network from the <container prefix>)
PS C:\> Get-Netnat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> New-VirtualSwitch -Type internal (attach VMs to this new vSwitch)
PS C:\> Start-Service docker
```
Docker/HNS 會將 Ip 指派給 Windows 容器，而系統管理員會將 Ip 指派給兩者的差異集合中的 Vm。

最後，您應該要有兩個內部 VM 交換器，而且兩者共用一個 NetNat。

## <a name="multiple-applications-using-the-same-nat"></a>多個應用程式使用相同的 NAT

某些情況下需要多個應用程式或服務使用相同的 NAT。 在此情況下，必須遵循下列工作流程，以便多個應用程式/服務可以使用較大的 NAT 內部子網路首碼

**_我們將在相同的主機上，將 Docker 4 Windows-Docker Beta-Linux VM 與 Windows 容器功能詳細說明。此工作流程可能會變更_**

1. C:\> net stop docker
2. 停止 Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *移除任何先前現有的容器網路（亦即刪除 vSwitch、刪除 NetNat、清除）*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (此子網路將用於 Windows 容器功能) *建立名為 nat 的內部 vSwitch*  
   *建立名為 "nat" 的 NAT 網路，IP 首碼為 10.0.76.0/24*  

6. Remove-NetNAT  
   *移除 DockerNAT 和 nat NAT 網路（保留內部 Vswitch）*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (這會建立一個較大的 NAT 網路讓 D4W 和容器共用)  
   *建立名為 DockerNAT 且具有較大首碼 10.0.0.0/17 的 NAT 網路*  

8. 執行 Docker4Windows (MobyLinux.ps1)  
   *建立內部 vSwitch DockerNAT*  
   *建立名為 "DockerNAT" 的 NAT 網路，IP 首碼為 10.0.75.0/24*  

9. Net start docker  
   *Docker 會使用使用者定義的 NAT 網路做為預設值來連接 Windows 容器*  

最後，您應該有兩個內部 vSwitch – 一個名為 DockerNAT，另一個名為 nat。 您只能只會有一個透過執行 Get-NetNat 確認的 NAT 網路 (10.0.0.0/17)。 Windows 主機網路服務 (HNS) 會從 10.0.76.0/24 子網路指派 Windows 容器的 IP 位址。 根據現有的 MobyLinux.ps1 指令碼，將會從 10.0.75.0/24 子網路指派 Docker 4 Windows 的 IP 位址。


## <a name="troubleshooting"></a>疑難排解

### <a name="multiple-nat-networks-are-not-supported"></a>不支援多個 NAT 網路  
本指南假設主機上沒有其他 NAT。 不過，應用程式或服務將會需要使用 NAT，而且可能會在安裝程序中建立。 由於 Windows (WinNAT) 只支援一個內部 NAT 子網路首碼，嘗試建立多個 NAT 會讓系統進入不明的狀態。

若要看看這是否是問題，請確定您只有一個 NAT：
```powershell
Get-NetNat
```

如果 NAT 已經存在，請刪除它
```powershell
Get-NetNat | Remove-NetNat
```
請確定您只有一個「內部」vmSwitch 供應用程式或功能使用 (例如 Windows 容器)。 記錄 vSwitch 的名稱
```powershell
Get-VMSwitch
```

請檢查是否有私人 IP 位址（例如 NAT 預設閘道 IP 位址-通常是_x_）。_y_。_z_.1），從仍然指派給介面卡的舊 NAT
```powershell
Get-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)"
```

如果舊的私人 IP 位址正在使用中，請將它刪除
```powershell
Remove-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)" -IPAddress <IPAddress>
```

**移除多個 Nat**  
我們看到報告顯示意外建立了多個 NAT 網路。 這是近期組建的 Bug 所造成 (包括 Windows Server 2016 Technical Preview 5 和 Windows 10 Insider Preview 組建)。 如果您在執行 docker 網路 ls 或 Get-ContainerNetwork 之後，看到多個 NAT 網路，請從提高權限的 PowerShell 執行下列作業︰

```powershell
$keys = Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
Remove-NetNat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```

執行後續的命令之前，請先重新開機作業系統（`Restart-Computer`）

```powershell
Get-NetNat | Remove-NetNat
Set-Service docker -StartupType Automatic
Start-Service docker 
```

請參閱[使用相同 NAT 的多個應用程式設定指南](#multiple-applications-using-the-same-nat)，視需要重建 NAT 環境。 

## <a name="references"></a>參考
深入了解 [NAT 網路](https://en.wikipedia.org/wiki/Network_address_translation)
