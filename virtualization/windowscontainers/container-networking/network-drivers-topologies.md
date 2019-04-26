---
title: Windows 容器網路功能
description: Windows 容器的網路驅動程式和拓撲。
keywords: Docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: e2b3c05a35896d51b1fbd1bf3f276791e4e08493
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "9577119"
---
# <a name="windows-container-network-drivers"></a>Windows 容器網路驅動程式  

使用者除了可以在 Windows 上運用 Docker 建立的預設 'nat' 網路，也可以定義自訂容器網路。 使用者可以使用 Docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) 命令來建立使用者定義的網路。 在 Windows 上，可使用下列網路驅動程式類型：

- **nat** – 如果容器連接到以 'nat' 驅動程式建立的網路，其將會連接到*內部* Hyper-V 交換器並從使用者指定的 (``--subnet``) IP 首碼收到 IP 位址。 支援從容器主機到容器端點的連接埠轉送/對應。
  
  >[!NOTE]
  >如果您已安裝在 Windows 10 Creators Update，支援多個 NAT 網路。

- **transparent** – 如果容器連接到以 'transparent' 驅動程式建立的網路，其將會透過*外部* Hyper-V 交換器直接連接到實體網路。 實體網路的 IP 可以透過外部 DHCP 伺服器，以靜態 (需要使用者指定的 ``--subnet`` 選項) 或動態方式指派。
  
  >[!NOTE]
  >在下列需求，因為連線透明網路上的容器主機不支援 Azure vm。
  
  > 需要： 當此模式用於虛擬化案例 （容器主機是 VM） _MAC 位址詐騙是必要_。

- **overlay** - 當 Docker 引擎以[群集模式](../manage-containers/swarm-mode.md)執行時，連接到重疊網路的容器可以與連接到相同網路的其他容器跨多個容器主機進行通訊。 建立在群集叢集上的每個重疊網路，在建立時即具有其本身的 IP 子網路 (由私人 IP 首碼定義)。 overlay 網路驅動程式會使用 VXLAN 封裝。 **使用適當的網路控制台 (Flannel 或 OVN) 可以搭配 Kubernetes。**
  > 需要： 請確定您的環境滿足這些所需的[必要條件](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks)的建立覆疊網路。

  > 需要： 需要 Windows Server 2016 [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217)、 Windows 10 Creators Update 或更新版本。

  >[!NOTE]
  >在 Windows Server 2019 執行 Docker EE 18.03 和更新，由 Docker 群集建立覆疊網路運用 VFP NAT 規則的輸出的連線。 這表示 thata 指定容器會收到 1 的 IP 位址。 這也表示，ICMP 型工具，例如`ping`或`Test-NetConnection`應該設定為使用他們的 TCP/UDP 選項中偵錯的情況。

- **l2bridge** - 連接到以 'l2bridge' 驅動程式建立之網路的容器，會位於與容器主機相同的 IP 子網路中，並透過*外部* Hyper-V 交換器連接到實體網路。 必須以靜態方式從與容器主機相同的首碼指派 IP 位址。 因為在輸入和輸出時都有第 2 層位址轉譯 (MAC 重寫) 作業的關係，主機上的所有容器端點都會有跟主機相同的 MAC 位址。
  > 需要： 當此模式用於虛擬化案例 （容器主機是 VM） _MAC 位址詐騙是必要_。
  
  > 需要： 需要 Windows Server 2016、 Windows 10 Creators Update 或更新版本。

- **l2tunnel** -與 l2bridge 類似，不過_此驅動程式應該只用於 Microsoft 雲端堆疊，例如 Azure_。 來自容器的封包會傳送至套用 SDN 原則的虛擬化主機。

## <a name="network-topologies-and-ipam"></a>網路拓撲和 IPAM

下表顯示如何針對每一種網路驅動程式的內部 (容器對容器) 和外部連接提供網路連線。

### <a name="networking-modesdocker-drivers"></a>網路功能模式/Docker 驅動程式

  | Docker Windows 網路驅動程式 | 典型的 os | 容器至容器 （單一節點） | 容器-至外部 （單一節點 + 多節點） | 容器至容器 （多節點） |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (預設)** | 適用於開發人員 | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li> 跨子網路： 不支援 （只有一個 NAT 內部首碼）</li></ul> | 透過管理 vNIC 路由 (繫結至 WinNAT) | 不直接支援：需要透過主機公開連接埠 |
  | **透明** | 適用於開發人員或小型部署 | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li>跨子網路：透過容器主機路由</li></ul> | 透過容器主機以及直接存取 (實體) 網路介面卡來路由 | 透過容器主機以及直接存取 (實體) 網路介面卡來路由 |
  | **Overlay** | 適用於多節點;所需的 Docker、 群集、 在 Kubernetes 中可用 | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li>跨子網路：網路流量透過 Mgmt vNIC 封裝和路由</li></ul> | 不直接支援 - 需要將第二個容器端點連接到 NAT 網路 | 相同/跨子網路：使用 VXLAN 封裝網路流量，並透過 Mgmt vNIC 路由 |
  | **L2Bridge** | 用於 Kubernetes 與 Microsoft SDN | <ul><li>相同子網路：透過 Hyper-V 虛擬交換器橋接的連線</li><li> 跨子網路：容器 MAC 位址在輸入和輸出時會重寫並路由</li></ul> | 容器 MAC 位址在輸入和輸出時會重寫 | <ul><li>相同子網路：橋接的連線</li><li>跨子網路： 透過 Mgmt vNIC WSv1709 和更新版本所路由</li></ul> |
  | **L2Tunnel**| 僅限 Azure | 相同/跨子網路：釘選到套用原則之實體主機的 Hyper-V 虛擬交換器 | 流量必須通過 Azure 虛擬網路閘道 | 相同/跨子網路：釘選到套用原則之實體主機的 Hyper-V 虛擬交換器 |

### <a name="ipam"></a>IPAM

針對每一種網路驅動程式，配置及指派 IP 位址的方式各不相同。 Windows 會使用主機網路服務 (HNS) 來為 nat 驅動程式提供 IPAM，並以 Docker 群集模式 (內部 KVS) 來為 overlay 驅動程式提供 IPAM。 所有其他網路驅動程式皆使用外部 IPAM。

| 網路模式/驅動程式 | IPAM |
| -------------------------|:----:|
| NAT | 動態 IP 配置並從內部 NAT 子網路首碼指派由主機網路服務 (HNS) |
| 透明 | 靜態或動態 (使用外部 DHCP 伺服器) IP 配置並從容器主機網路首碼中的 IP 位址指派 |
| 重疊 | 來自 Docker 引擎群集模式受管理首碼的動態 IP 配置並透過 HNS 指派 |
| L2Bridge | 靜態 IP 配置並從容器主機網路首碼 （也可指派透過 HNS） 中的 IP 位址指派 |
| L2Tunnel | 僅限 Azure - 動態 IP 配置並透過外掛程式指派 |

### <a name="service-discovery"></a>服務探索

只有某些 Windows 網路驅動程式可支援服務探索。

|  | 本機服務探索  | 全域服務探索 |
| :---: | :---------------     |  :---                |
| nat | 是 | 是，使用 Docker EE |  
| overlay | 是 | 是，使用 Docker EE 或 kube dns |
| 透明 | NO | 否 |
| l2bridge | 否 | \ [是，使用 kube-dns |
