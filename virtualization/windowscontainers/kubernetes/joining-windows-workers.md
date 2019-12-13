---
title: 加入 Windows 節點
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 使用 v 1.14 將 Windows 節點加入至 Kubernetes 叢集。
keywords: kubernetes，1.14，windows，使用者入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910328"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>將 Windows Server 節點加入叢集 #
[設定 Kubernetes 主要節點](./creating-a-linux-master.md)並[選取您想要的網路解決方案](./network-topologies.md)之後，您就可以加入 Windows Server 節點來形成叢集。 這需要在加入之前先[對 Windows 節點進行一些準備工作](#preparing-a-windows-node)。

## <a name="preparing-a-windows-node"></a>準備 Windows 節點 ##
> [!NOTE]  
> Windows 章節中的所有程式碼片段都是在_提升權限_的 PowerShell 中執行。

### <a name="install-docker-requires-reboot"></a>安裝 Docker （需要重新開機） ###
Kubernetes 使用[Docker](https://www.docker.com/)作為其容器引擎，因此我們必須加以安裝。 您可以遵循 [Docs 正式指示](../manage-docker/configure-docker-daemon.md#install-docker)、[Docker 指示](https://store.docker.com/editions/enterprise/docker-ee-server-windows)，或嘗試下列步驟：

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

如果在重新開機後，您會看到下列錯誤：

![文字](media/docker-svc-error.png)

然後手動啟動 docker 服務：

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>建立「暫停」（基礎結構）映射 ###
> [!Important]
> 請務必留意衝突的容器映射。不具有預期的標記可能會造成不相容的容器映射 `docker pull`，導致[部署問題](./common-problems.md#when-deploying-docker-containers-keep-restarting)，例如不確定的 `ContainerCreating` 狀態。

現在 `docker` 已安裝，您需要準備 "pause" 映像，供 Kubernetes 用來準備基礎架構 Pod。 這有三個步驟： 
  1. [提取映射](#pull-the-image)
  2. [將其標記](#tag-the-image)為 microsoft/nanoserver：最新
  3. 並執行[它](#run-the-container)


#### <a name="pull-the-image"></a>提取映射 ####     
 提取特定 Windows 版本的映射。 例如，如果您正在執行 Windows Server 2019：

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>標記影像 ####
您稍後將在本指南中使用的 Dockerfile 會尋找 `:latest` 的影像標記。 標記您剛提取的 nanoserver 映射，如下所示：

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>執行容器 ####
再次檢查容器是否在您的電腦上實際執行：

```powershell
docker run microsoft/nanoserver:latest
```

您應該會看到類似這樣的畫面︰

![文字](./media/docker-run-sample.png)

> [!tip]
> 如果您無法執行容器，請參閱：[搭配容器映射來比對容器主機版本](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>準備 Windows 目錄的 Kubernetes ####
建立 "Kubernetes for Windows" 目錄來儲存 Kubernetes 二進位檔，以及任何部署腳本和設定檔案。

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>複製 Kubernetes 憑證 #### 
將 Kubernetes 憑證檔案（`$HOME/.kube/config`）[從 master](./creating-a-linux-master.md#collect-cluster-information)複製到這個新的 `C:\k` 目錄。

> [!tip]
> 您可以使用[xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy)或[WinSCP](https://winscp.net/eng/download.php)之類的工具，在節點之間傳送設定檔。

#### <a name="download-kubernetes-binaries"></a>下載 Kubernetes 二進位檔 ####
若要能夠執行 Kubernetes，您必須先下載 `kubectl`、`kubelet`和 `kube-proxy` 二進位檔。 您可以從[最新版本](https://github.com/kubernetes/kubernetes/releases/)的 `CHANGELOG.md` 檔案中的連結下載這些檔案。
 - 例如，以下是[v 1.14 節點二進位](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries)檔。
 - 使用 [[展開-](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6)封存] 這類工具來解壓縮封存，並將二進位檔放入 `C:\k\`。

#### <a name="optional-setup-kubectl-on-windows"></a>選擇性在 Windows 上安裝 kubectl ####
如果您想要從 Windows 控制叢集，您可以使用 `kubectl` 命令來執行此動作。 首先，若要讓 `kubectl` 可供 `C:\k\` 目錄外部使用，請修改 `PATH` 環境變數：

```powershell
$env:Path += ";C:\k"
```

如果要讓此變更具有永久性，請在電腦目標修改變數：

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

接下來，我們將驗證叢集[憑證](#copy-kubernetes-certificate)是否有效。 若要設定 `kubectl` 尋找設定檔的位置，您可以傳遞 `--kubeconfig` 參數，或修改 `KUBECONFIG` 環境變數。 例如，如果設定位於 `C:\k\config`：

```powershell
$env:KUBECONFIG="C:\k\config"
```

若要讓這項設定對目前使用者的範圍具有永久性：

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

最後，若要檢查是否已正確探索設定，您可以使用：

```powershell
kubectl config view
```

如果您收到連線錯誤，

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

您應該仔細檢查 kubeconfig 位置，或試著再次複製它。

如果您沒有看到任何錯誤，表示節點現在已準備好加入叢集。

## <a name="joining-the-windows-node"></a>加入 Windows 節點 ##
視[您選擇的網路解決方案](./network-topologies.md)而定，您可以：
1. [將 Windows Server 節點加入 Flannel （vxlan 或主機-gw）叢集](#joining-a-flannel-cluster)
2. [將 Windows Server 節點加入具有 ToR 交換器的叢集](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>加入 Flannel 叢集 ###
[此 Microsoft 存放庫](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay)中有一組 Flannel 部署腳本，可協助您將此節點加入叢集。

下載[Flannel start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)腳本，其內容應解壓縮至 `C:\k`：

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

假設您已[準備好 Windows 節點](#preparing-a-windows-node)，而您的 `c:\k` 目錄看起來如下所示，則您已準備好加入節點。

![文字](./media/flannel-directory.png)

#### <a name="join-node"></a>加入節點 #### 
為了簡化加入 Windows 節點的程式，您只需要執行單一 Windows 腳本來啟動 `kubelet`、`kube-proxy`、`flanneld`和加入節點。

> [!Note]
> [啟動 ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)參照[install. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1)，它會下載其他檔案，例如 `flanneld` 可執行檔和[基礎結構 pod 的 Dockerfile](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) ，*並為您安裝這些*檔案。 針對重迭網路模式，[防火牆](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111)將會針對本機 UDP 埠4789開啟。 當第一次建立 pod 網路的新外部 vSwitch 時，可能會有多個 powershell 視窗開啟/關閉，以及幾秒鐘的網路中斷。

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# <a name="managementiptabmanagementip"></a>[ManagementIP](#tab/ManagementIP)
指派給 Windows 節點的 IP 位址。 您可以使用 `ipconfig` 來尋找此。

|  |  | 
|---------|---------|
|參數     | `-ManagementIP`        |
|預設值    | 馬丁 **必要**        |

# <a name="networkmodetabnetworkmode"></a>[NetworkMode](#tab/NetworkMode)
選擇做為[網路解決方案](./network-topologies.md)的網路模式 `l2bridge` （flannel 主機-gw）或 `overlay` （flannel vxlan）。

> [!Important] 
> `overlay` 網路模式（flannel vxlan）需要 Kubernetes v 1.14 二進位檔（或更新版本）和[KB4489899](https://support.microsoft.com/help/4489899)。

|  |  | 
|---------|---------|
|參數     | `-NetworkMode`        |
|預設值    | `l2bridge`        |


# <a name="clustercidrtabclustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
叢集[子網範圍](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  | 
|---------|---------|
|參數     | `-ClusterCIDR`        |
|預設值    | `10.244.0.0/16`        |


# <a name="servicecidrtabservicecidr"></a>[ServiceCIDR](#tab/ServiceCIDR)
[服務子網範圍](./getting-started-kubernetes-windows.md#service-subnet-def)。

|  |  | 
|---------|---------|
|參數     | `-ServiceCIDR`        |
|預設值    | `10.96.0.0/12`        |


# <a name="kubednsserviceiptabkubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS 服務 IP](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster)。

|  |  | 
|---------|---------|
|參數     | `-KubeDnsServiceIP`        |
|預設值    | `10.96.0.10`        |


# <a name="interfacenametabinterfacename"></a>[介面名稱](#tab/InterfaceName)
Windows 主機的網路介面名稱。 您可以使用 `ipconfig` 來尋找此。

|  |  | 
|---------|---------|
|參數     | `-InterfaceName`        |
|預設值    | `Ethernet`        |


# <a name="logdirtablogdir"></a>[LogDir](#tab/LogDir)
Kubelet 和 kube proxy 記錄會重新導向至其個別輸出檔案的目錄。

|  |  | 
|---------|---------|
|參數     | `-LogDir`        |
|預設值    | `C:\k`        |


---

> [!tip]
> 您已從[先前](./creating-a-linux-master.md#collect-cluster-information)的 Linux 主機記下叢集子網、服務子網和 KUBE DNS IP

執行此動作之後，您應該能夠：
  * 使用 `kubectl get nodes` 來觀看已加入的 Windows 節點
  * 請參閱3個 powershell 視窗開啟，一個用於 `kubelet`，一個用於 `flanneld`，另一個用於 `kube-proxy`
  * 如需在節點上執行的 `flanneld`、`kubelet`和 `kube-proxy`，請參閱主機代理程式進程

如果成功，請繼續進行[後續步驟](#next-steps)。

## <a name="joining-a-tor-cluster"></a>加入 ToR 叢集 ##
> [!NOTE]
> 如果您[先前](./network-topologies.md#flannel-in-host-gateway-mode)選擇 Flannel 做為您的網路解決方案，可以略過本節。

若要這樣做，您必須遵循在[Kubernetes 上為上游 L3 路由拓撲設定 Windows Server 容器](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)的指示。 這包括確定您設定上游路由器，讓指派給節點的 pod CIDR 首碼對應到其各自的節點 IP。

假設新節點是由 `kubectl get nodes`列出為「就緒」，kubelet + kube-proxy 正在執行，而且您已設定上游 ToR 路由器，則您已準備好進行後續步驟。

## <a name="next-steps"></a>後續步驟 ##
在本節中，我們討論了如何將 Windows worker 加入我們的 Kubernetes 叢集。 現在您已準備好開始進行步驟5：

> [!div class="nextstepaction"]
> [加入 Linux 背景工作](./joining-linux-workers.md)

或者，如果您沒有任何 Linux 背景工作角色，您可以隨意跳到步驟6：

> [!div class="nextstepaction"]
> [部署 Kubernetes 資源](./deploying-resources.md)
