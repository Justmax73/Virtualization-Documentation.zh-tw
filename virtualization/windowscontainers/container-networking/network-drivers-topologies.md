---
title: Windows 容器網路功能
description: Windows 容器的網路驅動程式和拓撲。
keywords: docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 46eefb03f8f5a53333f5e7eca7074ab34e72a767
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910068"
---
# <a name="windows-container-network-drivers"></a>Windows 容器網路驅動程式  

使用者除了可以在 Windows 上運用 Docker 建立的預設 'nat' 網路，也可以定義自訂容器網路。 您可以使用 Docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/)命令來建立使用者定義的網路。 在 Windows 上，可使用下列網路驅動程式類型：

- **nat** – 如果容器連接到以 'nat' 驅動程式建立的網路，其將會連接到*內部* Hyper-V 交換器並從使用者指定的 (``--subnet``) IP 首碼收到 IP 位址。 支援從容器主機到容器端點的連接埠轉送/對應。
  
  >[!NOTE]
  > 重新開機之後，在 Windows Server 2019 （或更新版本）上建立的 NAT 網路將不再保存。

  > 如果您已安裝 Windows 10 建立者更新（或以上版本），則支援多個 NAT 網路。
  
- **transparent** – 如果容器連接到以 'transparent' 驅動程式建立的網路，其將會透過*外部* Hyper-V 交換器直接連接到實體網路。 實體網路的 IP 可以透過外部 DHCP 伺服器，以靜態 (需要使用者指定的 ``--subnet`` 選項) 或動態方式指派。
  
  >[!NOTE]
  >基於下列需求，Azure Vm 不支援透過透明網路連接您的容器主機。
  
  > 需要：在虛擬化案例（容器主機是 VM）中使用此模式時 _，必須要有 MAC 位址_詐騙。

