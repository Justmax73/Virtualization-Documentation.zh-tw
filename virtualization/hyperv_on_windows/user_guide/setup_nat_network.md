---
title: 設定 NAT 網路
description: 設定 NAT 網路
keywords: windows 10, hyper-v
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
---

# 設定 NAT 網路

Windows 10 Hyper-V 可促成虛擬網路的原生網路位址轉譯 (NAT)。

本指南將逐步引導您完成：
* 建立 NAT 網路
* 將現有的虛擬機器連接到新網路
* 確認虛擬機器已正確連接

需求：
* Windows 組建 14295 或更新版本
* 啟用 Hyper-V 角色 (指示在[這裡](../quick_start/walkthrough_create_vm.md))

> **注意︰**目前，Hyper-V 只允許您建立一個 NAT 網路。

## NAT 概觀
NAT 讓虛擬機器能使用主機電腦的 IP 位址和連接埠來存取網路資源。

網路位址轉譯 (NAT) 是設計來節省 IP 位址的網路模式，它會將外部 IP 位址和連接埠對應至較大集合的內部 IP 位址。  基本上，NAT 交換器使用 NAT 對應資料表將來自某個 IP 位址和連接埠號碼的流量，路由傳送到與網路上的裝置 (虛擬機器、電腦、容器等) 相關聯的正確內部 IP 位址。

此外，NAT 還可藉由將通訊連接埠與唯一外部連接埠對應，以讓多部虛擬機器裝載需要相同 (內部) 通訊連接埠的應用程式。

由於所有這些理由，NAT 網路是非常普遍的容器技術 (請參閱[容器的網路功能](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking))。


## 建立 NAT 虛擬網路
讓我們逐步解說如何設定新的 NAT 網路。

1.  以系統管理員身分開啟 PowerShell 主控台。  

