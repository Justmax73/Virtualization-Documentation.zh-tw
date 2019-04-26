---
title: 從頭建立 Kubernetes 主機
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 開始建立 Kubernetes 叢集主機。
keywords: kubernetes，1.13]，主機 linux
ms.openlocfilehash: 8a3fb073616d115ab84e6cc36f0fb6cedbcf1f7d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578249"
---
# <a name="creating-a-kubernetes-master"></a>建立 Kubernetes 主機 #
> [!NOTE]
> 本指南已確認 Kubernetes v1.13 上。 適用的 Kubernetes 版本的版本，因為此區段可能會使不保存適用於所有未來的版本，則為 true 的假設。 初始化使用 kubeadm Kubernetes 主機的官方文件可以找到[以下](https://kubernetes.io/docs/setup/independent/install-kubeadm/)。 只需上面啟用[混合作業系統排程一節](#enable-mixed-os-scheduling)。

> [!NOTE]  
> 最近更新的 Linux 電腦，才能最近更新;Kubernetes 母片資源像[kube dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)、 [kube 排程器](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)和[kube apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)有不已移植到 Windows 尚未。 

> [!tip]
> **Ubuntu 16.04**邁向量身訂做的 Linux 指示。 其他執行 Kubernetes 認證的 Linux 散發版本應該也會提供您可以使用替代的對等命令。 它們會也與相互溝通成功 Windows。


## <a name="initialization-using-kubeadm"></a>使用 kubeadm 初始化 ##
除非明確指定，否則以**root**身分執行以下的任何命令。

首先，取得到提升權限的根殼層：

```bash
sudo –s
```

請確定您的電腦是最新狀態：

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>安裝 Docker ###
若要能夠使用容器，您需要容器引擎，例如 Docker。 若要取得最新版本，您可以使用[下列指示](https://docs.docker.com/install/linux/docker-ce/ubuntu/)適用於 Docker 安裝。 您可以確認該 docker 已正確安裝，執行`hello-world`容器：

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>安裝 kubeadm ###
下載`kubeadm`二進位檔以便您 Linux 散發套件並初始化您的叢集。

> [!Important]  
> 根據您 Linux 散發套件，您可能需要取代`kubernetes-xenial`下方使用正確的[代號名稱](https://wiki.ubuntu.com/Releases)。

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>準備主要節點 ###
在 Linux 上的 Kubernetes 需要關閉交換空間：

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>初始化主要 ###
記下您的叢集子網路 (例如 10.244.0.0/16) 和服務子網路 (例如 10.96.0.0/12)，並初始化您主要使用 kubeadm:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

這可能需要數分鐘。 完成後，您應該會看到類似這個確認已初始化您的主要畫面：

![文字](media/kubeadm-init.png)

> [!tip]
> 您應該注意這個 kubeadm 加入命令。 應該 kubeadm 權杖到期時，您可以使用`kubeadm token create --print-join-command`若要建立新的權杖。

> [!tip]
> 如果您有想要的 Kubernetes 版本您想要使用，您可以傳遞`--kubernetes-version`旗標，以 kubeadm。

我們不尚未完成。 若要使用`kubectl`身為一般的使用者，執行下列__**在 unelevated、 非根使用者 shell 中**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
現在您可以使用 kubectl 編輯或檢視您的叢集的相關資訊。

### <a name="enable-mixed-os-scheduling"></a>啟用混合作業系統排程 ###
根據預設，某些 Kubernetes 資源會寫入它們正在排程的所有節點上的這類的方式。 不過，在 multi-OS 環境中，我們不希望 Linux 資源會影響或雙排程到 Windows 節點，反之亦然。 基於這個原因，我們需要套用[節點選取器](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)標籤。 

在這方面，我們會將修補 linux kube proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)到目標只 Linux。

首先，我們來建立目錄來儲存.yaml 資訊清單檔案：
```bash
mkdir -p kube/yaml && cd kube/yaml
```

確認更新策略，亦即`kube-proxy`DaemonSet 設為[RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

接下來，修補 DaemonSet 下載[此節點選取器](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)，並將它做為只目標 Linux 套用：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

一旦成功，您應該看到 」 節點選取器 」 的`kube-proxy`與任何其他 DaemonSets 設為 `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![文字](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>收集叢集資訊 ###
若要成功加入未來節點到主機，您應該追蹤的下列資訊：
  1. `kubeadm join` 命令，從輸出 （[此處](#initialize-master)）
    * 範例： `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. 叢集子網路時，定義`kubeadm init`（[此處](#initialize-master)）
    * 範例： `10.244.0.0/16`
  3. 服務子網路時，定義`kubeadm init`（[此處](#initialize-master)）
    * 範例： `10.96.0.0/12`
    * 您也可以找到使用 `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube dns 服務 IP 
    * 範例： `10.96.0.10`
    * 請參閱 「 叢集 IP 」 欄位使用 `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes`config`之後產生的檔案`kubeadm init`（[這裡](#initialize-master)）。 如果您依照指示，這可以在下列路徑中找到：
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>驗證主機 ##
幾分鐘之後，系統應該處於下列狀態：

  - 在`kubectl get pods -n kube-system`，會有適用於[Kubernetes 主機元件](https://kubernetes.io/docs/concepts/overview/components/#master-components)中的 pod`Running`狀態。
  - 呼叫`kubectl cluster-info`會顯示 Kubernetes 主機 API 伺服器 DNS 附加元件的相關資訊。
  
> [!tip]
> 因為 kubeadm 安裝網路功能不程式、 DNS pod 仍然可能會在`ContainerCreating`或`Pending`狀態。 也會切換至`Running`[選擇的網路解決方案](./network-topologies.md)之後的狀態。

## <a name="next-steps"></a>後續步驟 ## 
在本節中，我們會討論如何設定使用 kubeadm Kubernetes 主機。 現在您已經準備好進行步驟 3:

> [!div class="nextstepaction"]
> [選擇網路解決方案](./network-topologies.md)