- **overlay** - 當 Docker 引擎以[群集模式](../manage-containers/swarm-mode.md)執行時，連接到重疊網路的容器可以與連接到相同網路的其他容器跨多個容器主機進行通訊。 建立在群集叢集上的每個覆疊網路，在建立時即具有其本身的 IP 子網路 (由私人 IP 首碼定義)。 overlay 網路驅動程式會使用 VXLAN 封裝。 **使用適當的網路控制平面（例如 Flannel）時，可以與 Kubernetes 搭配使用。**
  > 需要：請確定您的環境符合建立重迭[網路所需的必要條件。](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks)

  > 需要：在 Windows Server 2019 上，這需要[KB4489899](https://support.microsoft.com/help/4489899)。

  > 需要：在 Windows Server 2016 上，這需要[KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)。

  >[!NOTE]
  >在 Windows Server 2019 上，Docker Swarm 所建立的重迭網路會利用 VFP NAT 規則來輸出連線能力。 這表示指定的容器會收到1個 IP 位址。 這也表示以 ICMP 為基礎的工具（例如 `ping` 或 `Test-NetConnection`）應該使用其在偵錯工具中的 TCP/UDP 選項進行設定。

- **l2bridge** -與 `transparent` 網路模式類似，連接到以 ' l2bridge ' 驅動程式建立的網路的容器將會透過*外部*hyper-v 交換器連接到實體網路。 L2bridge 的差異在於，容器端點會有與主機相同的 MAC 位址，因為輸入和輸出的第2層位址轉譯（MAC 重新寫入）作業。 在叢集案例中，這有助於減輕交換器的壓力，而必須瞭解有時候短期容器的 MAC 位址。 L2bridge 網路可透過兩種不同的方式來設定：
  1. 已使用與容器主機相同的 IP 子網來設定 L2bridge 網路
  2. 已使用新的自訂 IP 子網設定 L2bridge 網路
  
  在 configuration 2 中，使用者必須在做為閘道的主機網路區間上新增端點，並為指定的前置詞設定路由功能。 
  > 需要：需要 Windows Server 2016、Windows 10 建立者更新或更新版本。

  > 需要：外部連線的[OutboundNAT 原則](./advanced.md#specify-outboundnat-policy-for-a-network)。

- **l2tunnel** -類似于 l2bridge，但_此驅動程式只應用於 Microsoft Cloud 堆疊（Azure）_ 。 來自容器的封包會傳送至套用 SDN 原則的虛擬化主機。


## <a name="network-topologies-and-ipam"></a>網路拓撲和 IPAM

下表顯示如何針對每一種網路驅動程式的內部 (容器對容器) 和外部連接提供網路連線。

### <a name="networking-modesdocker-drivers"></a>網路模式/Docker 驅動程式

  | Docker Windows 網路驅動程式 | 典型的使用案例 | 容器對容器（單一節點） | 容器對外部（單一節點 + 多節點） | 容器對容器（多節點） |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT （預設值）** | 適用於開發人員 | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li> 跨子網：不支援（僅限一個 NAT 內部首碼）</li></ul> | 透過管理 vNIC 路由 (繫結至 WinNAT) | 不直接支援：需要透過主機公開連接埠 |
  | **變成** | 適用於開發人員或小型部署 | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li>跨子網路：透過容器主機路由</li></ul> | 透過容器主機以及直接存取 (實體) 網路介面卡來路由 | 透過容器主機以及直接存取 (實體) 網路介面卡來路由 |
  | **Overlay** | 適用于多節點;Docker Swarm 的必要項（可在 Kubernetes 中取得） | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li>跨子網路：網路流量透過 Mgmt vNIC 封裝和路由</li></ul> | 不直接支援-需要在 Windows Server 2016 上連接到 NAT 網路的第二個容器端點，或在 Windows Server 2019 上 VFP NAT 規則。  | 相同/跨子網路：使用 VXLAN 封裝網路流量，並透過 Mgmt vNIC 路由 |
  | **L2Bridge** | 用於 Kubernetes 與 Microsoft SDN | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li> 跨子網路：容器 MAC 位址在輸入和輸出時會重寫並路由</li></ul> | 容器 MAC 位址在輸入和輸出時會重寫 | <ul><li>相同子網路：橋接的連線</li><li>跨子網：透過 WSv1809 和更新版本上的管理 vNIC 路由傳送</li></ul> |
  | **L2Tunnel**| 僅限 Azure | 相同/跨子網路：釘選到套用原則之實體主機的 Hyper-V 虛擬交換器 | 流量必須通過 Azure 虛擬網路閘道 | 相同/跨子網路：釘選到套用原則之實體主機的 Hyper-V 虛擬交換器 |

### <a name="ipam"></a>IPAM

針對每一種網路驅動程式，配置及指派 IP 位址的方式各不相同。 Windows 會使用主機網路服務 (HNS) 來為 nat 驅動程式提供 IPAM，並以 Docker 群集模式 (內部 KVS) 來為 overlay 驅動程式提供 IPAM。 所有其他網路驅動程式皆使用外部 IPAM。

| 網路模式/驅動程式 | IPAM |
| -------------------------|:----:|
| NAT | 來自內部 NAT 子網首碼的主機網路服務（HNS）動態 IP 配置和指派 |
| 透明 | 靜態或動態 (使用外部 DHCP 伺服器) IP 配置並從容器主機網路首碼中的 IP 位址指派 |
| Overlay | 來自 Docker 引擎群集模式受管理首碼的動態 IP 配置並透過 HNS 指派 |
| L2Bridge | 從容器主機的網路首碼（也可以透過 HNS 指派）中的 IP 位址進行靜態 IP 配置和指派 |
| L2Tunnel | 僅限 Azure - 動態 IP 配置並透過外掛程式指派 |

### <a name="service-discovery"></a>服務探索

只有某些 Windows 網路驅動程式可支援服務探索。

|  | 本機服務探索  | 全域服務探索 |
| :---: | :---------------     |  :---                |
| nat | 是 | 是，使用 Docker EE |  
| overlay | 是 | 是，使用 Docker EE 或 kube-dns |
| transparent | 否 | 否 |
| l2bridge | 否 | 是，使用 kube-dns |
