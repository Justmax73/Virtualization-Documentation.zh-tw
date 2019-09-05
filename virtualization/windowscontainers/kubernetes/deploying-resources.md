---
title: 加入 Linux 節點
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 在混合 OS Kubernetes 群集上部署 Kubernetes resoureces。
keywords: kubernetes、1.14、windows、快速入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: d252f356a3de98f224e1550536810dfc75345303
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2019
ms.locfileid: "10069942"
---
# <a name="deploying-kubernetes-resources"></a>部署 Kubernetes 資源 #
假設您有至少包含1個主版和1個工作的 Kubernetes 群集, 就可以開始部署 Kubernetes 資源。
> [!TIP] 
> 想知道目前在 Windows 上支援哪些 Kubernetes 資源嗎？ 如需詳細資訊, 請參閱[Windows 藍圖上](https://github.com/orgs/kubernetes/projects/8)[正式支援的功能](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations)與 Kubernetes。


## <a name="running-a-sample-service"></a>執行範例服務 ##
您將部署非常簡單的 [PowerShell Web 服務](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以確保成功加入叢集和網路設定正確。

在執行此動作之前, 請務必先確定所有的節點都正常。
```bash
kubectl get nodes
```

如果所有專案看起來都不錯, 您可以下載並執行下列服務:
> [!Important] 
> 之前`kubectl apply`, 請務必仔細檢查/修改範例檔案中`microsoft/windowsservercore`的圖像, 以移至[您的節點](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)可執行檔容器影像!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

這會建立部署和服務。 最後一個 [監視] 命令會無限期地查詢箱, 以追蹤其狀態;只要按`Ctrl+C`下, 就`watch`能在進行觀察時結束命令。

如果一切順利，您可以：

  - 在 Windows 節點的命令下`docker ps` , 查看每個 pod 2 個容器
  - 從 Linux 主機的 `kubectl get pods` 命令下看到 2 個 Pod
  - `curl` 於 Linux 主機連接埠 80 的 *Pod* IP，取得網頁伺服器回應。這示範跨越網路適當的節點對 Pod 通訊。
  - 透過 `docker exec`，*在 Pod 之間* Ping (包括在多個主機上，如果您有多個 Windows 節點)。這示範適當的 Pod 對 Pod 通訊
  - `curl` 從 Linux 主版和個別盒`kubectl get services`中的虛擬*服務 IP* (見下);這會示範適當的服務來進行 pod 通訊。
  - `curl` 具有 Kubernetes[預設 DNS 尾碼](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)的*服務名稱*, 示範正確的服務探索。
  - `curl` 從 Linux 主機或群集以外的電腦*NodePort* ;這會示範輸入連線。
  - `curl` 盒中的外部 Ip;這會示範輸出連線。

> [!Note]  
> Windows*容器主機*將**無法**從排程的服務存取服務 IP。 這是[已知的平臺限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip), 將會在未來版本的 Windows Server 中改進。 Windows*盒* **** 可以存取服務 IP。

## <a name="next-steps"></a>後續步驟 ##
在本節中, 我們將說明如何在 Windows 節點上排程 Kubernetes 資源。 本指南結束。 如果有任何問題, 請參閱疑難排解一節:

> [!div class="nextstepaction"]
> [疑難排解](./common-problems.md)

否則, 您可能也想要將 Kubernetes 元件執行為 Windows services:
> [!div class="nextstepaction"]
> [Windows 服務](./kube-windows-services.md)
