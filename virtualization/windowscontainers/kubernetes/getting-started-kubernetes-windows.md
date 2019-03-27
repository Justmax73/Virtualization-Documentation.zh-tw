---
title: Windows 上的 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 您可以將 Windows 節點加入 v1.13 Kubernetes 叢集。
keywords: kubernetes，1.13，windows，開始使用
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: df3185db086e8e38143fe60d90db864038980603
ms.sourcegitcommit: 1715411ac2768159cd9c9f14484a1cad5e7f2a5f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/26/2019
ms.locfileid: "9263485"
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes #
此頁面做為概略說明適用於開始使用 Windows 上的 Kubernetes 藉由將 Windows 節點加入 Linux 型叢集。 透過 Windows Server[版本 1809年](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)上的 Kubernetes 1.14 發行，使用者可以利用下列功能在 Kubernetes 中在 Windows 上：

  - **覆疊網路功能**： 使用 vxlan 模式中的 Flannel 設定虛擬覆疊網路
    - 使用[KB4489899](https://support.microsoft.com/en-us/help/4489899)安裝或[Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/)組建 18317 + 需要任一個 Windows Server 2019
    - 需要 Kubernetes v1.14 （或更新版本） 搭配`WinOverlay`啟用功能閘道
    - 需要 Flannel v0.11.0 （或更新版本）
  - **簡化的網路管理**： 主機閘道模式中使用 Flannel 節點之間的路線自動管理
  - **延展性改進**： 享受更快速且更可靠的容器啟動時間，歸功於[無裝置 vnic 與適用於 Windows Server 容器](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)
  - **hyper-v 隔離 (alpha)**: 協調使用的增強式安全性 （[請參閱 Windows 容器類型](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types)） 的核心模式隔離的[hyper-v 容器](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)
    - 需要 Kubernetes v1.10 （或更新版本） 搭配`HyperVContainer`啟用功能閘道
  - **儲存體外掛程式**： 適用於 Windows 容器使用 SMB 和 iSCSI 支援的[FlexVolume 儲存外掛程式](https://github.com/Microsoft/K8s-Storage-Plugins)

> [!TIP] 
> 如果您想要部署在 Azure 上的叢集，開放原始碼 AKS 引擎工具輕鬆完成此操作。 [這裡提供逐步解說](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)。

## <a name="prerequisites"></a>必要條件 ##

### <a name="plan-ip-addressing-for-your-cluster"></a>規劃您的叢集的 IP 位址 ###
<a name="definitions"></a>因為 Kubernetes 叢集導入新的子網路的 pod 和服務，務必要確保無它們與您的環境中任何其他現有網路會發生衝突。 以下是以釋放為了順利部署 Kubernetes 需要的所有位址空間：

| 子網路位址範圍 / | 描述 | 預設值 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**服務子網路** | 不可路由的純虛擬子網路，供 pod 用來一致存取服務，而不需顧及網路拓撲。 節點上執行的 `kube-proxy` 可在服務子網路與可路由的位址空間之間來回轉譯。 | 「 10.96.0.0/12 」 |
| <a name="cluster-subnet-def"></a>**叢集子網路** |  這是通用的子網路，供叢集中的所有 pod。 每個節點指派較小的 /24 子網路從這個供其 pod 使用。 它必須足以容納您的叢集中使用的所有 pod。 若要計算大小*最小值*的子網路： `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>針對每個節點的 100 pod 的 5 個節點的叢集的範例： `(5) + (5 *  100) = 505`。  | 「 10.244.0.0/16 」 |
| **Kubernetes DNS 服務 IP** | 「 Kube dns 」 服務會做為 DNS 解析 & 叢集服務探索的 IP 位址。 | 「 10.96.0.10 」 |
> [!NOTE]
> 沒有其他的 Docker 網路 (NAT) 取得建立根據預設，當您安裝 Docker。 我們將指派 Ip 從叢集子網路改為在進行 Windows 上的 Kubernetes 不需要它。



## <a name="what-you-will-accomplish"></a>您將會完成 ##

本指南結束時，您將會：

> [!div class="checklist"]
> * 建立[Kubernetes 主機](./creating-a-linux-master.md)節點。  
> * 選取[的網路解決方案](./network-topologies.md)。  
> * 加入[Windows 背景工作節點](./joining-windows-workers.md)或[Linux 背景工作節點](./joining-linux-workers.md)它。  
> * 部署[範例 Kubernetes 資源](./deploying-resources.md)。  
> * 涵蓋[常見問題與錯誤](./common-problems.md)。

## <a name="next-steps"></a>後續步驟 ##
在此區段中，我們討論重要的必要條件 & 現今成功部署 Windows 上的 Kubernetes 所需的假設。 若要了解如何設定 Kubernetes 主機繼續：

> [!div class="nextstepaction"]
> [建立 Kubernetes 主機](./creating-a-linux-master.md)