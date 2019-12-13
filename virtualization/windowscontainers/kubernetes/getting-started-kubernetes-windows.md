---
title: Windows 上的 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 使用 v 1.14 將 Windows 節點加入至 Kubernetes 叢集。
keywords: kubernetes，1.14，windows，使用者入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c380f5dc10430a94959718a5ce92f311603db733
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910358"
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes

此頁面可讓您透過將 Windows 節點加入 Linux 型叢集的方式，來瞭解如何在 Windows 上開始使用 Kubernetes。 隨著 Windows Server [1809 版](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)的 Kubernetes 1.14 發行，使用者可以利用 Windows 上 Kubernetes 中的下列功能：

- 覆迭**網路**：在 vxlan 模式中使用 Flannel 來設定虛擬重迭網路
    - 需要安裝具有[KB4489899](https://support.microsoft.com/help/4489899)的 windows server 2019 或[Windows Server vNext Insider preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/)組建 18317 +
    - 需要已啟用 `WinOverlay` 功能閘道的 Kubernetes v 1.14 （或更新版本）
    - 需要 Flannel v 0.11.0 （或更新版本）
- **簡化的網路管理**：在主機閘道模式中使用 Flannel，以在節點之間自動路由管理。
- 擴充**功能改進**：由於[Windows Server 容器的 deviceless vnic](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716)，可讓您更快速且更可靠的容器啟動時間。
- **Hyper-v 隔離（Alpha）** ：使用核心模式隔離來協調[hyper-v 隔離](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)，以增強安全性。 如需詳細資訊，請查看[Windows 容器類型](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types)。
    - 需要已啟用 `HyperVContainer` 功能閘道的 Kubernetes v 1.10 （或更新版本）。
- **儲存體外掛程式**：搭配適用于 Windows 容器的 SMB 和 iSCSI 支援來使用[FlexVolume 儲存體外掛程式](https://github.com/Microsoft/K8s-Storage-Plugins)。

>[!TIP]
>如果您想要在 Azure 上部署叢集，開放原始碼 AKS 引擎工具可讓您輕鬆完成。 若要深入瞭解，請[參閱逐步解說。](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)

## <a name="prerequisites"></a>必要條件

### <a name="plan-ip-addressing-for-your-cluster"></a>規劃叢集的 IP 位址

<a name="definitions"></a>當 Kubernetes 叢集引進 pod 和服務的新子網時，請務必確保它們都不會與您環境中的任何其他現有網路衝突。 以下是必須釋放才能成功部署 Kubernetes 的所有位址空間：

| 子網/位址範圍 | 說明 | 預設值 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**服務子網** | 一個不可路由傳送的單純虛擬子網，可供 pod 用來 uniformally 存取服務，而不需管也網路拓朴。 節點上執行的 `kube-proxy` 可在服務子網路與可路由的位址空間之間來回轉譯。 | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**叢集子網** |  這是叢集中所有 pod 使用的全域子網。 每個節點都會被指派一個較小/24 個子網，以供其 pod 使用。 它必須夠大，才能容納叢集中使用的所有 pod。 若要計算「最小」的子網路大小：`(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>適用于每個節點 100 pod 的5個節點叢集範例： `(5) + (5 *  100) = 505`。  | "10.244.0.0/16" |
| **Kubernetes DNS 服務 IP** | 將用於 DNS 解析 & 叢集服務探索的 IP 位址 "kube"。 | "10.96.0.10" |

> [!NOTE]
> 當您安裝 Docker 時，預設會建立另一個 Docker 網路（NAT）。 因為我們會改為從叢集子網指派 Ip，所以不需要在 Windows 上操作 Kubernetes。

## <a name="what-you-will-accomplish"></a>您將會完成

本指南結束時，您將會：

> [!div class="checklist"]
> * 建立[Kubernetes 主要](./creating-a-linux-master.md)節點。  
> * 選取了[網路解決方案](./network-topologies.md)。  
> * 已將[Windows 背景工作節點](./joining-windows-workers.md)或[Linux 背景工作角色節點](./joining-linux-workers.md)加入其中。  
> * 已部署[範例 Kubernetes 資源](./deploying-resources.md)。  
> * 涵蓋[常見問題與錯誤](./common-problems.md)。

## <a name="next-steps"></a>後續步驟

在本節中，我們討論了在今天成功部署 Kubernetes 到 Windows 所需的重要必要條件 & 假設。 繼續瞭解如何設定 Kubernetes 主機：

>[!div class="nextstepaction"]
>[建立 Kubernetes 主機](./creating-a-linux-master.md)