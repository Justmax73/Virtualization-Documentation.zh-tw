---
title: Windows 容器網路
description: Windows 容器的網路驅動程式和拓撲。
keywords: Docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 46eefb03f8f5a53333f5e7eca7074ab34e72a767
ms.sourcegitcommit: bb4ec1f05921f982c00bdb3ace6d9bc1d5355296
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/18/2019
ms.locfileid: "10297249"
---
# <a name="windows-container-network-drivers"></a>Windows 容器網路驅動程式  

使用者除了可以在 Windows 上運用 Docker 建立的預設 'nat' 網路，也可以定義自訂容器網路。 您可以使用 Docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/)命令來建立使用者定義的網路。 在 Windows 上，可使用下列網路驅動程式類型：

- **nat** – 如果容器連接到以 'nat' 驅動程式建立的網路，其將會連接到*內部* Hyper-V 交換器並從使用者指定的 (``--subnet``) IP 首碼收到 IP 位址。 支援從容器主機到容器端點的連接埠轉送/對應。
  
  >[!NOTE]
  > 重新開機之後，在 Windows Server 2019 （或更新版本）上建立的 NAT 網路已不再保留。

  > 如果您已安裝 Windows 10 創意者更新（或更新版本），則支援多個 NAT 網路。
  
- **transparent** – 如果容器連接到以 'transparent' 驅動程式建立的網路，其將會透過*外部* Hyper-V 交換器直接連接到實體網路。 實體網路的 IP 可以透過外部 DHCP 伺服器，以靜態 (需要使用者指定的 ``--subnet`` 選項) 或動態方式指派。
  
  >[!NOTE]
  >根據下列需求，Azure Vm 不支援透過透明網路連接您的容器主機。
  
  > 需要：在虛擬化案例中使用此模式時（容器主機是 VM）_需要 MAC 位址欺騙_。