2. 建立內部交換器  

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. 使用 [New-NetIPAddress](https://technet.microsoft.com/en-us/library/hh826150.aspx) 設定 NAT 閘道。  

  下面是一般的命令︰
  ``` PowerShell
  New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
  ```

  若要設定閘道，您將需要一些網路的相關資訊︰  
  * **IPAddress** - NAT 閘道 IP 指定要作為 NAT 閘道 IP 的 IPv4 或 IPv6 位址。  
    一般格式為 a.b.c.1 (例如 172.16.0.1)。  最後一個位置不一定要是 .1，不過通常是如此 (根據首碼長度)

    常見的閘道 IP 是 192.168.0.1  

  * **PrefixLength** - NAT 子網路首碼長度定義了 NAT 本機子網路的大小 (子網路遮罩)。
    子網路首碼長度是介於 0 到 32 之間的整數值。

    0 會對應整個網際網路，32 則只允許一個對應的 IP。  常見的值範圍從 24 到 12，視多少 IP 必須附加至 NAT 而定。

    常見的 PrefixLength 是 24 - 這是子網路遮罩 255.255.255.0

  * **InterfaceIndex** - 如果索引是上面所建立之虛擬交換器的介面索引。

    您可以藉由執行下列命令找到介面索引： `Get-NetAdapter`

    您的輸出看起來應該像這樣︰

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    內部交換器會有類似 `vEthernet (SwitchName)` 的名稱，和介面描述 `Hyper-V Virtual Ethernet Adapter`。

  請執行下列命令，以建立 NAT 閘道：

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

4. 使用 [New-NetNat](https://technet.microsoft.com/en-us/library/dn283361(v=wps.630).aspx) 設定 NAT 網路。  

  下面是一般的命令︰

  ``` PowerShell
  New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
  ```

  若要設定閘道，您將需要提供網路和 NAT 閘道的相關資訊︰  
  * **名稱** - NATOutsideName 描述 NAT 網路的名稱。  您將使用它來移除 NAT 網路。

  * **InternalIPInterfaceAddressPrefix** - NAT 子網路首碼說明上述的 NAT 閘道 IP 首碼，以及上述的 NAT 子網路首碼長度。

    一般格式將為 a.b.c.0/NAT 子網路首碼長度

    從上述說明，在本例中，我們將使用 192.168.0.0/24

  針對我們的範例，執行下列命令以設定 NAT 網路︰

  ``` PowerShell
  New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
  ```

恭喜！  您現在有了虛擬 NAT 網路了！  若要新增虛擬機器至 NAT 網路，請遵循[這些指示](setup_nat_network.md#connect-a-virtual-machine)。

## 連接虛擬機器

若要將虛擬機器連接到新的 NAT 網路，請使用 [VM 設定] 功能表將您在 [NAT 網路設定](setup_nat_network.md#create-a-nat-virtual-network)一節的第一個步驟中建立的內部交換器，連接到您的虛擬機器。


## 疑難排解

此工作流程假設主機上沒有其他 NAT。 不過，有時候多個應用程式或服務都必須使用 NAT。 由於 Windows (WinNAT) 只支援一個內部 NAT 子網路首碼，嘗試建立多個 NAT 會讓系統進入不明的狀態。

### 疑難排解步驟
1. 確定您只有一個 NAT

  ``` PowerShell
  Get-NetNat
  ```
2. 如果 NAT 已經存在，請刪除它

  ``` PowerShell
  Get-NetNat | Remove-NetNat
  ```

3. 確定您只有一個 NAT 的「內部」vmSwitch。 記錄 vSwitch 名稱以便進行步驟 4

  ``` PowerShell
  Get-VMSwitch
  ```

4. 檢查是否有來自舊 NAT 的私人 IP 位址 (例如 NAT 預設閘道 IP 位址 - 通常是 *.1) 仍指派給配接器

  ``` PowerShell
  Get-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)"
  ```

5. 如果舊的私人 IP 位址正在使用中，請將它刪除  
   ``` PowerShell
  Remove-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)" -IPAddress <IPAddress>
  ```

## 多個應用程式使用相同的 NAT

某些情況下需要多個應用程式或服務使用相同的 NAT。 在此情況下，必須遵循下列工作流程，以便多個應用程式/服務可以使用較大的 NAT 內部子網路首碼

**_作為範例，我們會詳細說明與 Windows 容器功能共置在相同主機上的現有 Docker 4 Windows - Docker Beta 版 - Linux VM。 此工作流程可能有所變更_**

1. C:\> net stop docker
2. 停止 Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *移除任何先前已存在的容器網路 (亦即刪除 vSwitch、刪除 NetNat、清除)*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (此子網路將用於 Windows 容器功能) *建立名為 nat 的內部 vSwitch*  
   *建立名為 “nat” 且 IP 首碼為 10.0.76.0/24 的 NAT 網路*  

6. Remove-NetNAT  
   *移除 DockerNAT 和 nat NAT 網路 (保留內部 vSwitch)*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (這會建立一個較大的 NAT 網路讓 D4W 和容器共用)  
   *建立名為 DockerNAT 且具有較大首碼 10.0.0.0/17 的 NAT 網路*  

8. 執行 Docker4Windows (MobyLinux.ps1)  
   *建立內部 vSwitch DockerNAT*  
   *建立名為 “DockerNAT” 且 IP 首碼為 10.0.75.0/24 的 NAT 網路*  

9. Net start docker  
   *Docker 將以使用者定義的 NAT 網路作為預設值來連接 Windows 容器*  

最後，您應該有兩個內部 vSwitch – 一個名為 DockerNAT，另一個名為 nat。 您只能只會有一個透過執行 Get-NetNat 確認的 NAT 網路 (10.0.0.0/17)。 Windows 主機網路服務 (HNS) 會從 10.0.76.0/24 子網路指派 Windows 容器的 IP 位址。 根據現有的 MobyLinux.ps1 指令碼，將會從 10.0.75.0/24 子網路指派 Docker 4 Windows 的 IP 位址。


## 參考
深入了解 [NAT 網路](https://en.wikipedia.org/wiki/Network_address_translation)


<!--HONumber=May16_HO5-->


