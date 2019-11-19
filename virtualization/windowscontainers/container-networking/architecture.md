---
title: Windows 容器網路
description: 簡單介紹 Windows 容器網路的架構。
keywords: Docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: e9d4a9ac88c6853ce019a2469ee80688490b8fdf
ms.sourcegitcommit: bb4ec1f05921f982c00bdb3ace6d9bc1d5355296
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/18/2019
ms.locfileid: "10297239"
---
# <a name="windows-container-networking"></a>Windows 容器網路

>[!IMPORTANT]
>請參閱適用于一般 Docker 網路命令、選項及語法的[Docker 容器網路](https://docs.docker.com/engine/userguide/networking/)。 * * * 除了[不支援的功能和網路選項](#unsupported-features-and-network-options)中所述的任何情況以外，在 Windows 上支援所有 Docker 網路命令，其語法與 Linux 相同。 不過，Windows 和 Linux 網路堆疊是不一樣的，因此您會發現 Windows 不支援某些 Linux 網路命令（例如，ifconfig）。

## <a name="basic-networking-architecture"></a>基本網路架構

本主題概述 Docker 如何在 Windows 上建立及管理主機網路。 Windows 容器在網路功能方面類似虛擬機器。 每個容器都有連接到 Hyper-V 虛擬交換器 (vSwitch) 的虛擬網路介面卡 (vNIC)。 Windows 支援可透過 Docker 來建立的五種不同[網路驅動程式或模式](./network-drivers-topologies.md)：*nat*、*overlay*、*transparent*、*l2bridge* 和 *l2tunnel*。 根據您的實體網路基礎結構和單一與多主機網路功能需求，您應該選擇最符合需求的網路驅動程式。

![文字](media/windowsnetworkstack-simple.png)

Docker 引擎第一次執行時，會建立預設 NAT 網路 'nat'，它會使用內部 vSwitch 和名為 `WinNAT` 的 Windows 元件。 如果主機上有透過 PowerShell 或 Hyper-V 管理員來建立的任何既有外部 vSwitch，Docker 也可以透過 *transparent* 網路驅動程式來使用，而當您執行 ``docker network ls`` 命令時，也會看到這些 vSwitch。  

![文字](media/docker-network-ls.png)

- **內部**vSwitch 是指未直接連線到容器主機上的網路介面卡的情況。
- **外部**vSwitch 是直接連接至容器主機上的網路介面卡的一個。

![文字](media/get-vmswitch.png)

'nat' 網路是在 Windows 上執行的容器的預設網路。 任何容器如果在 Windows 上執行，但沒有任何旗標或引數實作特定網路組態，其將會連接到預設的 'nat' 網路，且系統會自動從 'nat' 網路的內部首碼 IP 範圍指派 IP 位址給容器。 用於 'nat' 的預設內部 IP 首碼為 172.16.0.0/16。 

## <a name="container-network-management-with-host-network-service"></a>使用主機網路服務進行容器網路管理

主機網路服務 (HNS) 和主機運算服務 (HCS) 共同建立容器，並將端點連接至網路。

### <a name="network-creation"></a>建立網路

- HNS 為每個網路建立 Hyper-V 虛擬交換器
- HNS 建立所需的 NAT 及 IP 集區

### <a name="endpoint-creation"></a>端點建立

- HNS 為每個容器端點建立網路命名空間
- HNS/HCS 將 v(m)NIC 放在網路命名空間中
- HNS 建立 (vSwitch) 連接埠
- HNS 指派 IP 位址、DNS 資訊、路由等 (受制於網路模式) 給端點

### <a name="policy-creation"></a>原則建立

- 預設 NAT 網路：HNS 使用對應的 Windows 防火牆允許規則，建立 WinNAT 連接埠轉送規則/對應
- 所有其他網路：HNS 利用虛擬篩選平台 (VFP) 來建立原則
    - 這包括：負載平衡、ACL、封裝等。
    - 尋找我們[在此](https://docs.microsoft.com/en-us/windows-server/networking/technologies/hcn/hcn-top)發佈的 HNS api 與架構

![文字](media/HNS-Management-Stack.png)

## <a name="unsupported-features-and-network-options"></a>不支援的功能和網路選項

Windows 目前**不**支援下列網路選項：

- 附加至 l2bridge、NAT 及重迭網路的 Windows 容器不支援經由 IPv6 堆疊進行通訊。
- 經 IPsec 加密的容器通訊。
- 容器的 HTTP proxy 支援。
- [主機模式](https://docs.docker.com/ee/ucp/interlock/config/host-mode-networking/)網路 
- 透過透明網路驅動程式在虛擬 Azure 基礎結構上進行網路連接。

| 命令        | 不支援的選項   |
|---------------|:--------------------:|
| ``docker run``|   ``--ip6``, ``--dns-option`` |
| ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |
