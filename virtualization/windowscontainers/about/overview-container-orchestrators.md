---
title: Windows 容器 Orchestration 概述
description: 瞭解 Windows 容器 orchestrators。
keywords: Docker, 容器
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 99a3b47a9d80e21c246fb3b4f61d650557eb37fa
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161785"
---
# <a name="windows-container-orchestration-overview"></a>Windows 容器 Orchestration 概述

由於其大小和應用程式的規模小，所以容器非常適合敏捷傳遞環境和 microservice 的架構。 不過，使用容器與 microservers 的環境可以有幾百或上千個元件來保持追蹤。 您可能可以手動管理幾個虛擬電腦或物理伺服器，但無法正確管理不含自動化的生產規模容器環境。 此工作應該是在您的 orchestrator 中，這是一種自動化及管理大量容器以及它們彼此互動的方式。

Orchestrators 執行下列任務：

- 排程：已知容器圖像與資源要求時，orchestrator 會尋找適合執行容器的電腦。
- 關聯性/反關聯：指定是否要讓一組容器彼此接近效能，或針對可用性進行。
- 健康情況監視：監看容器的失敗事件，並自動重新安排容器排程。
- 容錯移轉：追蹤每個電腦上執行的專案，並將容器從失敗的電腦重新安排到健康節點。
- 縮放：根據需求、手動或自動來新增或移除容器實例。
- 網路：提供重迭的網路來協調容器，在多個主機電腦之間通訊。
- 服務探索：讓容器即使在主機電腦間移動並變更 IP 位址也能夠自動找到彼此。
- 協調應用程式升級：管理容器升級以避免應用程式服務中斷，並在發生錯誤時能夠復原。

## <a name="orchestrator-types"></a>Orchestrator 類型

Azure 提供兩個容器 orchestrators： Azure Kubernetes 服務（AKS）和 Service Fabric。

[Azure Kubernetes Service （AKS）](/azure/aks/)可讓您輕鬆建立、設定及管理預先設定的虛擬機器群集，以執行容器化的應用程式。 這可讓您使用現有的技能，並在不斷增長的社區專業知識中進行起草，以在 Microsoft Azure 上部署和管理容器式應用程式。 透過使用 AKS，您可以利用 Azure 的企業級功能，同時仍能透過 Kubernetes 和 Docker 影像格式維持應用程式可攜性。

[Azure Service Fabric](/azure/service-fabric/) 是一種分散式系統平台，可讓您輕鬆封裝、部署及管理可調整且可靠的微服務和容器。 Service Fabric 可解決開發及管理雲端原生應用程式時的重要問題。 開發人員和系統管理員可以避免複雜的基礎結構問題，將重心放在實作具有任務關鍵性而需求嚴苛並且是可調整、可靠及可管理的工作負載。 Service Fabric 代表新一代的平台，適用於建置及管理這些在容器中執行的企業級、第 1 層、雲端規模的應用程式。

## <a name="getting-started"></a>開始使用

若要開始部署 Azure Kubernetes 服務，請參閱[Kubernetes 安裝指南](../kubernetes/getting-started-kubernetes-windows.md)。

若要開始部署 Azure Service Fabric，請參閱[Service Fabric 快速入門](/azure/service-fabric/service-fabric-quickstart-containers.md)。