- **overlay** - 當 Docker 引擎以[群集模式](../manage-containers/swarm-mode.md)執行時，連接到重疊網路的容器可以與連接到相同網路的其他容器跨多個容器主機進行通訊。 建立在群集叢集上的每個重疊網路，在建立時即具有其本身的 IP 子網路 (由私人 IP 首碼定義)。 overlay 網路驅動程式會使用 VXLAN 封裝。 **使用合適的網路控制平面（例如 Flannel）時，可以搭配 Kubernetes 使用。**
  > 需要：請確定您的環境滿足這些必要的[先決條件](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks)，才能建立重迭網路。

  > 需要：在 Windows Server 2019 上，這需要[KB4489899](https://support.microsoft.com/help/4489899)。

  > 需要：在 Windows Server 2016 上，這需要[KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)。

  >[!NOTE]
  >在 Windows Server 2019 上，由 Docker 建立的重迭網路 Swarm 會利用 VFP NAT 規則來取得輸出連線。 這表示指定的容器會收到1個 IP 位址。 這也代表在調試情況中使用 TCP/ `ping` UDP `Test-NetConnection`選項（例如或）來設定 ICMP 的基本工具。

- **l2bridge** -與`transparent`網路模式類似，使用「l2bridge」驅動程式建立之網路的容器，會透過*外部*hyper-v 交換器連線至物理網路。 L2bridge 中的差異是，由於在進入與出口中有一個 Layer-2 位址轉換（MAC 重新寫入）作業，因此容器端點會擁有與主機相同的 MAC 位址。 在群集案例中，這可協助減輕需要瞭解有時短存留期容器之 MAC 位址的交換器壓力。 您可以使用兩種不同的方式來設定 L2bridge 網路：
  1. L2bridge 網路已使用與容器主機相同的 IP 子網進行設定
  2. L2bridge 網路是使用新的自訂 IP 子網進行設定
  
  在配置2中，使用者將需要在作為閘道的主機網路隔離艙上新增端點，並為指定的前置詞設定路由功能。 
  > 需要：需要 Windows Server 2016、Windows 10 創意者更新或更新版本。

  > 需要： [OutboundNAT 原則](./advanced.md#specify-outboundnat-policy-for-a-network)以進行外部連線。

- **l2tunnel** -類似 l2bridge，不過_這個驅動程式只應該在 Microsoft 雲端堆疊（Azure）中使用_。 來自容器的封包會傳送至套用 SDN 原則的虛擬化主機。


## <a name="network-topologies-and-ipam"></a>網路拓撲與 IPAM

下表顯示如何針對每一種網路驅動程式的內部 (容器對容器) 和外部連接提供網路連線。

### <a name="networking-modesdocker-drivers"></a>網路模式/Docker 驅動程式

  | Docker Windows 網路驅動程式 | 典型用途 | 容器對容器（單一節點） | 容器對外部（單節點 + 多重節點） | 容器對容器（多重節點） |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (預設)** | 適用於開發人員 | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li> 交叉子網上：不支援（只有一個 NAT 內部前置詞）</li></ul> | 透過管理 vNIC 路由 (繫結至 WinNAT) | 不直接支援：需要透過主機公開連接埠 |
  | **透明** | 適用於開發人員或小型部署 | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li>跨子網路：透過容器主機路由</li></ul> | 透過容器主機以及直接存取 (實體) 網路介面卡來路由 | 透過容器主機以及直接存取 (實體) 網路介面卡來路由 |
  | **Overlay** | 適合多重節點;適用于 Docker Swarm 的需要，可在 Kubernetes 中使用 | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li>跨子網路：網路流量透過 Mgmt vNIC 封裝和路由</li></ul> | 不直接支援-在 windows server 2016 上需要將第二個容器端點附加至 Windows Server 上的 NAT 網路，或 VFP NAT 2019 規則。  | 相同/跨子網路：使用 VXLAN 封裝網路流量，並透過 Mgmt vNIC 路由 |
  | **L2Bridge** | 用於 Kubernetes 與 Microsoft SDN | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li> 跨子網路：容器 MAC 位址在輸入和輸出時會重寫並路由</li></ul> | 容器 MAC 位址在輸入和輸出時會重寫 | <ul><li>相同子網路：橋接的連線</li><li>交叉子網上：透過 WSv1809 及更新版本的 [管理] vNIC 路由</li></ul> |
  | **L2Tunnel**| 僅限 Azure | 相同/跨子網路：釘選到套用原則之實體主機的 Hyper-V 虛擬交換器 | 流量必須通過 Azure 虛擬網路閘道 | 相同/跨子網路：釘選到套用原則之實體主機的 Hyper-V 虛擬交換器 |

### <a name="ipam"></a>IPAM

針對每一種網路驅動程式，配置及指派 IP 位址的方式各不相同。 Windows 會使用主機網路服務 (HNS) 來為 nat 驅動程式提供 IPAM，並以 Docker 群集模式 (內部 KVS) 來為 overlay 驅動程式提供 IPAM。 所有其他網路驅動程式皆使用外部 IPAM。

| 網路模式/驅動程式 | IPAM |
| -------------------------|:----:|
| NAT | 由主機網路服務（HNS）從內部 NAT 子網首碼開始的動態 IP 分配與作業 |
| 透明 | 靜態或動態 (使用外部 DHCP 伺服器) IP 配置並從容器主機網路首碼中的 IP 位址指派 |
| 重疊 | 來自 Docker 引擎群集模式受管理首碼的動態 IP 配置並透過 HNS 指派 |
| L2Bridge | 在容器主機的網路首碼中，從 IP 位址進行靜態 IP 分配與作業指派（也可以透過 HNS 指定） |
| L2Tunnel | 僅限 Azure - 動態 IP 配置並透過外掛程式指派 |

### <a name="service-discovery"></a>服務探索

只有某些 Windows 網路驅動程式可支援服務探索。

|  | 本機服務探索  | 全域服務探索 |
| :---: | :---------------     |  :---                |
| nat | 是 | 是，使用 Docker EE |  
| overlay | 是 | 是使用 Docker EE 或 kube-dns |
| 透明 | NO | 否 |
| l2bridge | 否 | 是使用 kube-dns |
