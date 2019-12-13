---
title: 加入 Linux 節點
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 在混合式 OS Kubernetes 叢集上部署 Kubernetes resoureces。
keywords: kubernetes，1.14，windows，使用者入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909958"
---
# <a name="deploying-kubernetes-resources"></a>部署 Kubernetes 資源 #
假設您有一個由至少1個主要和1個背景工作所組成的 Kubernetes 叢集，您就可以開始部署 Kubernetes 資源。
> [!TIP] 
> 想知道 Windows 目前支援哪些 Kubernetes 資源？ 如需詳細資訊，請參閱已[正式支援的功能](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations)和[Windows 藍圖上的 Kubernetes](https://github.com/orgs/kubernetes/projects/8) 。


## <a name="running-a-sample-service"></a>執行範例服務 ##
您將部署非常簡單的 [PowerShell Web 服務](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以確保成功加入叢集和網路設定正確。

在這麼做之前，請務必確定所有節點的狀況良好。
```bash
kubectl get nodes
```

如果一切看起來良好，您可以下載並執行下列服務：
> [!Important] 
> 在 `kubectl apply`之前，請務必仔細檢查/修改範例檔案中的 `microsoft/windowsservercore` 映射，使其[成為節點可執行檔容器映射](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)！

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

這會建立部署和服務。 最後一個監看式命令會無限期地查詢 pod 以追蹤其狀態;只要按 `Ctrl+C`，即可在完成觀察時結束 `watch` 命令。

如果一切順利，您可以：

  - 請參閱 Windows 節點上 `docker ps` 命令下每個 pod 的2個容器
  - 從 Linux 主機的 `kubectl get pods` 命令下看到 2 個 Pod
  - 從 Linux 主機的埠80上的*pod* ip 上 `curl` 會取得 web 伺服器回應;這示範了適當的節點，可在網路上透過 pod 進行通訊。
  - 透過 `docker exec`，*在 Pod 之間* Ping (包括在多個主機上，如果您有多個 Windows 節點)。這示範適當的 Pod 對 Pod 通訊
  - 從 Linux 主機和個別 pod `curl` 虛擬*服務 IP* （出現在 `kubectl get services`底下）;這會示範適當的服務來進行 pod 通訊。
  - `curl`*服務名稱*與 KUBERNETES[預設 DNS 尾碼](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)，以示範適當的服務探索。
  - 從 Linux 主機或叢集外的電腦 `curl` *NodePort* ;這會示範輸入連線能力。
  - 從 pod 內部 `curl` 外部 Ip;這會示範輸出連線能力。

> [!Note]  
> Windows*容器主機*將**無法**從其上排程的服務存取服務 IP。 這是[已知的平臺限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)，將在未來的版本中改善到 Windows Server。 不過 *，Windows pod 可以存取*服務 IP。

## <a name="next-steps"></a>後續步驟 ##
在本節中，我們討論了如何排程 Windows 節點上的 Kubernetes 資源。 這就是本指南的結論。 如有任何問題，請參閱疑難排解一節：

> [!div class="nextstepaction"]
> [疑難排解](./common-problems.md)

否則，您可能也會想要以 Windows 服務的形式執行 Kubernetes 元件：
> [!div class="nextstepaction"]
> [Windows 服務](./kube-windows-services.md)
