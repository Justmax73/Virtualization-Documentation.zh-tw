---
title: "Windows 容器的網路功能"
description: "設定 Windows 容器的網路功能。"
keywords: "docker, 容器"
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: 23d4b665da627f35cf5fce49c3c9974d0ef287dd
ms.openlocfilehash: e56a5b984cc1c42e27628d00a5cd532788aef11c
ms.lasthandoff: 02/10/2017

---

# 容器的網路功能

Windows 容器在網路功能方面類似於虛擬機器。 每個容器都有一個連線到虛擬交換器 (vSwitch) 的虛擬網路介面卡 (vNIC)，用以轉送輸入和輸出流量。 為了強制執行相同主機上的容器隔離，系統會為每個 Windows Server 和 Hyper-V 容器 (其中已針對容器安裝網路介面卡) 建立網路區間。 Windows Server 容器使用主機 vNIC 連結到虛擬交換器。 Hyper-V 容器使用綜合 VM NIC (不向公用程式 VM 公開) 連結到虛擬交換器。

Windows 容器支援四種不同的網路驅動程式或模式︰*nat*、*transparent*、*l2bridge* 和 *l2tunnel*。 根據您的實體網路基礎結構和單一與多主機網路功能需求，您應該選擇最符合需求的網路模式。

Docker 引擎依預設會在 dockerd 服務第一次執行時建立 NAT 網路。 建立的預設內部 IP 首碼是 172.16.0.0/12。 容器端點會自動附加至這個預設網路，並從內部首碼指派一個 IP 位址。

> 注意︰如果容器主機 IP 也是這個相同首碼，您必須變更內部 NAT IP 首碼，如下所述。

可以在相同的容器主機上建立其他使用不同驅動程式 (例如 transparent、l2bridge) 的網路。 下表顯示如何提供每個模式的內部 (容器對容器) 和外部連接的網路連線。

- **網路位址轉譯 (NAT)** – 每個容器將會從內部、私用 IP 首碼 (例如 172.16.0.0/12) 收到 IP 位址。 支援連接埠轉送/從容器主機到容器端點的對應

- **透明** – 每個容器端點直接連接至實體網路。 實體網路的 IP 可以是靜態指派，或是從外部 DHCP 伺服器動態指派。

- **\[新增！\] 覆疊** - 當 Docker 引擎以[群集模式](./swarm-mode.md)執行時，覆疊網路 (以 VXLAN 技術為基礎) 可於連接多部容器主機上的容器端點。 於群集叢集上建立的每個覆疊網路，在建立時即具有其本身的 IP 子網路，其由私人 IP 首碼所定義。

- **L2 橋接器** - 每個容器端點會在與容器主機相同的 IP 子網路中。 必須以靜態方式從與容器主機相同的首碼指派 IP 位址。 因為第 2 層位址轉譯的關係，主機上的所有容器端點會具有相同的 MAC 位址。

- **L2 通道** - _ 這種模式應該僅限用於 Microsoft 雲端堆疊_

> 若要了解如何將容器端點連接至具有 Microsoft SDN 堆疊的重疊虛擬網路，請參閱[將容器連接至虛擬網路](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)主題。

## 單一節點

|  | 容器對容器 | 容器外部 |
| :---: | :---------------     |  :---                |
| NAT | 透過 Hyper-V 虛擬交換器橋接的連線 | 透過 WinNAT 路由並套用位址轉譯 |
| 透明 | 透過 Hyper-V 虛擬交換器橋接的連線 | 直接存取實體網路 |
| 覆疊 | VXLAN 封裝發生在 VFP 將延伸模組轉送至 Hyper-V 虛擬交換器的過程中，*內部主機*通訊會藉由透過 Hyper-V 虛擬交換器橋接的連線來進行 | 透過 WinNAT 路由並套用位址轉譯
| l2bridge | 透過 Hyper-V 虛擬交換器橋接的連線|  存取實體網路並具有 MAC 位址轉譯|  



## 多節點

