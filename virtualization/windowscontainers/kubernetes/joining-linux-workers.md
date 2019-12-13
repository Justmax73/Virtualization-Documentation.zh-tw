---
title: 加入 Linux 節點
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 使用 v 1.14 將 Linux 節點加入至 Kubernetes 叢集。
keywords: kubernetes，1.14，windows，使用者入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 88207939c82bfe8ffa0b088cfd61cf4ab22cb10a
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909948"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>將 Linux 節點加入叢集

[設定 Kubernetes 主要節點](creating-a-linux-master.md)並[選取您想要的網路解決方案](network-topologies.md)之後，您就可以將 Linux 節點加入至您的叢集。 這需要在加入之前先[對 Linux 節點進行一些準備工作](joining-linux-workers.md#preparing-a-linux-node)。
> [!tip]
> Linux 指示是針對**Ubuntu 16.04**量身打造。 其他經認證可執行 Kubernetes 的 Linux 散發套件也應該提供您可以替代的對應命令。 它們也會順利與 Windows 交互操作。

## <a name="preparing-a-linux-node"></a>準備 Linux 節點

> [!NOTE]
> 除非另有明確指定，否則請在提高**許可權的根使用者 shell**中執行任何命令。

首先，進入根 shell：

```bash
sudo –s
```

請確定您的電腦是最新的：

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>安裝 Docker

若要能夠使用容器，您需要容器引擎，例如 Docker。 若要取得最新版本，您可以使用[這些指示](https://docs.docker.com/install/linux/docker-ce/ubuntu/)來安裝 Docker。 您可以藉由執行 `hello-world` 映射，確認 docker 已正確安裝：

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>安裝 kubeadm

下載 Linux 散發套件的 `kubeadm` 二進位檔，並初始化您的叢集。

> [!Important]  
> 視您的 Linux 散發套件而定，您可能需要將下列 `kubernetes-xenial` 取代為正確的[代號](https://wiki.ubuntu.com/Releases)。

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>停用交換

Linux 上的 Kubernetes 需要關閉交換空間：

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>（僅限 Flannel）啟用 iptables 的橋接 IPv4 流量

如果您選擇 Flannel 作為網路解決方案，建議您啟用橋接的 IPv4 流量至 iptables 鏈。 您應該已[為主要主機完成此](network-topologies.md#flannel-in-host-gateway-mode)作業，而現在必須針對要加入的 Linux 節點重複此動作。 您可以使用下列命令來完成此動作：

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>複製 Kubernetes 憑證

**如一般，（非根）使用者**，請執行下列3個步驟。

1. 建立 Linux 目錄的 Kubernetes：

```bash
mkdir -p $HOME/.kube
```

2. [從 master](./creating-a-linux-master.md#collect-cluster-information)複製 Kubernetes 憑證檔案（`$HOME/.kube/config`），並在背景工作角色中另存為 `$HOME/.kube/config`。

> [!tip]
> 您可以使用以 scp 為基礎的工具（例如[WinSCP](https://winscp.net/eng/download.php) ），在節點之間傳送設定檔。

3. 設定複製之設定檔的檔案擁有權，如下所示：

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>正在加入節點

最後，若要加入叢集，請執行我們稍[早記下](./creating-a-linux-master.md#initialize-master)的 `kubeadm join` 命令**做為 root**：

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

如果成功，您應該會看到類似的輸出：

![文字](./media/node-join.png)

## <a name="next-steps"></a>後續步驟

在本節中，我們討論了如何將 Linux 背景工作加入我們的 Kubernetes 叢集。 現在您已準備好開始進行步驟6：
> [!div class="nextstepaction"]
> [部署 Kubernetes 資源](./deploying-resources.md)