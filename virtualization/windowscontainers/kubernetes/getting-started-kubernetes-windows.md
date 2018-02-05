---
title: "Windows 上的 Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "將 Windows 節點加入 v1.9 beta Kubernetes 叢集。"
keywords: "kubernetes, 1.9, windows, 開始使用"
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: f1b832f8a21c034582e157342acf7826fb7b6ea3
ms.sourcegitcommit: b0e21468f880a902df63ea6bc589dfcff1530d6e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2018
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes #
隨著 Kubernetes 1.9 與 Windows Server [版本 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking) 最新版本問世，使用者可以利用 Windows 網路的最新功能：

  - **共用 Pod 區間**：基礎架構和背景工作角色 Pod 現在共用網路區間 (類似於 Linux 命名空間)
  - **端點最佳化**：由於區間共用，容器服務現在需要追蹤的端點數量是以前至少一半
  - **資料路徑最佳化**：改進虛擬篩選平台和主機網路服務，允許核心型負載平衡


此頁面是開始將全新 Windows 節點加入現有 Linux 型叢集的指南。 若要完全從頭開始，請參考[此頁面](./creating-a-linux-master.md) &mdash; 可供部署 Kubernetes 叢集的許多資源之一 &mdash; 以從頭設定主機，就像我們當初一樣。


<a name="definitions"></a>在本指南中，所參考一些字詞的定義如下：

  - **外部網路**是您的節點通訊所跨越的網路。
  - <a name="cluster-subnet-def"></a>**叢集子網路**是一種可路由的虛擬網路。從這個叢集子網路指派規模較小的子網路給節點，供其 Pod 使用。
  - **服務子網路**是在 11.0/16 上不可路由的純虛擬子網路，供 Pod 用來一致存取服務，而不需顧及網路拓撲。 節點上執行的 `kube-proxy` 可在服務子網路與可路由的位址空間之間來回轉譯。


## <a name="what-we-will-accomplish"></a>我們將會完成 ##
本指南結束時，我們將會：

