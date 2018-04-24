---
title: Windows 上的 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: 將 Windows 節點加入 v1.9 beta Kubernetes 叢集。
keywords: kubernetes, 1.9, windows, 開始使用
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6309ca8c0fd50e1b8e926776bef6dfe82bb815f0
ms.sourcegitcommit: ee86ee093b884c79039a8ff417822c6e3517b92d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
---
# <a name="kubernetes-on-windows"></a>Windows 上的 Kubernetes #

隨著 Kubernetes 1.9 與 Windows Server [版本 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking) 最新版本問世，使用者可以利用 Windows 網路的最新功能：

  - **共用 Pod 區間**：基礎架構和背景工作角色 Pod 現在共用網路區間 (類似於 Linux 命名空間)
  - **端點最佳化**：由於區間共用，容器服務現在需要追蹤的端點數量是以前至少一半
  - **資料路徑最佳化**：改進虛擬篩選平台和主機網路服務，允許核心型負載平衡


此頁面是開始將全新 Windows 節點加入現有 Linux 型叢集的指南。 若要完全從頭開始，請參考[此頁面](./creating-a-linux-master.md) &mdash; 可供部署 Kubernetes 叢集的許多資源之一 &mdash; 以從頭設定主機，就像我們當初一樣。

> [!TIP] 
> 如果您想要在 Azure 上部署叢集，開放原始碼 ACS 引擎工具可用來輕鬆完成此操作。 [這裡提供逐步解說](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md)。

<a name="definitions"></a>在本指南中，所參考一些字詞的定義如下：

  - **外部網路**是您的節點通訊所跨越的網路。
  - <a name="cluster-subnet-def"></a>**叢集子網路**是一種可路由的虛擬網路。從這個叢集子網路指派規模較小的子網路給節點，供其 Pod 使用。
  - **服務子網路**是在 11.0/16 上不可路由的純虛擬子網路，供 Pod 用來一致存取服務，而不需顧及網路拓撲。 節點上執行的 `kube-proxy` 可在服務子網路與可路由的位址空間之間來回轉譯。

## <a name="what-you-will-accomplish"></a>您將會完成 ##

本指南結束時，您將會：

