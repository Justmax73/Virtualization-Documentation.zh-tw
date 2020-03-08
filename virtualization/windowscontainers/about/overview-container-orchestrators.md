---
title: Windows 容器協調流程總覽
description: 瞭解 Windows container 協調器。
keywords: Docker, 容器
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853903"
---
# <a name="windows-container-orchestration-overview"></a>Windows 容器協調流程總覽

由於容器的大小和應用程式方向很小，因此適用于 agile 傳遞環境和以微服務為基礎的架構。 不過，使用容器和微服務的環境可以有數百或數千個元件來追蹤。 您或許可以手動管理數十部虛擬機器或實體伺服器，但沒有任何方法可以適當地管理生產級別的容器環境，而不需要自動化。 這項工作應該落在您的 orchestrator 中，這是一種程式，可自動化和管理大量容器，以及它們彼此互動的方式。

協調器會執行下列工作：

- 排程：當提供容器映射和資源要求時，orchestrator 會尋找適合執行容器的電腦。
- 親和性/反親和性：指定一組容器是否應該在彼此附近執行，以達到可用性的效能或距離。
- 健康情況監視：監看容器的失敗事件，並自動重新安排容器排程。
- 容錯移轉：追蹤每部電腦上執行的內容，並將容器從失敗的電腦重新排程到狀況良好的節點。
- 調整：新增或移除容器實例以符合需求、手動或自動。
- 網路功能：提供重迭的網路，以協調要在多部主機電腦之間通訊的容器。
- 服務探索：讓容器即使在主機電腦間移動並變更 IP 位址也能夠自動找到彼此。
- 協調應用程式升級：管理容器升級以避免應用程式服務中斷，並在發生錯誤時能夠復原。

## <a name="orchestrator-types"></a>Orchestrator 類型

Azure 提供兩個容器協調器： Azure Kubernetes Service （AKS）和 Service Fabric。

[Azure Kubernetes Service （AKS）](/azure/aks/)可讓您輕鬆建立、設定和管理預先設定為執行容器化應用程式的虛擬機器叢集。 這可讓您使用現有的技能，並在大量且不斷成長的社區專業知識中進行繪製，以在 Microsoft Azure 上部署和管理容器型應用程式。 藉由使用 AKS，您可以利用 Azure 的企業級功能，同時仍能透過 Kubernetes 和 Docker 映射格式來維護應用程式可攜性。

[Azure Service Fabric](/azure/service-fabric/) 是一種分散式系統平台，可讓您輕鬆封裝、部署及管理可調整且可靠的微服務和容器。 Service Fabric 可解決開發及管理雲端原生應用程式時的重要問題。 開發人員和系統管理員可以避免複雜的基礎結構問題，將重心放在實作具有任務關鍵性而需求嚴苛並且是可調整、可靠及可管理的工作負載。 Service Fabric 代表新一代的平台，適用於建置及管理這些在容器中執行的企業級、第 1 層、雲端規模的應用程式。

## <a name="getting-started"></a>開始使用

若要開始部署 Azure Kubernetes service，請參閱[Kubernetes 安裝指南](../kubernetes/getting-started-kubernetes-windows.md)。

若要開始部署 Azure Service Fabric，請參閱[Service Fabric 快速入門](/azure/service-fabric/service-fabric-quickstart-containers.md)。
