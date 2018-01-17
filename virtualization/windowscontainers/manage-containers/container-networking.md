---
title: "Windows 容器的網路功能"
description: "設定 Windows 容器的網路功能。"
keywords: "Docker, 容器"
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 394aa58c3421e512d005f59d5bd30667f1c26f16
ms.sourcegitcommit: 6eefb890f090a6464119630bfbdc2794e6c3a3df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/18/2017
---
# <a name="windows-container-networking"></a>Windows 容器的網路功能
> ***請參考 [Docker 容器的網路功能](https://docs.docker.com/engine/userguide/networking/)，以了解一般 Docker 網路功能的命令、選項和語法。*** 除了本文件所述的任何例外情形，所有 Docker 網路功能命令在 Windows 上使用的語法皆與 Linux 相同。 但請注意，Windows 與 Linux 的網路堆疊並不同，因此您會發現 Windows 不支援某些 Linux 網路命令 (例如 ifconfig)。

## <a name="basic-networking-architecture"></a>基本網路架構
本主題概述 Docker 如何在 Windows 上建立及管理網路。 Windows 容器在網路功能方面類似虛擬機器。 每個容器都有連接到 Hyper-V 虛擬交換器 (vSwitch) 的虛擬網路介面卡 (vNIC)。 Windows 支援可透過 Docker 來建立的五種不同網路驅動程式或模式：*nat*、*overlay*、*transparent*、*l2bridge* 和 *l2tunnel*。 根據您的實體網路基礎結構和單一與多主機網路功能需求，您應該選擇最符合需求的網路驅動程式。

<figure>
  <img src="media/windowsnetworkstack-simple.png">
</figure>  

Docker 引擎第一次執行時，會建立預設 NAT 網路 'nat'，它會使用內部 vSwitch 和名為 `WinNAT` 的 Windows 元件。 如果主機上有透過 PowerShell 或 Hyper-V 管理員來建立的任何既有外部 vSwitch，Docker 也可以透過 *transparent* 網路驅動程式來使用，而當您執行 ``docker network ls`` 命令時，也會看到這些 vSwitch。  

<figure>
  <img src="media/docker-network-ls.png">
</figure>

> - ***內部*** vSwitch 是指非直接連接至容器主機網路介面卡的 vSwitch 

> - ***外部*** vSwitch 是指_直接_連接至容器主機網路介面卡的 vSwitch  

<figure>
  <img src="media/get-vmswitch.png">
</figure>

'nat' 網路是在 Windows 上執行的容器的預設網路。 任何容器如果在 Windows 上執行，但沒有任何旗標或引數實作特定網路組態，其將會連接到預設的 'nat' 網路，且系統會自動從 'nat' 網路的內部首碼 IP 範圍指派 IP 位址給容器。 用於 'nat' 的預設內部 IP 首碼為 172.16.0.0/16。 


## <a name="windows-container-network-drivers"></a>Windows 容器網路驅動程式  

使用者除了可以在 Windows 上運用 Docker 建立的預設 'nat' 網路，也可以定義自訂容器網路。 使用者可以使用 Docker CLI 命令 [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) 來建立使用者定義的網路。 在 Windows 上，可使用下列網路驅動程式類型：

- **nat** – 如果容器連接到以 'nat' 驅動程式建立的網路，其將會從使用者指定的 (``--subnet``) IP 首碼收到 IP 位址。 支援從容器主機到容器端點的連接埠轉送/對應。
> 注意：現在可透過 Windows 10 Creators Update 支援多個 NAT 網路！ 

- **transparent** – 如果容器連接到以 'transparent' 驅動程式建立的網路，其將會直接連接到實體網路。 實體網路的 IP 可以透過外部 DHCP 伺服器，以靜態 (需要使用者指定的 ``--subnet`` 選項) 或動態方式指派。 

- **overlay** - __最新！__  當 Docker 引擎以[群集模式](./swarm-mode.md)執行時，連接到覆疊網路的容器可以與連接到相同網路的其他容器跨多個容器主機進行通訊。 建立在群集叢集上的每個覆疊網路，在建立時即具有其本身的 IP 子網路 (由私人 IP 首碼定義)。 overlay 網路驅動程式會使用 VXLAN 封裝。
> 需要含 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) 的 Windows Server 2016 或 Windows 10 Creators Update 

