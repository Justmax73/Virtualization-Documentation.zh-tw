---
title: 關於 Windows 容器協調器
description: 深入了解 Windows 容器協調器。
keywords: Docker, 容器
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 1ccf63b0ae55501ba32f8bdd61994e7f8006b5e6
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674877"
---
# <a name="about-windows-container-orchestrators"></a>關於 Windows 容器協調器

由於他們的小的大小和應用程式的方向，容器都是最適合用於靈活傳遞的環境和微服務型架構。 不過，使用容器和 microservers 的環境可以有數百或數千個元件，以追蹤。 您可以手動管理幾十個虛擬機器或實體伺服器，但沒有任何方式來適當地管理生產規模的容器環境不使用自動化。 您的協調器，也就是處理程序會自動執行和管理大量的容器，以及它們如何與彼此互動應該切換這項工作。

協調器執行下列工作：

- 排程： 指定容器映像和資源要求、 時 orchestrator 尋找適合的電腦來執行容器。
- 親和性/反親和性： 指定是否執行一組容器應該彼此相近效能或距離的可用性。
- 健康情況監視：監看容器的失敗事件，並自動重新安排容器排程。
- 容錯移轉： 追蹤每部電腦上執行什麼，並重新安排容器失敗電腦連接至正常的節點。
- 縮放比例： 新增或移除容器執行個體，以手動或自動符合需求。
- 網路功能： 提供覆疊網路來協調容器跨多個主機電腦進行通訊。
- 服務探索：讓容器即使在主機電腦間移動並變更 IP 位址也能夠自動找到彼此。
- 協調應用程式升級：管理容器升級以避免應用程式服務中斷，並在發生錯誤時能夠復原。

## <a name="orchestrator-types"></a>Orchestrator 類型

Azure 提供兩種容器協調器： Azure Kubernetes Service (AKS) 和 Service Fabric。

[Azure Kubernetes Service (AKS)](/azure/aks/)可讓您更容易建立、 設定及管理虛擬機器來執行容器化應用程式預先設定的叢集。 這可讓您使用您現有的技術與運用大量不斷增長的社群專業知識，以部署和管理容器型應用程式在 Microsoft Azure 上的繪製。 藉由使用 AKS，您可以利用 Azure 企業級功能的同時仍維持應用程式可攜性，透過 Kubernetes 和 Docker 映像格式。

[Azure Service Fabric](/azure/service-fabric/) 是一種分散式系統平台，可讓您輕鬆封裝、部署及管理可調整且可靠的微服務和容器。 Service Fabric 可解決開發及管理雲端原生應用程式時的重要問題。 開發人員和系統管理員可以避免複雜的基礎結構問題，將重心放在實作具有任務關鍵性而需求嚴苛並且是可調整、可靠及可管理的工作負載。 Service Fabric 代表新一代的平台，適用於建置及管理這些在容器中執行的企業級、第 1 層、雲端規模的應用程式。

## <a name="getting-started"></a>開始使用

若要開始部署 Azure Kubernetes 服務，請參閱[Kubernetes 設定指南](../kubernetes/getting-started-kubernetes-windows.md)。

若要開始部署 Azure Service Fabric，請參閱[Service Fabric 快速入門](/azure/service-fabric/service-fabric-quickstart-containers.md)。