|  | 容器對容器 | 容器外部 |
| :---: | :----       | :---------- |
| NAT | 必須參考外部容器主機 IP 和連接埠；透過 WinNAT 路由並套用位址轉譯 | 必須參考外部主機；透過 WinNAT 路由並套用位址轉譯 |
| 透明 | 必須直接參考容器 IP 端點 | 直接存取實體網路 |
| 覆疊 | VXLAN 封裝發生在 VFP 將延伸模組轉送至 Hyper-V 虛擬交換器的過程中，*內部主機*通訊直接參考 IP 端點 | 透過 WinNAT 路由並套用位址轉譯| 
| l2bridge | 必須直接參考容器 IP 端點| 存取實體網路並具有 MAC 位址轉譯|


## 建立網路

### (預設) NAT 網路

Windows Docker 引擎會建立預設的 NAT 網路 (Docker 名之為 'nat')，且使用 IP 首碼 172.16.0.0/12。 如果使用者想要使用特定的 IP 首碼來建立 NAT 網路，他們可以變更 Docker 組態 daemon.json 檔中的選項 (位於 C:\ProgramData\Docker\config\daemon.json - 若不存在則加以建立)，執行下列兩件事之一。
 1. 使用 _"fixed-cidr": "<IP 首碼> / Mask"_ 選項，這會使用指定的 IP 首碼和遮罩，建立預設的 NAT 網路。
 2. 使用 _"bridge":"none"_ 選項，這不會建立預設網路；使用者可以使用 *docker network create -d <driver>* 命令，以任何驅動程式建立使用者定義的網路。

在執行這些組態選項的其中一者之前，必須先停止 Docker 服務，並刪除任何預先存在的 NAT 網路。

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

如果在 daemon.json 檔案中新增 "fixed-cidr" 選項，則 Docker 引擎會使用指定的自訂 IP 首碼和遮罩，建立使用者定義的 NAT 網路。 但若新增的是 "bridge:none" 選項，即必須手動建立網路。