> [!div class="checklist"]  
> * 備妥[網路拓撲](#network-topology)。  
> * 設定 [Linux 主要](#preparing-the-linux-master)節點。  
> * 將 [Windows 背景工作角色節點](#preparing-a-windows-node)聯結此節點。  
> * 部署[範例 Windows 服務](#running-a-sample-service)。  
> * 涵蓋[常見問題與錯誤](./common-problems.md)。  


## <a name="network-topology"></a>網路拓撲 ##
有多個方式可以讓虛擬[叢集子網路](#cluster-subnet-def)成為可路由的。 您可以：

  - 設定[主機閘道模式](./configuring-host-gateway-mode.md)，在節點間設定靜態的下一個躍點路由，以啟用 Pod 對 Pod 通訊。
  - 設定智慧 Top-of-Rack (ToR) 交換器以路由子網路。
  - 使用協力廠商重疊外掛程式，例如 [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) (搶鮮版中提供 Windows 對 Flannel 的支援)。


## <a name="preparing-the-linux-master"></a>準備 Linux 主機 ##
無論您是遵循[我們的指示](./creating-a-linux-master.md)還是已經有現有的叢集，Linux 主機只需要一件事，就是 Kubernetes 的憑證設定。 這可能是在 `/etc/kubernetes/admin.conf`、`~/.kube/config` 或其他地方，視設定而定。


## <a name="preparing-a-windows-node"></a>準備 Windows 節點 ##
> [!Note]  
> Windows 章節中的所有程式碼片段都是在_提升權限_的 PowerShell 中執行。

Kubernetes 使用 [Docker](https://www.docker.com/) 做為其容器協調器，所以我們必須安裝它。 您可以遵循 [MSDN 正式指示](virtualization/windowscontainers/manage-docker/configure-docker-daemon.md#install-docker)、[Docker 指示](https://store.docker.com/editions/enterprise/docker-ee-server-windows)，或嘗試下列步驟：

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

[這個 Microsoft 存放庫](https://github.com/Microsoft/SDN)上的指令碼集合，可協助我們將此節點加入叢集。 您可以直接在[這裡](https://github.com/Microsoft/SDN/archive/master.zip)下載 ZIP 檔案。 我們只需要 `Kubernetes/windows` 資料夾，其中的內容應該移至 `C:\k\`：

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

將[先前指出的](#preparing-the-linux-master)憑證檔案複製到此新的 `C:\k` 目錄。


### <a name="creating-the-pause-image"></a>建立 "Pause" 映像 ###
現在 `docker` 已安裝，我們需要準備 "pause" 映像，供 Kubernetes 用來準備基礎架構 Pod。

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!Note]  
> 雖然這實際上_未必_是最新發行的 Windows Server Core 映像，但我們稍後即將部署的範例服務對其有相依性，因此加以標記為 `:latest`。 請務必小心不要讓容器映像發生衝突，如果沒有加上預期的標記，可能會使不相容的容器影像產生 `docker pull` 現象，並造成[部署問題](./common-problems.md#when-deploying-docker-containers-keep-restarting)。 


### <a name="downloading-binaries"></a>下載二進位檔 ###
在 `pull` 發生的同時，請從 Kubernetes 下載下列用戶端二進位檔案：

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

您可以從 `CHANGELOG.md` 檔案的 1.9 最新版本中的連結下載這些二進位檔案。 撰寫本文時，最新版本是 [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1)，而 Windows 二進位檔在[這裡](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz)。 使用工具 (例如 [7-Zip](http://www.7-zip.org/)) 來解壓縮封存檔案，並將二進位檔放置在 `C:\k\` 中。


### <a name="joining-the-cluster"></a>加入叢集 ###
節點現在已經準備好加入叢集。 在兩個不同、*提升權限*的 PowerShell 視窗中，執行這些指令碼 (依此順序)。 第一個指令碼的 `-ClusterCidr` 參數是設定的[叢集子網路](#cluster-subnet-def)；在此，它是 `192.168.0.0/16`。

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

在一分鐘內，Windows 節點就能從 Linux 主機 (在 `kubectl get nodes` 下) 看到！


### <a name="validating-your-network-topology"></a>驗證您的網路拓撲 ###
有幾個基本測試可驗證適當的網路設定：

  - **節點對節點連線**：主機和 Windows 背景工作角色節點之間的 Ping 應該會雙向成功。

  - **Pod 子網路對節點連線**：虛擬 Pod 介面和節點之間的 Ping。 在 Linux 和 Windows 的 `route -n` 和 `ipconfig` 下分別尋找閘道位址，並尋找 `cbr0` 介面。

如果這些基本測試都不成功，請嘗試[疑難排解頁面](./common-problems.md#network-connectivity)來解決常見問題。


## <a name="running-a-sample-service"></a>執行範例服務 ##
我們將部署非常簡單的 [PowerShell Web 服務](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以確保成功加入叢集和網路設定正確。


在 Linux 主機，下載並執行服務：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

這將會建立部署和服務，然後無限期監看 Pod，追蹤其狀態。當完成觀察時，只要按下 `Ctrl+C` 即可結束 `watch` 命令。


如果一切順利，您就可以驗證下列項目：

  - 在 Windows 端的 `docker ps` 命令下看到 4 個容器
  - `curl` 於 Linux 主機連接埠 80 的 *Pod* IP，取得網頁伺服器回應。這示範跨越網路適當的節點對 Pod 通訊。
  - `curl` 於連接埠 4444 的*節點* IP，取得網頁伺服器回應。這示範適當的主機對容器連接埠對應。
  - 透過 `docker exec`，*在 Pod 之間* Ping (包括在多個主機上，如果您有多個 Windows 節點)。這示範適當的 Pod 對 Pod 通訊
  - `curl` 虛擬*服務 IP* (在 `kubectl get services` 底下看到)，來自 Linux 主機和個別的 Pod。
  - `curl` *服務名稱*與 Kubernetes [預設 DNS 尾碼](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)，這示範 DNS 功能。

> [!Warning]  
> Windows 節點無法存取服務 IP。 這是[已知限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)。
