---
title: 加入 Linux 節點
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Linux 節點加入 v1.14 Kubernetes 叢集。
keywords: kubernetes，1.14，windows，開始使用
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 88207939c82bfe8ffa0b088cfd61cf4ab22cb10a
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622943"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>將 Linux 節點加入叢集

一旦您有[設定 Kubernetes 主機的節點](creating-a-linux-master.md)，並[選取您想要的網路的解決方案](network-topologies.md)，您準備好將 Linux 節點加入叢集。 這需要一些[準備 Linux 節點上的](joining-linux-workers.md#preparing-a-linux-node)，才能將加入。
> [!tip]
> **Ubuntu 16.04**邁向量身訂做的 Linux 指示。 其他執行 Kubernetes 認證的 Linux 散發版本應該也會提供您可以使用替代的對等命令。 它們會也與相互溝通成功 Windows。

## <a name="preparing-a-linux-node"></a>準備 Linux 節點

> [!NOTE]
> 除非明確指定，否則執行任何命令在**提升權限，根使用者殼層**。

首先，取得根殼層到：

```bash
sudo –s
```

請確定您的電腦是最新狀態：

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>安裝 Docker

若要能夠使用容器，您需要容器引擎，例如 Docker。 若要取得最新版本，您可以使用[下列指示](https://docs.docker.com/install/linux/docker-ce/ubuntu/)適用於 Docker 安裝。 您可以確認該 docker 已正確安裝，執行`hello-world`映像：

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>安裝 kubeadm

下載`kubeadm`二進位檔以便您 Linux 散發套件並初始化您的叢集。

> [!Important]  
> 根據您 Linux 散發套件，您可能需要取代`kubernetes-xenial`下方使用正確的[代號名稱](https://wiki.ubuntu.com/Releases)。

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>停用交換

在 Linux 上的 Kubernetes 需要關閉交換空間：

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(僅 flannel)讓 iptables 橋接的 IPv4 流量

如果您選擇 Flannel 做為您網路的解決方案，建議您啟用橋接 iptables 鏈結的 IPv4 流量。 您應該有[已經完成這個的主要動作](network-topologies.md#flannel-in-host-gateway-mode)，現在需要重複它打算加入 Linux 節點。 它也可以在完成使用下列命令：

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>複製 Kubernetes 的憑證

**以一般，（非根） 的使用者**，請執行下列 3 個步驟。

1. 建立 Kubernetes Linux 目錄：

```bash
mkdir -p $HOME/.kube
```

2. 複製的 Kubernetes 的憑證檔案 (`$HOME/.kube/config`)[從主機](./creating-a-linux-master.md#collect-cluster-information)將儲存為`$HOME/.kube/config`背景工作上。

> [!tip]
> 您可以使用 scp 為基礎的工具，例如[WinSCP](https://winscp.net/eng/download.php)節點之間傳輸設定檔。

3. 設定已複製的組態檔的檔案擁有權，如下所示：

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>將節點

最後，若要加入叢集，執行`kubeadm join`[我們前面所向下](./creating-a-linux-master.md#initialize-master)**做為根**命令：

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

如果成功，您應該會看到類似輸出如下：

![文字](./media/node-join.png)

## <a name="next-steps"></a>後續步驟

在此區段中，我們會討論如何 Linux 工作者加入我們的 Kubernetes 叢集。 現在您已經準備好進行步驟 6:
> [!div class="nextstepaction"]
> [部署 Kubernetes 資源](./deploying-resources.md)