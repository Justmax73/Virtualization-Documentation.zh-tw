---
title: 加入 Linux 節點
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 部署 Kubernetes resoureces 混合作業系統 Kubernetes 叢集上。
keywords: kubernetes，1.13，windows，開始使用
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 380eeb536b9642210c49bc91edf680b695d54a90
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621036"
---
# <a name="deploying-kubernetes-resources"></a>部署 Kubernetes 資源 #
假設您有至少 1 個主機和 1 的背景工作所組成的 Kubernetes 叢集，您已經準備好部署 Kubernetes 資源。
> [!TIP] 
> 好奇現今在 Windows 上支援何種 Kubernetes 資源？ 請如需詳細資訊參閱[正式支援的功能](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features)與[Windows 藍圖上的 Kubernetes](https://trello.com/b/rjTqrwjl/windows-k8s-roadmap) 。


## <a name="running-a-sample-service"></a>執行範例服務 ##
您將部署非常簡單的 [PowerShell Web 服務](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以確保成功加入叢集和網路設定正確。

之前此情況下，它一律是不錯的想法，以確定我們的所有節點都皆狀況良好。
```bash
kubectl get nodes
```

如果一切看起來良好，您可以下載並執行下列服務：
> [!Important] 
> 之前`kubectl apply`，請確定為按兩下-check/修改`microsoft/windowsservercore`映像，[這是由您的節點可執行的容器映像](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)範例檔案中 ！

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

這會建立部署和服務。 最後一個監看命令會查詢 pod 無限期，追蹤其狀態;只要按下`Ctrl+C`結束`watch`當完成觀察時的命令。

如果一切順利，您可以：

  - 請參閱下方的 pod 每 2 個容器`docker ps`命令在 Windows 節點
  - 從 Linux 主機的 `kubectl get pods` 命令下看到 2 個 Pod
  - `curl` 於 Linux 主機連接埠 80 的 *Pod* IP，取得網頁伺服器回應。這示範跨越網路適當的節點對 Pod 通訊。
  - 透過 `docker exec`，*在 Pod 之間* Ping (包括在多個主機上，如果您有多個 Windows 節點)。這示範適當的 Pod 對 Pod 通訊
  - `curl` 虛擬*服務 IP* (底下看到`kubectl get services`) 從 Linux 主機和個別的 pod;這示範適當的服務對 pod 通訊。
  - `curl` *服務名稱*與 Kubernetes[預設 DNS 尾碼](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)，示範適當的服務探索。
  - `curl` 從 Linux 主機或叢集; 以外的電腦*NodePort*這示範輸入的連線。
  - `curl` pod; 內的外部 Ip這示範輸出的連線。

> [!Note]  
> Windows*容器主機*將會**不**無法從排程在他們的服務存取服務 IP。 這是[已知的平台限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)，將會改善到 Windows Server 的未來版本中。 Windows *pod* **都**能不過存取服務 IP。

### <a name="port-mapping"></a>連接埠對應 ### 
此外，也可以透過其各自節點存取 Pod 裝載的服務，方法是對應節點上的連接埠。 [這裡有另一個範例 YAML](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml)，其中節點上的連接埠 4444 對應至 Pod 上的連接埠 80，以示範這項功能。 若要部署它，請依照先前相同的步驟操作：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

現在應該可以在連接埠 4444 的*節點* IP 上執行 `curl`，並接收網頁伺服器回應。 請注意，這會將規模調整限制在每個節點單一 Pod，因為它必須強制執行一對一的對應。


## <a name="next-steps"></a>後續步驟 ##
在此區段中，我們會討論如何排程 Windows 節點上的 Kubernetes 資源。 這包含本指南。 如果有任何問題，請檢閱疑難排解章節：

> [!div class="nextstepaction"]
> [疑難排解](./common-problems.md)

否則，您可能也會興趣 Windows 服務身分執行 Kubernetes 元件：
> [!div class="nextstepaction"]
> [Windows 服務](./kube-windows-services.md)