---
title: "Windows 容器的網路功能"
description: "設定 Windows 容器的網路功能。"
keywords: "docker, 容器"
author: jmesser81
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: 7b5cf299109a967b7e6aac839476d95c625479cd
ms.openlocfilehash: 2e26177f3e653e9102dc91070b987e28ef713bed

---

# 容器的網路功能

Windows 容器在網路功能方面類似於虛擬機器。 每個容器都有一個連線到虛擬交換器的虛擬網路介面卡，用以轉送輸入和輸出流量。 為了強制執行相同主機上的容器隔離，系統會為每個 Windows Server 和 Hyper-V 容器 (其中已針對容器安裝網路介面卡) 建立網路區間。 Windows Server 容器使用主機 vNIC 連結到虛擬交換器。 Hyper-V 容器使用綜合 VM NIC (不向公用程式 VM 公開) 連結到虛擬交換器。

Windows 容器支援四種不同的網路驅動程式或模式︰*nat*、*transparent**l2bridge* 和 *l2tunnel*。 根據您的實體網路基礎結構和單一與多主機網路功能需求，您應該選擇最符合需求的網路模式。 

Docker 引擎依預設會在 dockerd 服務第一次執行時建立 NAT 網路。 建立的預設內部 IP 首碼是 172.16.0.0/12。 

> 注意︰如果容器主機 IP 也是這個相同首碼，您必須變更內部 NAT IP 首碼，如下所述

