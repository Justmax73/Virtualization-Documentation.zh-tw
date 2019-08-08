---
title: Windows 容器的網路功能
description: Windows 容器的進階網路功能。
keywords: Docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 6480f0657d7def8d6da69bfc52ace81d08b0add4
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998805"
---
# <a name="advanced-network-options-in-windows"></a>Windows 中的進階網路選項

支援多種網路驅動程式選項，以利用 Windows 特定的功能和特性。 

## <a name="switch-embedded-teaming-with-docker-networks"></a>搭配 Docker 網路的 Switch Embedded Teaming (交換器內嵌小組)

> 適用於所有網路驅動程式

建立容器主機網路來供 Docker 使用時，您可以利用 [Switch Embedded Teaming (交換器內嵌小組)](https://docs.microsoft.com/windows-server/virtualization/hyper-v-virtual-switch/RDMA-and-Switch-Embedded-Teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set)，以 `-o com.docker.network.windowsshim.interface` 選項來指定多個網路介面卡 (以逗號分隔)。

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

## <a name="specify-outboundnat-policy-for-a-network"></a>指定網路的 OutboundNAT 原則

> 適用于 l2bridge 網路

一般情況下, 當您`l2bridge`使用`docker network create`建立容器網路時, 容器端點沒有套用的 HNS OutboundNAT 原則, 導致樹枝無法到達外部世界。 如果您要建立網路, 您可以使用此`-o com.docker.network.windowsshim.enable_outboundnat=<true|false>`選項來套用 OutboundNAT HNS 原則, 為容器提供外部世界的存取權:

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true MyL2BridgeNetwork
```

如果有一組目的地 (例如, 需要容器與容器連線), 那麼我們也需要指定例外情況:

```
C:\> docker network create -d l2bridge -o com.docker.network.windowsshim.enable_outboundnat=true -o com.docker.network.windowsshim.outboundnat_exceptions=10.244.10.0/24
```

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

```
PS C:\> Get-NetAdapter
```

## <a name="specify-the-dns-suffix-andor-the-dns-servers-of-a-network"></a>指定 DNS 尾碼及/或網路的 DNS 伺服器

> 適用於所有網路驅動程式 

使用 `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` 選項來指定網路的 DNS 尾碼，並使用 `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` 選項來指定網路的 DNS 伺服器。 舉例來說，您可使用下列命令將網路的 DNS 尾碼設定為 "example.com"，並將網路的 DNS 伺服器設為 4.4.4.4 和 8.8.8.8︰

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

如需詳細資訊，請參閱[這篇文章](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/)。

## <a name="tips--insights"></a>提示與深入解析
以下是有用的提示與深入解析，靈感來自我們聽到社群對 Windows 容器網路的常見問題...

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS 需要容器主機啟用 IPv6 
HNS 是隸屬於 [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)，需要 Windows 容器主機啟用 IPv6。 如果您遇到像下面這種錯誤，很有可能是因為主機已停用 IPv6。
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
我們正在進行平台改造，以自動偵測/避免此問題。 目前可以使用下列因應措施來確保主機啟用 IPv6：

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```


#### <a name="linux-containers-on-windows"></a>Windows 上的 Linux 容器

**新增：** 我們正致力於_在不使用 Moby Linux VM_的情形下，讓 Linux 及 Windows 容器可以並列方式執行。 如需詳細資訊，請參閱[關於 Windows 上的 Linux 容器 (LCOW) 的這篇部落格文章](https://blog.docker.com/2017/11/docker-for-windows-17-11/)。 以下說明如何[開始](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10-linux)使用。
> 注意：LCOW 即將取代掉 Moby Linux VM，將會使用預設 HNS「nat」內部 vSwitch。

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition"></a>Moby Linux VM 使用 DockerNAT 交換器來搭配 Docker for Windows ([Docker CE](https://www.docker.com/community-edition) 的產品)

Windows 10 上的 Docker for Windows (適用於 Docker CE 引擎的 Windows 驅動程式) 將會使用名為 'DockerNAT' 的內部 vSwitch，來將 Moby Linux VM 連接至容器主機。 在 Windows 上使用 Moby Linux VM 的開發人員應注意，其主機是使用 DockerNAT vSwitch，而不是 HNS 服務建立的「nat」vSwitch (用於 Windows 容器的預設交換器)。



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
* 另一個選項是使用 '-o com.docker.network.windowsshim.interface' 選項將透明網路的外部 vSwitch 繫結至容器主機上尚未使用的特定網路介面卡 (亦即不是頻外建立的 vSwitch 所使用的網路介面卡)。 此檔的 [在[單一容器主機上建立多個透明網路](advanced.md#creating-multiple-transparent-networks-on-a-single-container-host)] 區段中的 [-o] 選項會進一步說明。


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