```none
# Create a user-defined NAT network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

依預設，容器端點將會連線到預設 NAT 網路。 如未建立 NAT 網路 (因為 daemon.json 裡指定了 "bridge:none") 或需要存取不同的使用者定義網路，則使用者可以在 docker run 命令中指定 *--network* 參數。

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### 連接埠對應

當容器連接至 NAT 網路時，若要存取在其中執行的應用程式，需要在容器主機和容器端點之間建立連接埠對應。 這些對應必須在建立容器時，或容器處於已停止狀態時指定。

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Creates a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

也支援使用 -p 參數或在 Dockerfile 中使用 EXPOSE 命令搭配 -p 參數，進行動態連接埠對應。 如未指定，容器主機上會隨機選擇暫時的連接埠，並可在執行 'docker ps' 時進行檢查。

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
 1. 每部容器主機只支援一個 NAT 內部 IP 首碼，所以必須藉由分割首碼定義「多個」NAT 網路 (請參閱本文件的＜多個 NAT 網路＞一節)。
 2. 容器的端點只能從使用容器內部 IP 和連接埠的容器主機連接 (請使用「docker 網路檢查 <CONTAINER ID>」尋找此資訊)。

可以使用不同的驅動程式建立額外的網路。

> Docker 網路驅動程式全為小寫。

### 透明網路

若要使用廣域網路模式，請使用驅動程式名稱 'transparent' 建立容器網路。

```none
C:\> docker network create -d transparent MyTransparentNetwork
```
> 注意︰如果在建立透明網路時發生錯誤，可能是系統上有未經 Docker 自動探索到的外部 vSwitch，以致透明網路無法繫結至容器主機的外部網路介面卡。 如需詳細資訊，請參考＜注意事項和陷阱＞下的＜現有的 vSwitch 妨礙建立透明網路＞一節。

如果容器主機已虛擬化，而且您想要使用 DHCP 進行 IP 指派，則必須在虛擬機器網路介面卡上啟用 MACAddressSpoofing。 否則，Hyper-V 主機將會封鎖來自 VM 中具有多個 MAC 位址之容器的網路流量。

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> 若要建立多個透明網路，您必須指定外部 Hyper-V 虛擬交換器 (自動建立) 應該繫結的 (虛擬) 網路介面卡。

連線到透明網路的容器端點 IP 位址可以是靜態指派，或是從外部 DHCP 伺服器動態指派。

使用靜態 IP 指派時，您必須先確認建立網路時指定了 *--subnet* 和 *--gateway* 參數。 子網路和閘道 IP 位址應該與容器主機的網路設定相同 - 也就是和實體網路相同。

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
使用 `docker run` 命令的 *--ip* 選項指定 IP 位址：

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> 請確定此 IP 位址未指派給實體網路上的任何其他網路裝置

因為容器端點能直接存取實體網路，所以不需要指定連接埠對應。

### 覆疊網路

*若要使用覆疊網路模式，您必須使用以群集模式執行的 Docker 主機來作為管理員節點。* 如需深入了解群集模式，以及如何初始化群集管理員，請參閱[開始使用群集模式](./swarm-mode.md)主題。

若要建立覆疊網路，請自**群集管理員節點**執行下列命令：

```none
# Create an overlay network from a swarm manager node, called "myOverlayNet"
C:\> docker network create --driver=overlay myOverlayNet
```

### L2 橋接器

若要使用 L2 橋接網路模式，請使用驅動程式名稱 'l2bridge' 建立容器網路。 同樣地，必須指定對應至實體網路的子網路和閘道。

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

只有 l2bridge 網路支援靜態 IP 指派。

> 在 SDN 網狀架構上使用 l2bridge 網路時，只支援動態 IP 指派。 如需詳細資訊，請參閱[將容器連接到虛擬網路](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)主題。

## 其他的作業和設定

> 我們會持續不斷地改善 Windows 上的 Docker。 為確保您能夠存取所有的最新功能，請確認您所使用的 Docker 引擎是最新版本。 您可以使用 `docker -v` 檢查您的 Docker 版本。 如需有關設定 Docker 方面的指引，請參閱 [Windows 上的 Docker 引擎](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon)主題。

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
C:\> docker network rm <network name>
```

這會清除容器網路所使用的任何 Hyper-V 虛擬交換器，以及任何建立的網路位址轉譯 (WinNAT - NetNat 執行個體)。

### 網路檢查

您可以執行下列程式碼，查看哪些容器連接到特定的網路，以及與這些容器端點建立關聯的 IP。

```none
C:\> docker network inspect <network name>
```

### 將網路名稱指定給 HNS 服務

**一般而言，當您使用 `docker network create` 建立容器網路時，您所提供的網路名稱會由 Docker 服務使用，但不會由 HNS 服務使用。**

如果您要建立網路，您就可以在 `docker network create` 命令上使用 `-o com.docker.network.windowsshim.networkname=<network name>` 選項，以指定 HNS 服務提供的名稱。 例如，您可以使用下列命令，以建立名稱為指定給 HNS 服務的透明網路︰

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

#### 範例：預設 HNS 命名行為

若要了解此命名選項的行為，下列螢幕擷圖示取示範了*不*使用此命名選項時，HNS 服務如何為網路命名。 在此範例中，Docker 可以看到網路的名稱 "MyTransparentNetwork"，就如透過 `docker network ls` 命令所示。 但 HNS 服務看不到網路的名稱，就如透過 `Get-ContainerNetwork` Windows PowerShell 命令所示；反之，HNS 會自動產生長串的英數字元網路名稱。

><figure>
  <img src="media/SpecifyName_Capture.PNG">
  <figcaption>範例︰「不」<i></i>會將網路的名稱指定至 HNS 服務。 </figcaption>
</figure>

#### 範例︰將網路名稱指定給 HNS 服務

*有*使用 `-o com.docker.network.windowsshim.networkname=<network name>` 時，HNS 服務反而會使用指定的名稱，而不是產生的名稱。 下方螢幕擷取圖示範此行為。