> [!div class="checklist"]  
> * 設定 [Linux 主要](#preparing-the-linux-master)節點。  
> * 將 [Windows 背景工作角色節點](#preparing-a-windows-node)聯結此節點。  
> * 備妥[網路拓撲](#network-topology)。  
> * 部署[範例 Windows 服務](#running-a-sample-service)。  
> * 涵蓋[常見問題與錯誤](./common-problems.md)。  

## <a name="preparing-the-linux-master"></a>準備 Linux 主機 ##

無論您是遵循[指示](./creating-a-linux-master.md)還是已經有現有的叢集，Linux 主機只需要一件事，就是 Kubernetes 的憑證設定。 這可能是在 `/etc/kubernetes/admin.conf`、`~/.kube/config` 或其他地方，視設定而定。

## <a name="preparing-a-windows-node"></a>準備 Windows 節點 ##

> [!NOTE]  
> Windows 章節中的所有程式碼片段都是在_提升權限_的 PowerShell 中執行。

Kubernetes 使用 [Docker](https://www.docker.com/) 做為其容器協調器，所以我們必須安裝它。 您可以遵循 [Docs 正式指示](../manage-docker/configure-docker-daemon.md#install-docker)、[Docker 指示](https://store.docker.com/editions/enterprise/docker-ee-server-windows)，或嘗試下列步驟：

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

如果您位於 Proxy 伺服器後方，必須定義下列 PowerShell 環境變數：
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

[這個 Microsoft 存放庫](https://github.com/Microsoft/SDN)上的指令碼集合，可協助您將此節點加入叢集。 您可以直接在[這裡](https://github.com/Microsoft/SDN/archive/master.zip)下載 ZIP 檔案。 您只需要 `Kubernetes/windows` 資料夾，其中的內容應該移至 `C:\k\`：

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

將[先前指出的](#preparing-the-linux-master)憑證檔案複製到此新的 `C:\k` 目錄。

## <a name="network-topology"></a>網路拓撲 ##

有多個方式可以讓虛擬[叢集子網路](#cluster-subnet-def)成為可路由的。 您可以：

  - 設定[主機閘道模式](./configuring-host-gateway-mode.md)，在節點間設定靜態的下一個躍點路由，以啟用 Pod 對 Pod 通訊。
  - 設定智慧 Top-of-Rack (ToR) 交換器以路由子網路。
  - 使用協力廠商重疊外掛程式，例如 [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) (搶鮮版中提供 Windows 對 Flannel 的支援)。

### <a name="creating-the-pause-image"></a>建立 "pause" 映像 ###

現在 `docker` 已安裝，您需要準備 "pause" 映像，供 Kubernetes 用來準備基礎架構 Pod。

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!NOTE]
> 雖然這實際上_未必_是最新發行的 Windows Server Core 映像，但您稍後即將部署的範例服務對其有相依性，因此加以標記為 `:latest`。 請務必小心不要讓容器映像發生衝突，如果沒有加上預期的標記，可能會使不相容的容器影像產生 `docker pull` 現象，並造成[部署問題](./common-problems.md#when-deploying-docker-containers-keep-restarting)。 


### <a name="downloading-binaries"></a>下載二進位檔 ###
在 `pull` 發生的同時，請從 Kubernetes 下載下列用戶端二進位檔案：

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

您可以從 `CHANGELOG.md` 檔案的 1.9 最新版本中的連結下載這些二進位檔案。 撰寫本文時，最新版本是 [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1)，而 Windows 二進位檔在[這裡](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz)。 使用工具 (例如 [7-Zip](http://www.7-zip.org/)) 來解壓縮封存檔案，並將二進位檔放置在 `C:\k\` 中。

為了在 `C:\k\` 目錄以外使用 `kubectl` 命令，請修改 `PATH` 環境變數：

```powershell
$env:Path += ";C:\k"
```

如果要讓此變更具有永久性，請在電腦目標修改變數：

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

### <a name="joining-the-cluster"></a>加入叢集 ###
使用下列命令，確認叢集設定是否有效：

```powershell
kubectl version
```

如果您收到連線錯誤，

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

檢查設定探索是否正常運作：

```powershell
kubectl config view
```

若要變更 `kubectl` 尋找設定檔的位置，您可以傳遞 `--kubeconfig` 參數或修改 `KUBECONFIG` 環境變數。 例如，如果設定位於 `C:\k\config`：

```powershell
$env:KUBECONFIG="C:\k\config"
```

若要讓這項設定對目前使用者的範圍具有永久性：

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

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

如果這些基本測試都不成功，請嘗試[疑難排解頁面](./common-problems.md#common-networking-errors)來解決常見問題。


## <a name="running-a-sample-service"></a>執行範例服務 ##

您將部署非常簡單的 [PowerShell Web 服務](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml)，以確保成功加入叢集和網路設定正確。

在 Linux 主機，下載並執行服務：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

這會建立部署和服務，然後無限期監看 Pod，追蹤其狀態。當完成觀察時，只要按下 `Ctrl+C` 即可結束 `watch` 命令。

如果一切順利，您可以：

  - 在 Windows 節點的 `docker ps` 命令下看到 4 個容器
  - 從 Linux 主機的 `kubectl get pods` 命令下看到 2 個 Pod
  - `curl` 於 Linux 主機連接埠 80 的 *Pod* IP，取得網頁伺服器回應。這示範跨越網路適當的節點對 Pod 通訊。
  - 透過 `docker exec`，*在 Pod 之間* Ping (包括在多個主機上，如果您有多個 Windows 節點)。這示範適當的 Pod 對 Pod 通訊
  - `curl` 虛擬*服務 IP* (在 `kubectl get services` 底下看到)，來自 Linux 主機和個別的 Pod。
  - `curl` *服務名稱*與 Kubernetes [預設 DNS 尾碼](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)，這示範 DNS 功能。

> [!Warning]  
> Windows 節點無法存取服務 IP。 這是[已知的平台限制](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip)，將會在 Windows Server 的下一個更新中改善。


### <a name="port-mapping"></a>連接埠對應 ### 
此外，也可以透過其各自節點存取 Pod 裝載的服務，方法是對應節點上的連接埠。 [這裡有另一個範例 YAML](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml)，其中節點上的連接埠 4444 對應至 Pod 上的連接埠 80，以示範這項功能。 若要部署它，請依照先前相同的步驟操作：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

現在應該可以在連接埠 4444 的*節點* IP 上執行 `curl`，並接收網頁伺服器回應。 請注意，這會將規模調整限制在每個節點單一 Pod，因為它必須強制執行一對一的對應。