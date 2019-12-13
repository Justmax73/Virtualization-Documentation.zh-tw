---
title: 從頭建立 Kubernetes 主機
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 建立 Kubernetes 叢集主機。
keywords: kubernetes，1.14，master，linux
ms.openlocfilehash: b1ec23b039ce6f5c42859452ecf3a8a5b35e006c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910418"
---
# <a name="creating-a-kubernetes-master"></a>建立 Kubernetes 主機 #
> [!NOTE]
> 本指南已于 Kubernetes v 1.14 進行驗證。 由於從版本到版本的 Kubernetes 變動性，因此本節可能會假設所有未來版本都不會有 true。 您可以在[這裡](https://kubernetes.io/docs/setup/independent/install-kubeadm/)找到使用 Kubeadm 初始化 Kubernetes 主機的官方檔。 請直接在上面啟用[混合 OS 排程一節](#enable-mixed-os-scheduling)。

> [!NOTE]  
> 最近更新的 Linux 機器必須遵循下列步驟：Kubernetes[的主要](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)資源（例如[kube](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)）尚未移植到[Windows，但](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)尚未將其移植到 Windows。 

> [!tip]
> Linux 指示是針對**Ubuntu 16.04**量身打造。 其他經認證可執行 Kubernetes 的 Linux 散發套件也應該提供您可以替代的對應命令。 它們也會順利與 Windows 交互操作。


## <a name="initialization-using-kubeadm"></a>使用 kubeadm 初始化 ##
除非另有明確指定，否則請以**root**身分執行下列任何命令。

首先，進入提升許可權的根命令介面：

```bash
sudo –s
```

請確定您的電腦是最新的：

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>安裝 Docker ###
若要能夠使用容器，您需要容器引擎，例如 Docker。 若要取得最新版本，您可以使用[這些指示](https://docs.docker.com/install/linux/docker-ce/ubuntu/)來安裝 Docker。 您可以藉由執行 `hello-world` 容器，確認 docker 已正確安裝：

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>安裝 kubeadm ###
下載 Linux 散發套件的 `kubeadm` 二進位檔，並初始化您的叢集。

> [!Important]  
> 視您的 Linux 散發套件而定，您可能需要將下列 `kubernetes-xenial` 取代為正確的[代號](https://wiki.ubuntu.com/Releases)。

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>準備主要節點 ###
Linux 上的 Kubernetes 需要關閉交換空間：

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>初始化 master ###
記下您的叢集子網（例如 10.244.0.0/16）和服務子網（例如 10.96.0.0/12），並使用 kubeadm 初始化您的主伺服器：

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

這可能需要數分鐘。 完成後，您應該會看到類似下面的畫面，確認您的主應用程式已初始化：

![文字](media/kubeadm-init.png)

> [!tip]
> 您應該記下此 kubeadm join 命令。 應 kubeadm token 到期後，您可以使用 `kubeadm token create --print-join-command` 建立新的權杖。

> [!tip]
> 如果您想要使用所需的 Kubernetes 版本，您可以將 `--kubernetes-version` 旗標傳遞至 kubeadm。

我們尚未完成。 若要使用 `kubectl` 做為一般使用者，請 __**在停權一致非 root 使用者介面中**執行下列命令__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
現在您可以使用 kubectl 來編輯或查看叢集的相關資訊。

### <a name="enable-mixed-os-scheduling"></a>啟用混合 OS 排程 ###
根據預設，某些 Kubernetes 資源會以其在所有節點上排程的方式寫入。 不過，在多 OS 環境中，我們不希望 Linux 資源干擾或重複排程到 Windows 節點，反之亦然。 基於這個理由，我們需要套用[NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)標籤。 

就這一點而言，我們要將 linux kube-proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)修補成僅以 linux 為目標。

首先，讓我們建立一個目錄來儲存 yaml 資訊清單檔案：
```bash
mkdir -p kube/yaml && cd kube/yaml
```

確認 `kube-proxy` DaemonSet 的更新策略已設定為[RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)：

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

接下來，下載[此 nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)以修補 DaemonSet，並將其套用至僅適用于 Linux 的目標：

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

成功之後，您應該會看到 `kube-proxy` 的「節點選取器」，以及任何其他 Daemonset 設定為 `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![文字](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>收集叢集資訊 ###
若要成功將未來節點加入至主伺服器，您應該追蹤下列資訊：
  1. 從輸出 `kubeadm join` 命令（[這裡](#initialize-master)）
    * 範例： `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. `kubeadm init` 期間定義的叢集子網（[這裡](#initialize-master)）
    * 範例： `10.244.0.0/16`
  3. `kubeadm init` 期間定義的服務子網（[這裡](#initialize-master)）
    * 範例： `10.96.0.0/12`
    * 也可以使用 `kubectl cluster-info dump | grep -i service-cluster-ip-range` 找到
  4. Kube-dns 服務 IP 
    * 範例： `10.96.0.10`
    * 可以在使用 `kubectl get svc/kube-dns -n kube-system` 的 [叢集 IP] 欄位中找到
  5. 在 `kubeadm init` 之後產生的 Kubernetes `config` 檔案（[這裡](#initialize-master)）。 如果您已遵循指示進行，可以在下列路徑中找到：
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>驗證主伺服器 ##
幾分鐘之後，系統應該處於下列狀態：

  - 在 [`kubectl get pods -n kube-system`] 底下，會有適用于 `Running` 狀態中[Kubernetes 主要元件](https://kubernetes.io/docs/concepts/overview/components/#master-components)的 pod。
  - 除了 DNS 附加元件之外，呼叫 `kubectl cluster-info` 也會顯示 Kubernetes 主要 API 伺服器的相關資訊。
  
> [!tip]
> 由於 kubeadm 不會設定網路功能，因此 DNS pod 可能仍處於 `ContainerCreating` 或 `Pending` 狀態。 在[選擇網路解決方案](./network-topologies.md)之後，它們會切換為 `Running` 狀態。

## <a name="next-steps"></a>後續步驟 ## 
在本節中，我們討論到如何使用 kubeadm 來設定 Kubernetes 主機。 現在您已準備好開始進行步驟3：

> [!div class="nextstepaction"]
> [選擇網路解決方案](./network-topologies.md)