容器端點會附加至這個預設網路，並從內部首碼指派一個 IP 位址。 Windows 中目前只支援一個 NAT 網路 (但擱置的[提取要求](https://github.com/docker/docker/pull/25097)可能有助於解決此限制)。 

可以在相同的容器主機上建立其他使用不同驅動程式 (例如 transparent、l2bridge) 的網路。 下表顯示如何提供每個模式的內部 (容器對容器) 和外部連接的網路連線。

- **網路位址轉譯** – 每個容器將會從內部、私用 IP 首碼 (例如 172.16.0.0/12) 收到 IP 位址。 支援連接埠轉送/從容器主機到容器端點的對應

- **透明** – 每個容器端點直接連接至實體網路。 可以使用外部 DHCP 伺服器，以靜態或動態方式指派來自實體網路的 IP。

- **L2 橋接器** - 每個容器端點會在與容器主機相同的 IP 子網路中。 必須以靜態方式從與容器主機相同的首碼指派 IP 位址。 因為第 2 層位址轉譯的關係，主機上的所有容器端點會具有相同的 MAC 位址。

- **L2 通道** - _ 這種模式應該僅限用於 Microsoft 雲端堆疊_

> 若要了解如何將容器端點連接至具有 Microsoft SDN 堆疊的重疊虛擬網路，請參閱[將容器連接至虛擬網路](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)主題。

## 單一節點

|  | 容器對容器 | 容器外部 |
| :---: | :---------------     |  :---                |
| NAT | 透過 Hyper-V 虛擬交換器橋接的連線 | 透過 WinNAT 路由並套用位址轉譯 | 
| 透明 | 透過 Hyper-V 虛擬交換器橋接的連線 | 直接存取實體網路 | 
| l2bridge | 透過 Hyper-V 虛擬交換器橋接的連線|  存取實體網路並具有 MAC 位址轉譯|  



## 多節點

|  | 容器對容器 | 容器外部 |
| :---: | :----       | :---------- |
| NAT | 必須參考外部容器主機 IP 和連接埠；透過 WinNAT 路由並套用位址轉譯 | 必須參考外部主機；透過 WinNAT 路由並套用位址轉譯 | 
| 透明 | 必須直接參考容器 IP 端點 | 直接存取實體網路 | 
| l2bridge | 必須直接參考容器 IP 端點| 存取實體網路並具有 MAC 位址轉譯| 


## 建立網路 

### (預設) NAT 網路

Windows Docker 引擎會建立預設的 'nat' 網路，且使用 IP 首碼 172.16.0.0/12。 在 Windows 容器主機上目前只允許一個 NAT 網路。 如果使用者想要使用特定的 IP 首碼來建立 NAT 網路，他們可以變更 Docker 組態 daemon.json 檔中的選項 (位於 C:\ProgramData\Docker\config\daemon.json - 若不存在則加以建立)，執行下列兩件事之一。
 1. 使用 _"fixed-cidr": "<IP 首碼> / Mask"_ 選項，這會使用指定的 IP 首碼和遮罩，建立預設的 NAT 網路。
 2. 使用 _"bridge":"none"_ 選項，這不會建立預設網路；使用者可以使用 *docker network create -d <driver>* 命令，以任何驅動程式建立使用者定義的網路。

在執行這些組態選項的其中一者之前，必須先停止 Docker 服務，並刪除任何預先存在的 NAT 網路。

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

如果使用者在 daemon.json 檔案中加入 "fixed-cidr" 選項，Docker 引擎現在將使用指定的自訂 IP 首碼和遮罩，建立使用者定義的 NAT 網路。 如果使用者新增 "bridge:none" 選項，則他們必須以手動方式建立網路。

```none
# Create a user-defined nat network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

依預設，容器端點將會連線到預設 NAT 網路。 如果未建立預設 NAT 網路 (因為 daemon.json 裡指定了 "bridge:none") 或需要存取不同的使用者定義網路，則使用者可以在 docker run 命令中指定 *--network* 參數。

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### 連接埠對應

當容器連接至 NAT 網路時，若要存取在其中執行的應用程式，需要在容器主機和容器端點之間建立連接埠對應。 這些對應必須在建立容器時，或容器處於已停止狀態時指定。

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Create a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

也支援使用 -p 參數或在 Dockerfile 中使用 EXPOSE 命令搭配 -p 參數，進行動態連接埠對應。 容器主機上會隨機選擇暫時的連接埠，並可在執行 Docker ps 時進行檢查。

```none
C:\> docker run -itd -p 80 windowsservercore cmd

# Network services running on port TCP:80 in this container can be accessed externally on port TCP:14824
C:\> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
bbf72109b1fc        windowsservercore   "cmd"               6 seconds ago       Up 2 seconds        *0.0.0.0:14824->80/tcp*   drunk_stonebraker

# Container image specified EXPOSE 80 in Dockerfile - publish this port mapping
C:\> docker network 
```
> 從 WS2016 TP5 和大於 14300 的 Windows 測試人員組建開始，將會自動為所有 NAT 連接埠對應建立防火牆規則。 此防火牆規則對容器主機來說是通用的，亦不會針對特定容器端點或網路介面卡進行當地語系化。

Windows NAT (WinNAT) 實作具有一些功能問題，此部落格文章中討論：[WinNAT 功能和限制](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/) 
 1. 每個容器主機只支援一個 NAT 網路 (一個內部 IP 首碼)
 2. 使用內部 IP 和連接埠，才能從容器端點連線到容器主機。

可以使用不同的驅動程式建立額外的網路。 

> Docker 網路驅動程式全為小寫。

### 透明網路

若要使用廣域網路模式，請使用驅動程式名稱 'transparent' 建立容器網路。 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```

如果容器主機已虛擬化，而且您想要使用 DHCP 進行 IP 指派，則必須在虛擬機器網路介面卡上啟用 MACAddressSpoofing。 否則，Hyper-V 主機將會封鎖來自 VM 中具有多個 MAC 位址之容器的網路流量。

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> 若要建立多個透明網路，您必須指定外部 HYPER-V 虛擬交換器 (自動建立) 應該繫結的 (虛擬) 網路介面卡。

若要將網路 (已透過 Hyper-V 虛擬交換器連接) 繫結至特定網路介面，請使用選項 *-o com.docker.network.windowsshim.interface=<Interface>*

```none
# Create a transparent network which is attached to the "Ethernet 2" network interface
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

*com.docker.network.windowsshim.interface* 的值是配接器的*名稱*，來自︰ 
```none
Get-NetAdapter
```

連線到透明網路的容器端點 IP 位址可以是靜態指派，或是從外部 DHCP 伺服器動態指派。

使用靜態 IP 指派時，您必須先確認建立網路時指定了 *--subnet* 和 *--gateway* 參數。 子網路和閘道 IP 位址應該與容器主機的網路設定相同 - 也就是和實體網路相同。

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
使用 docker run 命令的 *--ip* 選項指定 IP 位址

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> 確定此 IP 位址未指派給實體網路上的任何其他網路裝置

因為容器端點能直接存取實體網路，所以不需要指定連接埠對應

### L2 橋接器 

若要使用 L2 橋接網路模式，請使用驅動程式名稱 'l2bridge' 建立容器網路。 同樣地，必須指定對應至實體網路的子網路和閘道。

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

只有 l2bridge 網路支援靜態 IP 指派。 

> 在 SDN 網狀架構上使用 l2bridge 網路時，只支援動態 IP 指派。 如需詳細資訊，請參閱[將容器連接到虛擬網路](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)主題。

## 其他的作業和設定

### 列出可用網路

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### 移除網路

使用 `docker network rm` 刪除容器網路。

```none
C:\> docker network rm "<network name>"
```

這會清除容器網路所使用的任何 Hyper-V 虛擬交換器，以及任何建立的網路位址轉譯 (WinNAT - NetNat 執行個體)。

### 網路檢查 

您可以執行下列程式碼，查看哪些容器連接到特定的網路，以及已與這些容器端點建立關聯的 IP。

```none
C:\> docker network inspect <network name>
```

### 多個容器網路

您可以在單一容器主機上建立多個容器網路，注意事項如下：
* 每個容器主機只能建立一個 NAT 網路。
* 使用外部 vSwitch 進行連線的多個網路 (例如廣域網路、L2 橋接或 L2 廣域網路) 皆必須使用自己的網路介面卡。

### 網路選取

建立 Windows 容器時，可以指定容器網路介面卡要連接的網路。 如果未指定任何網路，則會使用預設 NAT 網路。

若要將容器連結至非預設 NAT 網路，可搭配使用 --network 選項與 Docker run 命令。

```none
C:\> docker run -it --network=MyTransparentNet windowsservercore cmd
```

### 靜態 IP 位址

```none
C:\> docker run -it --network=MyTransparentNet --ip=10.80.123.32 windowsservercore cmd
```

靜態 IP 指派會直接在容器的網路介面卡上執行，且容器必須處於停止狀態，才能執行。 在執行容器時，不支援容器網路介面卡的「熱新增」或變更網路堆疊。


## 注意事項和陷阱

### 防火牆

需建立容器主機的特定防火牆規則，以啟用 ICMP (Ping) 和 DHCP。 Windows Server 容器需使用 ICMP 和 DHCP 以在相同主機上兩個容器之間 Ping，並接收透過 DHCP 動態指派的 IP 位址。 在 TP5 中，可透過 Install-ContainerHost.ps1 指令碼建立這些規則。 在 TP5 之後，這些規則會自動建立。 所有對應至 NAT 連接埠轉送規則的防火牆規則會自動建立，並在容器停止時將其清除。

### 不支援的功能

目前，不支援透過 Docker CLI 使用下列網路功能
 * 容器連結 (例如 --link)
 * 以名稱/服務為主的容器 IP 解析。 _這將很快在服務更新中獲得支援_
 * 預設重疊網路驅動程式

目前，Windows Docker 不支援下列網路選項：
 * --add-host
 * --dns
 * --dns-opt
 * --dns-search
 * -h、--hostname
 * --net-alias
 * --aux-address
 * --internal
 * --ip-range

 > 在 Windows Server 2016 Technical Preview 5 與最新 Windows Insider Preview (WIP)「正式發行前小眾測試」組建中有個已知錯誤，即升級為新組建時，會造成容器網路和 vSwitch 重複 (即「流失」)。 若要解決這個問題，請執行下列指令碼。
```none
$KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
$keys = get-childitem $KeyPath
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
remove-netnat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # Note: failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```
> 重新啟動主機，然後執行剩餘的步驟︰
```none
Get-NetNat | Remove-NetNat -Confirm $false
Set-Service docker -StartupType automatic
Start-Service docker 
```



<!--HONumber=Aug16_HO4-->