><figure>
  <img src="media/SpecifyName_Capture_2.PNG">
  <figcaption>範例︰使用 `-o com.docker.network.windowsshim.networkname=<network name>` 選項，將網路名稱指定給 HNS 服務。</figcaption>
</figure>


### 將網路繫結至特定網路介面

若要將網路 (透過 Hyper-V 虛擬交換器附加) 繫結至特定的網路介面，請在 `docker network create` 命令上使用 `-o com.docker.network.windowsshim.interface=<Interface>` 選項。 舉例來說，您可以使用下列命令建立附加到「乙太網路 2」網路介面的透明網路：

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 注意︰*com.docker.network.windowsshim.interface* 的值為網路介面卡的*名稱*，此名稱可使用下列項目找到︰

>```none
PS C:\> Get-NetAdapter
```

### Set the VLAN ID for a Network

To set a VLAN ID for a network, use the option, `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` to the `docker network create` command. For instance, you might use the following command to create a transparent network with a VLAN ID of 11:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
當您為網路設定 VLAN 識別碼時，您是在為連結至該網路的任何容器端點設定 VLAN 隔離。

**注意︰**請確保主機網路介面卡 (實體) 處於主幹模式，以讓 vSwitch 處理所有標記的流量 (但 vNIC (容器端點) 連接埠必須在正確的 VLAN 上處於存取模式)。


### 指定 DNS 尾碼及/或網路的 DNS 伺服器

使用 `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` 選項來指定網路的 DNS 尾碼，並使用 `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` 選項來指定網路的 DNS 伺服器。 舉例來說，您可使用下列命令將網路的 DNS 尾碼設定為 "example.com"，並將網路的 DNS 伺服器設為 4.4.4.4 和 8.8.8.8︰

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

### 多個容器網路
您可以在單一容器主機上建立多個容器網路，注意事項如下：

* 使用外部 vSwitch 進行連線的多個網路 (例如廣域網路、L2 橋接或 L2 廣域網路) 皆必須使用自己的網路介面卡。
* 目前，在單一容器主機上建立多個 NAT 網路的解決方案，是分割現有 NAT 網路的內部首碼。 如需對此的進一步指引，請參考下節＜多個 NAT 網路＞。

### 多個 NAT 網路
藉由分割主機的 NAT 網路內部首碼，有可能在單一容器主機上定義多個 NAT 網路。

任何新 NAT 網路的分割都必須建立在較大的內部 NAT 網路首碼之下。 從 PowerShell 執行下列命令，並參考 "InternalIPInterfaceAddressPrefix" 欄位即可找到首碼。

```none
PS C:\> Get-NetNAT
```

例如，主機的 NAT 網路內部首碼可能是 172.16.0.0/12。 在此例中，您可以使用 Docker 建立其他的 NAT 網路，*只要它們都落在 172.16.0.0/12 首碼下*。 例如，使用 IP 首碼 172.16.0.0/16 (閘道，172.16.0.1) 及 172.17.0.0/16 (閘道，172.17.0.1) 可建立兩個 NAT 網路。

```none
C:\> docker network create -d nat --subnet=172.16.0.0/16 --gateway=172.16.0.1 CustomNat1
C:\> docker network create -d nat --subnet=172.17.0.0/16 --gateway=172.17.0.1 CustomNat2
```

新建立的網路可以列出，使用︰
```none
C:\> docker network ls
```


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

## Docker Compose 和服務探索

> 如何使用 Docker Compose 與服務探索定義多服務且向外延展之應用程式的實用範例，請瀏覽我們 [Virtualization Blog](https://blogs.technet.microsoft.com/virtualization/) 上的[這篇文章](https://blogs.technet.microsoft.com/virtualization/2016/10/18/use-docker-compose-and-service-discovery-on-windows-to-scale-out-your-multi-service-container-application/)。

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) 可用來定義與設定容器網路以及要使用這些網路的容器/服務。 Compose 的「網路」機碼在定義容器要連接的網路時，會作為最上層機碼使用。 例如，下列語法會將由 Docker 建立的既存 NAT 網路，定義成在指定的 Compose 檔案中所定義之所有容器/服務的「預設」網路。

