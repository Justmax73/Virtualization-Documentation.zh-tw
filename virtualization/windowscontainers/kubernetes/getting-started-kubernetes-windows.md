---
title: Windows 上的 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 將 Windows 節點加入具有 v 1.14 的 Kubernetes 群集。
keywords: kubernetes、1.14、windows、快速入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 18734f102042ec951255061dcd82229e18d29a15
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998385"
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes

此頁面可讓您將 Windows 節點加入 Linux 群集, 以快速開始使用 Windows 上的 Kubernetes。 在 Windows Server[版本 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)上發行 Kubernetes 1.14, 使用者可以利用 Windows 版 Kubernetes 中的下列功能:

- **覆蓋網路**: 在 vxlan 模式中使用 Flannel 來設定虛擬重迭網路
    - 需要安裝裝有[KB4489899](https://support.microsoft.com/help/4489899)的 Windows Server 2019 或[windows server VNext](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/)測試人員預覽版 18317 +
    - 需要啟用功能入口的`WinOverlay` Kubernetes v 1.14 (或更新版本)
    - 需要 Flannel v 0.11.0 (或更新版本)
- **簡化網路管理**: 在主版中使用 Flannel, 以在節點間自動路由管理。
- **伸縮性**改良: 感謝您[Deviceless vNICs Windows Server 容器](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716), 享受更快速且更可靠的容器啟動時間。
- **Hyper-v 隔離 (Alpha)**: 使用核心模式隔離來協調[hyper-v 隔離](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers), 以提高安全性。 如需詳細資訊, 請閱讀[Windows 容器類型](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types)。
    - 需要啟用功能入口的`HyperVContainer` Kubernetes v 1.10 (或更新版本)。
- **儲存外掛程式**: 將[FlexVolume 儲存外掛程式](https://github.com/Microsoft/K8s-Storage-Plugins)搭配 Windows 樹枝的 SMB 和 iSCSI 支援使用。

>[!TIP]
>如果您想要在 Azure 上部署群集, 「開放來源 AKS 引擎」工具就能輕鬆實現這個功能。 若要深入瞭解, 請參閱我們的逐步逐步[練習](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)。

## <a name="prerequisites"></a>必要條件

### <a name="plan-ip-addressing-for-your-cluster"></a>規劃群集的 IP 位址

<a name="definitions"></a>當 Kubernetes 群集為盒式和服務推出新的子網時, 請務必確保這些子網不會與您環境中的任何其他現有網路發生衝突。 以下是必須釋放才能成功部署 Kubernetes 的所有位址空間:

| 子網/位址範圍 | 描述 | 預設值 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**服務子網上** | 在沒有網路拓撲的 caring 情況下, 由盒用來 uniformally access 服務的無法路由的純粹虛擬子網。 節點上執行的 `kube-proxy` 可在服務子網路與可路由的位址空間之間來回轉譯。 | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**群集子網上** |  這個全域子網是由群集中的所有箱所使用。 會為每個節點指派一個較小/24 個子網上的子網, 以便供其盒用完。 它必須足夠大, 才能容納您的群集中使用的所有盒。 若要計算*最小*的子網大小: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>每個節點100盒的5節點群集範例: `(5) + (5 *  100) = 505`。  | "10.244.0.0/16" |
| **Kubernetes DNS 服務 IP** | "Kube-dns" 服務的 IP 位址, 將用於 DNS 解析 & 叢集服務探索。 | "10.96.0.10" |

> [!NOTE]
> 有另一個 Docker 網路 (NAT) 是在安裝 Docker 時預設建立的。 當我們改為從群集子網上指派 Ip 時, 不需要在 Windows 上執行 Kubernetes。

## <a name="what-you-will-accomplish"></a>您將會完成

本指南結束時，您將會：

> [!div class="checklist"]
> * 已建立[Kubernetes 主](./creating-a-linux-master.md)節點。  
> * 已選取[網路方案](./network-topologies.md)。  
> * 已將[Windows worker 節點](./joining-windows-workers.md)或[Linux 工作節點](./joining-linux-workers.md)加入到其中。  
> * 已部署[範例 Kubernetes 資源](./deploying-resources.md)。  
> * 涵蓋[常見問題與錯誤](./common-problems.md)。

## <a name="next-steps"></a>後續步驟

在本節中, 我們會討論在 Windows 上成功部署 Kubernetes 所需的重要必備元件 & 假設。 繼續瞭解如何設定 Kubernetes 主版:

>[!div class="nextstepaction"]
>[建立 Kubernetes 主版](./creating-a-linux-master.md)