- **l2bridge** - 如果容器連接到以 'l2bridge' 驅動程式建立的網路，其將會在與容器主機相同的 IP 子網路中。 必須以靜態方式從與容器主機相同的首碼指派 IP 位址。 因為在輸入和輸出時都有第 2 層位址轉譯 (MAC 重寫) 作業的關係，主機上的所有容器端點都會有相同的 MAC 位址。
> 需要 Windows Server 2016 或 Windows 10 Creators Update

- **l2tunnel** - _此驅動程式應該僅限用於 Microsoft 雲端堆疊_

> 若要了解如何將容器端點連接至具有 Microsoft SDN 堆疊的重疊虛擬網路，請參閱[將容器連接至虛擬網路](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)主題。

> Windows 10 Creators Update 導入平台支援，可新增容器端點至執行中的容器 (也就是「熱新增」)。 這將會導致端對端擱置[待處理的 Docker 提取要求](https://github.com/docker/libnetwork/pull/1661)

## <a name="network-topologies-and-ipam"></a>網路拓撲和 IPAM
下表顯示如何針對每一種網路驅動程式的內部 (容器對容器) 和外部連接提供網路連線。

<figure>
  <img src="media/network-modes-table.png">
</figure>

### <a name="ipam"></a>IPAM 
針對每一種網路驅動程式，配置及指派 IP 位址的方式各不相同。 Windows 會使用主機網路服務 (HNS) 來為 nat 驅動程式提供 IPAM，並以 Docker 群集模式 (內部 KVS) 來為 overlay 驅動程式提供 IPAM。 所有其他網路驅動程式皆使用外部 IPAM。

<figure>
  <img src="media/ipam.png">
</figure>

# <a name="details-on-windows-container-networking"></a>Windows 容器網路詳細資訊

## <a name="isolation-namespace-with-network-compartments"></a>以網路區間進行隔離 (命名空間)
每個容器端點皆位於自己的__網路區間__，就像是 Linux 的網路命名空間。 管理主機 vNIC 和主機網路堆疊位於預設的網路區間。 為了在相同主機上的容器之間強制執行網路隔離，系統會為每個 Windows Server 和 Hyper-V 容器 (其中已針對容器安裝網路介面卡) 建立網路區間。 Windows Server 容器使用主機 vNIC 連結到虛擬交換器。 Hyper-V 容器使用綜合 VM NIC (不向公用程式 VM 公開) 連結到虛擬交換器。 

<figure>
  <img src="media/network-compartment-visual.png">
</figure>

```powershell 
Get-NetCompartment
```

## <a name="windows-firewall-security"></a>Windows 防火牆安全性

Windows 防火牆可用來在所有連接埠 ACL 中強制執行網路安全性。

> 注意：依預設，連接至覆疊網路的所有容器端點都已建立「全部允許」規則   

<figure>
  <img src="media/windows-firewall-containers.png">
</figure>

## <a name="container-network-management-with-host-network-service"></a>使用主機網路服務進行容器網路管理

下圖顯示主機網路服務 (HNS) 和主機運算服務 (HCS) 如何共同建立容器，並將端點連接至網路。 

<figure>
  <img src="media/HNS-Management-Stack.png">
</figure>

# <a name="advanced-network-options-in-windows"></a>Windows 中的進階網路選項
支援多種網路驅動程式選項，以利用 Windows 特定的功能和特性。 

## <a name="switch-embedded-teaming-with-docker-networks"></a>搭配 Docker 網路的 Switch Embedded Teaming (交換器內嵌小組)

> 適用於所有網路驅動程式 

建立容器主機網路來供 Docker 使用時，您可以利用 [Switch Embedded Teaming (交換器內嵌小組)](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set)，以 `-o com.docker.network.windowsshim.interface` 選項來指定多個網路介面卡 (以逗號分隔)。 

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>為網路設定 VLAN 識別碼

> 適用於 transparent 和 l2bridge 網路驅動程式 

若要為網路設定 VLAN 識別碼，請在 `docker network create` 命令中使用 `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` 選項。 例如，您可以使用下列命令來建立 VLAN 識別碼為 11 的透明網路：

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
當您為網路設定 VLAN 識別碼時，您是在為連結至該網路的任何容器端點設定 VLAN 隔離。

> 請確保主機網路介面卡 (實體) 處於主幹模式，以讓 vSwitch 能夠處理所有標記的流量 (vNIC (容器端點) 連接埠在正確的 VLAN 上處於存取模式)。

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>將網路名稱指定給 HNS 服務

> 適用於所有網路驅動程式 

一般而言，當您使用 `docker network create` 建立容器網路時，您所提供的網路名稱會由 Docker 服務使用，但不會由 HNS 服務使用。 如果您要建立網路，您就可以在 `docker network create` 命令上使用 `-o com.docker.network.windowsshim.networkname=<network name>` 選項，以指定 HNS 服務提供的名稱。 例如，您可以使用下列命令，以指定給 HNS 服務的名稱來建立透明網路︰

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>將網路繫結至特定網路介面

> 適用於 'nat' 以外的所有網路驅動程式  

若要將網路 (透過 Hyper-V 虛擬交換器連接) 繫結至特定的網路介面，請在 `docker network create` 命令中使用 `-o com.docker.network.windowsshim.interface=<Interface>` 選項。 舉例來說，您可以使用下列命令建立附加到「乙太網路 2」網路介面的透明網路：

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 注意︰*com.docker.network.windowsshim.interface* 的值為網路介面卡的*名稱*，可使用下列命令找到此名稱︰

>```
PS C:\> Get-NetAdapter
```
## Specify the DNS Suffix and/or the DNS Servers of a Network

> Applies to all network drivers 

Use the option, `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` to specify the DNS suffix of a network, and the option, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` to specify the DNS servers of a network. For example, you might use the following command to set the DNS suffix of a network to "example.com" and the DNS servers of a network to 4.4.4.4 and 8.8.8.8:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## VFP

The Virtual Filtering Platform (VFP) extension is a Hyper-V virtual switch, forwarding extension used to enforce network policy and manipulate packets. For instance, VFP is used by the 'overlay' network driver to perform VXLAN encapsulation and by the 'l2bridge' driver to perform MAC re-write on ingresss and egress. The VFP extension is only present on Windows Server 2016 and Windows 10 Creators Update. To check and see if this is running correctly a user run two commands:

```powershell
Get-Service vfpext

# This should indicate the extension is Running: True 
Get-VMSwitchExtension  -VMSwitchName <vSwitch Name> -Name "Microsoft Azure VFP Switch Extension"
```

## <a name="tips--insights"></a>提示與深入解析
以下是有用的提示與深入解析，靈感來自我們聽到社群對 Windows 容器網路的常見問題...

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS 需要容器主機啟用 IPv6 
HNS 是隸屬於 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217)，需要 Windows 容器主機啟用 IPv6。 如果您遇到像下面這種錯誤，很有可能是因為主機已停用 IPv6。
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
我們正在進行平台改造，以自動偵測/避免此問題。 目前可以使用下列因應措施來確保主機啟用 IPv6：

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition-instead-of-hns-internal-vswitch"></a>Moby Linux VM 使用 DockerNAT 交換器來搭配 Docker for Windows ([Docker CE](https://www.docker.com/community-edition) 的產品)，而不是使用 HNS 內部 vSwitch 
Windows 10 上的 Docker for Windows (適用於 Docker CE 引擎的 Windows 驅動程式) 將會使用名為 'DockerNAT' 的內部 vSwitch，來將 Moby Linux VM 連接至容器主機。 在 Windows 上使用 Moby Linux VM 的開發人員應注意，其主機是使用 DockerNAT vSwitch，而不是 HNS 服務建立的 vSwitch (用於 Windows 容器的預設交換器)。 

#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>若要在虛擬容器主機上使用 DHCP 來進行 IP 指派，請啟用 MACAddressSpoofing 
如果容器主機已虛擬化，而且您想要使用 DHCP 來進行 IP 指派，則必須在虛擬機器的網路介面卡上啟用 MACAddressSpoofing。 否則，Hyper-V 主機將會封鎖來自 VM 中具有多個 MAC 位址之容器的網路流量。 您可以使用此 PowerShell 命令來啟用 MACAddressSpoofing：
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
如果您執行 VMware 做為 Hypervisor，您必須啟用混合模式以使此功能發揮作用。 [這裡](https://kb.vmware.com/s/article/1004099)可以取得詳細資料。
#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>在單一容器主機上建立多個透明網路
如果想要建立多個透明網路，您必須指定外部 Hyper-V 虛擬交換器應該繫結的 (虛擬) 網路介面卡。 若要指定網路的介面，請使用下列語法：
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>使用靜態 IP 指派時，請記得要指定 *--subnet* 和 *--gateway*
使用靜態 IP 指派時，您必須先確認建立網路時已指定 *--subnet* 和 *--gateway* 參數。 子網路和閘道 IP 位址應該與容器主機的網路設定相同，也就是實體網路。 例如，以下說明您可能會如何建立透明網路，然後使用靜態 IP 指派在該網路上執行端點：

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>L2Bridge 網路不支援 DHCP IP 指派
唯有使用 l2bridge 驅動程式來建立的容器網路可支援靜態 IP 指派。 如前述，請記得要使用 *--subnet* 和 *--gateway* 參數來建立為靜態 IP 指派而設定的網路。

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>使用外部 vSwitch 的每個網路都必須要有自己的網路介面卡
請注意，如果在相同的容器主機上建立多個使用外部 vSwitch 來連接的網路 (例如透明網路、L2 橋接網路、L2 透明網路)，則每個網路都需要有自己的網路介面卡。 

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>在已停止與執行中的容器上進行 IP 指派
靜態 IP 指派會直接在容器的網路介面卡上執行，且容器必須處於停止狀態，才能執行。 在執行容器時，(在 Windows Server 2016 中) 不支援容器網路介面卡的「熱新增」或變更網路堆疊。
> 注意：在 Windows 10 Creators Update 上，此行為已變更，因為現在該平台可支援「熱新增」。 在合併這個[待處理的 Docker 提取要求](https://github.com/docker/libnetwork/pull/1661)之後，此功能將會啟用 E2E

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>現有的 vSwitch (Docker 看不到) 會阻止建立透明網路
如果在建立透明網路時發生錯誤，可能是系統上有未經 Docker 自動探索到的外部 vSwitch，以致透明網路無法繫結至容器主機的外部網路介面卡。 

在建立透明網路時，Docker 會建立網路的外部 vSwitch，再嘗試將參數繫結至 (外部) 網路介面卡，此介面卡可以是 VM 網路介面卡或實體網路介面卡。 如果已在容器主機上建立了 vSwitch，且*顯示至 Docker*，則 Windows Docker 引擎會使用該參數，而不是新建一個參數。 不過，如果 vSwitch 是在頻外建立 (亦即使用 HYPER-V 管理員或 PowerShell 在容器主機上建立的)，且截至目前還未顯示在 Docker，則 Windows Docker 引擎會嘗試建立新的 vSwitch，然後無法連接到容器主機外部網路介面卡的新參數 (因為網路介面卡已經被頻外建立的參數所連接)。

例如，如果執行 Docker 服務時先在主機上建立新的 vSwitch，再嘗試建立透明網路，這個問題就會發生。 在此情況下，Docker 無法識別您建立的參數，即會為透明網路建立新的 vSwitch。

有三種方法可以解決此問題︰

* 您當然可以刪除頻外建立的 vSwitch，這可讓 Docker 建立新的 vSwitch 並將它順利連接到主機網路介面卡。 選擇此方法之前，請確定沒有其他服務使用您的頻外 vSwitch (例如 HYPER-V)。
* 或者，如果您決定使用頻外建立的外部 vSwitch，請重新啟動 Docker 和 HNS 服務以*將參數顯示到 Docker*。
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* 另一個選項是使用 '-o com.docker.network.windowsshim.interface' 選項將透明網路的外部 vSwitch 繫結至容器主機上尚未使用的特定網路介面卡 (亦即不是頻外建立的 vSwitch 所使用的網路介面卡)。 如需 '-o' 選項的詳細說明，請參閱本文件前面的[透明網路](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network)一節。


## <a name="unsupported-features-and-network-options"></a>不支援的功能和網路選項 

Windows 不支援下列網路選項，無法將其傳遞至 ``docker run``：
 * 容器連結 (例如 ``--link``) - _ 替代方案視服務探索而定_
 * IPv6 位址 (例如 ``--ip6``)
 * DNS 選項 (例如 ``--dns-option``)
 * 多個 DNS 搜尋網域 (例如 ``--dns-search``)
 
Windows 不支援下列網路選項和功能，無法將其傳遞至 ``docker network create``：
 * --aux-address
 * --internal
 * --ip-range
 * --ipam-driver
 * --ipam-opt
 * --ipv6 

Docker 服務不支援下列網路選項
* 資料層加密 (例如 ``--opt encrypted``) 


## <a name="windows-server-2016-work-arounds"></a>Windows Server 2016 因應方式 

雖然我們不斷增加新功能並努力開發，但其中某些功能將無法支援較舊的平台。 最佳行動方案是「跳上火車」，以獲得 Windows 10 和 Windows Server 的最新更新。  下節列出適用於 Windows Server 2016 和 Windows 10 較舊版本 (亦即 1704 Creators Update 之前) 的一些因應方式和注意事項

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>WS2016 容器主機上的多個 NAT 網路

任何新 NAT 網路的分割都必須建立在較大的內部 NAT 網路首碼之下。 從 PowerShell 執行下列命令，並參考 "InternalIPInterfaceAddressPrefix" 欄位即可找到首碼。

```
PS C:\> Get-NetNAT
```

例如，主機的 NAT 網路內部首碼可能是 172.16.0.0/16。 在此例中，可以使用 Docker 來建立其他 NAT 網路，*但其必須為 172.16.0.0/16 首碼的子集*。 例如，使用 IP 首碼 172.16.1.0/24 (閘道，172.16.1.1) 及 172.16.2.0/24 (閘道，172.16.2.1) 可建立兩個 NAT 網路。

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

新建立的網路可以使用下列方式列出︰
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) 可用來定義與設定容器網路以及要使用這些網路的容器/服務。 Compose 的「網路」機碼在定義容器要連接的網路時，會做為最上層機碼使用。 例如，下列語法會將由 Docker 建立的既存 NAT 網路，定義成在指定的 Compose 檔案中所定義之所有容器/服務的「預設」網路。

```
networks:
 default:
  external:
   name: "nat"
```

同樣地，下列語法可以用來定義自訂的 NAT 網路。

> 注意：下例中定義的「自訂 NAT 網路」，是定義為容器主機的既存 NAT 內部首碼的磁碟分割。 如需詳細內容，請參閱上述＜多個 NAT 網路＞一節。

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

如需使用 Docker Compose 定義/設定容器網路的詳細資訊，請參閱 [Compose File reference](https://docs.docker.com/compose/compose-file/)。

### <a name="service-discovery"></a>服務探索
只有某些 Windows 網路驅動程式可支援服務探索。

|  | 本機服務探索  | 全域服務探索 |
| :---: | :---------------     |  :---                |
| nat | 是 | NA |  
| overlay | 是 | 是 |
| transparent | 否 | 否 |
| l2bridge | 否 | 否 |


