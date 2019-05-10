---
title: 加入 Windows 節點
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 您可以將 Windows 節點加入 v1.13 Kubernetes 叢集。
keywords: kubernetes，1.13，windows，開始使用
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: fec18baa58ae6aa7d637be575548a1823a551207
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620986"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>將 Windows Server 節點加入叢集 #
一旦您有[設定 Kubernetes 主機的節點](./creating-a-linux-master.md)，並[選取您想要的網路的解決方案](./network-topologies.md)，您準備好要加入 Windows Server 來形成叢集的節點。 這需要一些[準備 Windows 節點上的](#preparing-a-windows-node)，才能將加入。

## <a name="preparing-a-windows-node"></a>準備 Windows 節點 ##
> [!NOTE]  
> Windows 章節中的所有程式碼片段都是在_提升權限_的 PowerShell 中執行。

### <a name="install-docker-requires-reboot"></a>安裝的 Docker （需要重新開機） ###
Kubernetes 會使用[Docker](https://www.docker.com/)做為其容器引擎，因此我們必須安裝它。 您可以遵循 [Docs 正式指示](../manage-docker/configure-docker-daemon.md#install-docker)、[Docker 指示](https://store.docker.com/editions/enterprise/docker-ee-server-windows)，或嘗試下列步驟：

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

如果在重新開機之後，您會看到下列錯誤：

![文字](media/docker-svc-error.png)

手動啟動 docker 服務：

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>建立"pause"（基礎結構） 映像 ###
> [!Important]
> 請務必小心衝突的容器映像;不需要預期的標記可能會造成`docker pull`的不相容的容器映像，造成[部署問題](./common-problems.md#when-deploying-docker-containers-keep-restarting)例如無限期`ContainerCreating`狀態。

現在 `docker` 已安裝，您需要準備 "pause" 映像，供 Kubernetes 用來準備基礎架構 Pod。 有三個步驟： 
  1. [提取映像](#pull-the-image)
  2. [它標記](#tag-the-image)為 microsoft / nanoserver:latest
  3. 並[執行它](#run-the-container)


#### <a name="pull-the-image"></a>提取映像 ####     
 提取映像個您特定的 Windows 版本。 例如，如果您正在執行 Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>標記的影像 ####
您將使用本指南稍後的 Dockerfiles 尋找`:latest`標記的影像。 標記 nanoserver 映像，您只是提取，如下所示：

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>執行容器 ####
仔細檢查容器實際執行您的電腦上：

```powershell
docker run microsoft/nanoserver:latest
```

您應該會看到類似這樣：

![文字](./media/docker-run-sample.png)

> [!tip]
> 如果您無法執行容器請請參閱：[符合容器主機版本與容器映像](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>準備 Windows 的 Kubernetes 目錄 ####
建立 「 Windows Kubernetes 」 目錄來儲存 Kubernetes 二進位檔，以及任何的部署指令碼和組態檔。

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>複製 Kubernetes 的憑證 #### 
複製的 Kubernetes 的憑證檔案 (`$HOME/.kube/config`)[從主機](./creating-a-linux-master.md#collect-cluster-information)到這個新`C:\k`目錄。

> [!tip]
> 您可以使用工具例如[xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy)或[WinSCP](https://winscp.net/eng/download.php)節點之間傳輸設定檔。

#### <a name="download-kubernetes-binaries"></a>下載 Kubernetes 二進位檔 ####
若要能夠執行 Kubernetes 方式執行，您必須下載`kubectl`， `kubelet`，以及`kube-proxy`二進位檔。 您可以下載這些中的連結從`CHANGELOG.md`檔案的[最新發行版本](https://github.com/kubernetes/kubernetes/releases/)。
 - 例如，以下是[v1.13 節點的二進位檔](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#node-binaries)。
 - 使用解壓縮封存，並將放入二進位檔的工具，例如[展開封存](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) `C:\k\`。

#### <a name="optional-setup-kubectl-on-windows"></a>（選擇性）在 Windows 上的安裝程式 kubectl ####
如果您想要控制從 Windows 叢集，您可以使用`kubectl`命令。 首先，讓`kubectl`以外`C:\k\`目錄中，修改`PATH`環境變數：

```powershell
$env:Path += ";C:\k"
```

如果要讓此變更具有永久性，請在電腦目標修改變數：

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

接下來，我們將會驗證[叢集憑證](#copy-kubernetes-certificate)有效。 若要設定的位置其中`kubectl`尋找設定檔，您可以傳遞`--kubeconfig`參數或修改`KUBECONFIG`環境變數。 例如，如果設定位於 `C:\k\config`：

```powershell
$env:KUBECONFIG="C:\k\config"
```

若要讓這項設定對目前使用者的範圍具有永久性：

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

最後，若要檢查設定探索是否正常運作，您可以使用：

```powershell
kubectl config view
```

如果您收到連線錯誤，

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

您應該仔細檢查 kubeconfig 位置，或嘗試複製它一次。

如果您看到沒有任何錯誤的節點現已準備好加入叢集。

## <a name="joining-the-windows-node"></a>加入 Windows 節點 ##
根據[您所選擇的網路解決方案](./network-topologies.md)，您可以：
1. [Windows Server 節點加入 Flannel （vxlan 或主機 gw） 叢集](#joining-a-flannel-cluster)
2. [Windows Server 節點加入叢集上使用 ToR 交換器](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>加入 Flannel 叢集 ###
沒有[這個 Microsoft 存放庫](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay)，可協助您將此節點加入叢集上 Flannel 部署指令碼的集合。

下載[Flannel start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)指令碼，其中的內容解壓縮到`C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

假設您[準備好您的 Windows 節點](#preparing-a-windows-node)，以及您`c:\k`目錄看起來就像下面，您就已經準備好加入節點。

![文字](./media/flannel-directory.png)

#### <a name="join-node"></a>加入節點 #### 
若要簡化將 Windows 節點的程序，您只需要執行單一 Windows 指令碼來啟動`kubelet`， `kube-proxy`， `flanneld`，並加入節點。

> [!Note]
> [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)參考[install.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1)，例如下載其他檔案的`flanneld`可執行檔和[基礎架構 pod 的 Dockerfile](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *並安裝您差別*。 覆疊網路模式，如[防火牆](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111)將會開啟本機的 UDP 連接埠 4789。 可能有多個 powershell 視窗中的幾秒鐘的網路中斷而以及開啟/關閉時正在建立新的外部 vSwitch pod 網路第一次。

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
IP 位址指派給 Windows 節點。 您可以使用`ipconfig`若要尋找這。

|  |  | 
|---------|---------|
|參數     | `-ManagementIP`        |
|預設值    | n.A. **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
網路模式`l2bridge`(flannel 主機 gw) 或`overlay`(flannel vxlan) 做為[網路解決方案](./network-topologies.md)選擇。

> [!Important] 
> `overlay` 網路功能模式 (flannel vxlan) 需要 Kubernetes v1.14 二進位檔案或更新版本。

|  |  | 
|---------|---------|
|參數     | `-NetworkMode`        |
|預設值    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[叢集子網路範圍](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  | 
|---------|---------|
|參數     | `-ClusterCIDR`        |
|預設值    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
[服務子網路範圍](./getting-started-kubernetes-windows.md#service-subnet-def)。

|  |  | 
|---------|---------|
|參數     | `-ServiceCIDR`        |
|預設值    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Kubernetes DNS 服務 IP](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster)。

|  |  | 
|---------|---------|
|參數     | `-KubeDnsServiceIP`        |
|預設值    | `10.96.0.10`        |


# [<a name="interfacename"></a>預設](#tab/InterfaceName)
Windows 主機的網路介面的名稱。 您可以使用`ipconfig`若要尋找這。

|  |  | 
|---------|---------|
|參數     | `-InterfaceName`        |
|預設值    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
其中 kubelet 和 kube proxy 記錄檔會被重新導向至其各自的輸出檔目錄。

|  |  | 
|---------|---------|
|參數     | `-LogDir`        |
|預設值    | `C:\k`        |


---

> [!tip]
> 您已經記下向下叢集子網路、 服務子網路，並從 Linux 主機 KUBE-DNS IP[較舊版本](./creating-a-linux-master.md#collect-cluster-information)

在執行這之後，您應該能夠：
  * 檢視已加入使用的 Windows 節點 `kubectl get nodes`
  * 請參閱 3 個 powershell 視窗中開啟，一個用於`kubelet`，一個適用於`flanneld`，和另一個用於 `kube-proxy`
  * 請參閱主機代理程式處理程序的`flanneld`， `kubelet`，以及`kube-proxy`的節點上執行

如果成功，繼續進行[下一個步驟](#next-steps)。

## <a name="joining-a-tor-cluster"></a>加入 ToR 叢集 ##
> [!NOTE]
> 如果您選擇 Flannel 做為您網路解決方案[之前](./network-topologies.md#flannel-in-host-gateway-mode)，您可以略過本節。

若要這樣做，您需要依照指示來[設定上游 L3 路由拓撲的 Kubernetes 上的 Windows Server 容器](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)。 這包括確定您設定上游路由器，讓這類 pod CIDR 首碼指派給節點會對應到其各自節點 IP。

假設新節點會藉由 「 就緒 」 形式列出`kubectl get nodes`，讓 kubelet + kube proxy 正在執行時，並且您已設定上游 ToR 路由器，您已準備好進行下一個步驟。

## <a name="next-steps"></a>後續步驟 ##
在此區段中，我們會討論如何 Windows 工作者加入我們的 Kubernetes 叢集。 現在您已經準備好進行步驟 5:

> [!div class="nextstepaction"]
> [加入 Linux 工作者](./joining-linux-workers.md)

或者，如果您沒有任何 Linux 工作者放心地跳至步驟 6:

> [!div class="nextstepaction"]
> [部署 Kubernetes 資源](./deploying-resources.md)
