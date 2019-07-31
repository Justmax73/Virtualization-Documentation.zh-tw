---
title: 加入 Windows 節點
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 將 Windows 節點加入具有 v 1.14 的 Kubernetes 群集。
keywords: kubernetes、1.14、windows、快速入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882981"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>將 Windows Server 節點加入群集 #
在您[設定 Kubernetes 主節點](./creating-a-linux-master.md)並[選取您想要的網路方案](./network-topologies.md)後, 就可以加入 Windows Server 節點來組成群集。 在加入之前[, 必須先準備一些 Windows 節點](#preparing-a-windows-node)。

## <a name="preparing-a-windows-node"></a>準備 Windows 節點 ##
> [!NOTE]  
> Windows 章節中的所有程式碼片段都是在_提升權限_的 PowerShell 中執行。

### <a name="install-docker-requires-reboot"></a>安裝 Docker (需要重新開機) ###
Kubernetes 會使用[Docker](https://www.docker.com/)作為其容器引擎, 所以我們需要安裝它。 您可以遵循 [Docs 正式指示](../manage-docker/configure-docker-daemon.md#install-docker)、[Docker 指示](https://store.docker.com/editions/enterprise/docker-ee-server-windows)，或嘗試下列步驟：

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

如果重新開機之後, 您會看到下列錯誤:

![文字](media/docker-svc-error.png)

然後手動啟動 docker 服務:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>建立 "暫停" (基礎結構) 圖像 ###
> [!Important]
> 必須小心地考慮容器影像的衝突, 否則請務必小心。不具備預期的標籤會導致`docker pull`無法相容的容器影像, 從而導致諸如未確定`ContainerCreating`狀態等[部署問題](./common-problems.md#when-deploying-docker-containers-keep-restarting)。

現在 `docker` 已安裝，您需要準備 "pause" 映像，供 Kubernetes 用來準備基礎架構 Pod。 如此一來, 有三個步驟: 
  1. [拉中影像](#pull-the-image)
  2. [將它標記](#tag-the-image)為 microsoft/nanoserver: 最新
  3. 並[](#run-the-container)執行


#### <a name="pull-the-image"></a>拉入影像 ####     
 針對您的特定 Windows 版本, 拉入影像。 例如, 如果您執行的是 Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>標記影像 ####
本指南稍後將會用到的 Dockerfiles, 以尋找`:latest`影像標記。 標記您剛剛拉出的 nanoserver 影像, 如下所示:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>執行容器 ####
仔細檢查容器實際在您的電腦上執行:

```powershell
docker run microsoft/nanoserver:latest
```

您應該會看到如下所示的內容:

![文字](./media/docker-run-sample.png)

> [!tip]
> 如果您無法執行容器, 請參閱: 將[容器主機版本與容器影像搭配使用](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>為 Windows 目錄準備 Kubernetes ####
建立「Kubernetes for Windows」目錄來儲存 Kubernetes 二進位檔案, 以及任何部署腳本與 config 檔案。

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>複製 Kubernetes 憑證 #### 
將 Kubernetes 憑證檔案 ()`$HOME/.kube/config`[從 [主要](./creating-a-linux-master.md#collect-cluster-information)檔] 複製`C:\k`到這個新目錄。

> [!tip]
> 您可以使用[xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy)或[WinSCP](https://winscp.net/eng/download.php)等工具, 在節點之間傳輸配置檔案。

#### <a name="download-kubernetes-binaries"></a>下載 Kubernetes 二進位檔案 ####
若要能夠執行 Kubernetes, 您必須先下載`kubectl`、 `kubelet`和`kube-proxy`二進位檔案。 您可以從[最新版本](https://github.com/kubernetes/kubernetes/releases/)之檔案的`CHANGELOG.md`連結下載這些檔案。
 - 例如, 以下是[v 1.14 節點二進位檔案](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries)。
 - 使用像是 [[展開](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6)] 封存之類的工具來解壓縮封存並將二進位`C:\k\`儲存在中。

#### <a name="optional-setup-kubectl-on-windows"></a>可選在 Windows 上設定 kubectl ####
如果您想要從 Windows 控制群集, 您可以使用`kubectl`命令來執行此操作。 首先, 若要`kubectl`使其在`C:\k\`目錄外可用, 請`PATH`修改環境變數:

```powershell
$env:Path += ";C:\k"
```

如果要讓此變更具有永久性，請在電腦目標修改變數：

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

接下來, 我們會驗證[群集憑證](#copy-kubernetes-certificate)是否有效。 若要設定`kubectl`尋找設定檔的位置, 您可以傳遞`--kubeconfig`參數或修改`KUBECONFIG`環境變數。 例如，如果設定位於 `C:\k\config`：

```powershell
$env:KUBECONFIG="C:\k\config"
```

若要讓這項設定對目前使用者的範圍具有永久性：

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

最後, 若要檢查是否已正確探索設定, 您可以使用:

```powershell
kubectl config view
```

如果您收到連線錯誤，

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

您應該仔細檢查 kubeconfig 位置, 或嘗試再次複製。

如果您看到 [沒有錯誤], 節點現在已準備好加入群集。

## <a name="joining-the-windows-node"></a>加入 Windows 節點 ##
視[您選擇的網路方案](./network-topologies.md)而定, 您可以:
1. [將 Windows Server 節點加入 Flannel (vxlan 或主機-gw) 群集](#joining-a-flannel-cluster)
2. [使用 ToR 開關將 Windows Server 節點加入群集](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>加入 Flannel 群集 ###
[這個 Microsoft 文件庫](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay)有一個 Flannel 部署腳本集合, 可協助您將這個節點加入群集。

下載[Flannel start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)腳本, 應該將其解壓縮至`C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

假設您已[準備好 Windows 節點](#preparing-a-windows-node), 且`c:\k`目錄看起來像這樣, 您就可以加入節點了。

![文字](./media/flannel-directory.png)

#### <a name="join-node"></a>加入節點 #### 
若要簡化加入 Windows 節點的程式, 您只需要執行單一 Windows 腳本來啟動`kubelet`、 `kube-proxy` `flanneld`、以及加入節點。

> [!Note]
> [start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)參考[安裝。](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1)ps1 會下載其他檔案, 例如`flanneld`可執行檔和[基礎結構盒的 Dockerfile](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) ,*並為您安裝*。 針對重迭網路模式, 將會開啟本機 UDP 埠4789的[防火牆](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111)。 在第一次建立盒式網路的新外部 vSwitch 時, 可能會開啟/關閉多個 powershell 視窗, 並顯示幾秒鐘的網路中斷。

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
指派給 Windows 節點的 IP 位址。 您可以使用`ipconfig`來尋找這種情況。

|  |  | 
|---------|---------|
|參數     | `-ManagementIP`        |
|預設值    | n.A. **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
已選取為`l2bridge` [網路解決方案](./network-topologies.md)的網路模式 (flannel `overlay`主機-gw) 或 (flannel vxlan)。

> [!Important] 
> `overlay` 網路模式 (flannel vxlan) 需要 Kubernetes v 1.14 二進位檔案 (或更新版本) 與[KB4489899](https://support.microsoft.com/help/4489899)。

|  |  | 
|---------|---------|
|參數     | `-NetworkMode`        |
|預設值    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[群集子網範圍](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  | 
|---------|---------|
|參數     | `-ClusterCIDR`        |
|預設值    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
[服務子網範圍](./getting-started-kubernetes-windows.md#service-subnet-def)。

|  |  | 
|---------|---------|
|參數     | `-ServiceCIDR`        |
|預設值    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS 服務 IP](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster)。

|  |  | 
|---------|---------|
|參數     | `-KubeDnsServiceIP`        |
|預設值    | `10.96.0.10`        |


# [<a name="interfacename"></a>InterfaceName](#tab/InterfaceName)
Windows 主機網路介面的名稱。 您可以使用`ipconfig`來尋找這種情況。

|  |  | 
|---------|---------|
|參數     | `-InterfaceName`        |
|預設值    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Kubelet 和 kube proxy 記錄的目錄會重新導向到各自的輸出檔案。

|  |  | 
|---------|---------|
|參數     | `-LogDir`        |
|預設值    | `C:\k`        |


---

> [!tip]
> 您已在[舊版](./creating-a-linux-master.md#collect-cluster-information)Linux 主機上記下群集子網、服務子網及 KUBE DNS IP

執行此動作後, 您應該能夠:
  * 使用來查看加入的 Windows 節點 `kubectl get nodes`
  * 請參閱開啟3個 powershell 視窗, `kubelet`一個適用`flanneld`于, 另一個用於 `kube-proxy`
  * 請參閱、 `kubelet`和`kube-proxy`在節點`flanneld`上執行的主機代理程式進程

如果成功, 請繼續執行[下一個步驟](#next-steps)。

## <a name="joining-a-tor-cluster"></a>加入 ToR 群集 ##
> [!NOTE]
> 如果您[先前](./network-topologies.md#flannel-in-host-gateway-mode)選擇 [Flannel] 做為您的網路解決方案, 您可以略過本節。

若要這樣做, 您必須依照在[Kubernetes 上針對上游 L3 路由拓撲進行設定 Windows Server 容器](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)的指示進行。 這包括確認您已設定上游路由器, 讓指派給節點的 pod CIDR 首碼會對應到各自的節點 IP。

假設 [新節點] 列為 [就緒] `kubectl get nodes`, kubelet + kube proxy 正在執行, 而且您已經設定了上游 ToR 路由器, 您就可以進行後續步驟了。

## <a name="next-steps"></a>後續步驟 ##
在本節中, 我們將說明如何將 Windows 工作人員加入我們的 Kubernetes 群集。 現在您已準備好進行步驟 5:

> [!div class="nextstepaction"]
> [加入 Linux 工作人員](./joining-linux-workers.md)

或者, 如果您沒有任何 Linux 工人可隨時跳過, 請繼續執行步驟 6:

> [!div class="nextstepaction"]
> [部署 Kubernetes 資源](./deploying-resources.md)