```none
networks:
 default:
  external:
   name: "nat"
```

同樣地，下列語法可以用來定義自訂的 NAT 網路。

> 注意：下例中定義的「自訂 NAT 網路」，是定義為容器主機的既存 NAT 內部首碼的磁碟分割。 如需詳細內容，請參閱上述＜多個 NAT 網路＞一節。

```none
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.17.0.0/16
```

如需使用 Docker Compose 定義/設定容器網路的詳細資訊，請參閱 [Compose File reference](https://docs.docker.com/compose/compose-file/)。

### 服務探索
Docker 內建的服務探索，處理容器和服務的服務註冊及 IP (DNS) 名稱對應；使用服務探索，就可能讓所有容器端點依名稱 (容器名稱或服務名稱) 發現彼此。 這在使用多個容器端點定義單一服務以向外延展的案例中，特別有價值。 在這種情況下，服務探索讓服務可以被視為單一的實體，不管它在幕後執行多少容器。 對多容器服務而言，連入網路流量是使用循環配置資源方法管理，使用 DNS 負載平衡將流量統一分散在所有實作給定服務的容器執行個體。

## 覆疊網路與 Docker 群集模式 (多節點容器網路功能)
原生覆疊網路驅動程式和 Docker 群集模式組合，以提供 Windows 上多節點 (叢集) 案例的支援。 如需深入了解覆疊及群集模式，請造訪我們的[部落格文章](https://blogs.technet.microsoft.com/virtualization/2017/02/09/overlay-network-driver-with-support-for-docker-swarm-mode-now-available-to-windows-insiders-on-windows-10/)，該文章會在覆疊或群集功能發行時隨附給 Windows 10 的 Windows 測試人員，或參考[開始使用群集模式](./swarm-mode.md) (英文) 主題。

## 注意事項和陷阱

### 現有的 vSwitch 妨礙建立透明網路

在建立透明網路時，Docker 會建立網路的外部 vSwitch，再嘗試將參數繫結至 (外部) 網路介面卡，此介面卡可以是 VM 網路介面卡或實體網路介面卡。 如果已在容器主機上建立了 vSwitch，且*顯示至 Docker*，則 Windows Docker 引擎會使用該參數，而不是新建一個參數。 不過，如果 vSwitch 是在頻外建立 (亦即使用 HYPER-V 管理員或 PowerShell 在容器主機上建立的)，且截至目前還未顯示在 Docker，則 Windows Docker 引擎會嘗試建立新的 vSwitch，然後無法連接到容器主機外部網路介面卡的新參數 (因為網路介面卡已經被頻外建立的參數所連接)。

例如，如果執行 Docker 服務時先在主機上建立新的 vSwitch，再嘗試建立透明網路，這個問題就會發生。 在此情況下，Docker 無法識別您建立的參數，即會為透明網路建立新的 vSwitch。

有三種方法可以解決此問題︰

* 您當然可以刪除頻外建立的 vSwitch，這可讓 Docker 建立新的 vSwitch 並將它順利連接到主機網路介面卡。 選擇此方法之前，請確定沒有其他服務使用您的頻外 vSwitch (例如 HYPER-V)。
* 或者，如果您決定使用頻外建立的外部 vSwitch，請重新啟動 Docker 和 HNS 服務以*將參數顯示到 Docker*。
```none
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* 另一個選項是使用 '-o com.docker.network.windowsshim.interface' 選項將透明網路的外部 vSwitch 繫結至容器主機上尚未使用的特定網路介面卡 (亦即不是頻外建立的 vSwitch 所使用的網路介面卡)。 前文提及的本文件[透明網路](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network)一節中會詳加說明 '-o' 選項。

### 不支援的功能

現今不支援透過 Docker CLI 使用下列網路功能
 * 容器連結 (例如 --link)

目前，Windows Docker 不支援下列網路選項：
 * --add-host
 * --dns-opt
 * --dns-search
 * --aux-address
 * --internal
 * --ip-range

