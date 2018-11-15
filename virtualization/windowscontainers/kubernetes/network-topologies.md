---
title: 網路拓撲
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 在 Windows 和 Linux 上支援的網路拓撲。
keywords: kubernetes，1.12，windows，開始使用
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: bcbd7b530b58b663305ea5d8b84a75eaf971f997
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948057"
---
# <a name="network-solutions"></a>網路解決方案 #

一旦您已[安裝的 Kubernetes 主要節點](./creating-a-linux-master.md)您準備好選取的網路功能的方案。 有多種方法可以讓虛擬[叢集子網路](./getting-started-kubernetes-windows.md#cluster-subnet-def)可路由傳送到節點。 選擇其中一個下列選項適用於 Kubernetes Windows 上現今：

1. 使用第三方 CNI 外掛程式，例如[Flannel](network-topologies.md#flannel-in-host-gateway-mode)安裝路由來為您。
1. 設定智慧[頂端 rack (ToR) 切換](network-topologies.md#configuring-a-tor-switch)至路由子網路。

> [!tip]  
> 沒有第三個網路上運用開啟 vSwitch (OvS) 的 Windows 和開啟虛擬網路 (OVN) 的解決方案。 記載此超出範圍，此文件，但您可以閱讀[這些指示](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)來設定它。

## <a name="flannel-in-host-gateway-mode"></a>Flannel 主機閘道模式

Flannel 網路功能的可用選項的其中一個是*主機閘道模式*(主機-gw)，這涉及所有節點上 pod 子網路之間靜態路徑的設定。
> [!NOTE]  
> 這是不同 Flannel，它會使用 VXLAN 封裝，並在現在開發中的*覆疊*網路模式。 觀看這個空間...

### <a name="prepare-kubernetes-master-for-flannel"></a>準備 flannel Kubernetes 主機

在我們的叢集中[Kubernetes 主機](./creating-a-linux-master.md)建議一些次要的準備。 建議使用 Flannel 時讓 iptables 鏈結的橋接的 IPv4 流量。 這可以使用下列命令：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>下載和設定 Flannel ###
下載最新的 Flannel 資訊清單：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

有兩件事您需要執行啟用主機 gw 跨兩個 Windows/Linux 的網路功能。

在`net-conf.json`一節的您 kube flannel.yml，您可再次檢查：
1. 網路後端所使用的類型設定為`host-gw`而不是`vxlan`。
2. 叢集子網路 (例如 「 10.244.0.0/16 」) 會設定為所需。

套用 2 個步驟之後, 您`net-conf.json`應該看起來如下：
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>啟動 Flannel 與驗證 ###
啟動 Flannel 使用：

```bash
kubectl apply -f kube-flannel.yml
```

接下來，由於 Flannel pod Linux 型，套用到我們 Linux[節點選取器](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修補程式`kube-flannel-ds`DaemonSet 只目標 Linux （我們將會啟動 Flannel 」 flanneld 「 主機代理程式上的程序 Windows 稍後加入時）：

```
kubectl patch ds/kube-flannel-ds --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

在幾分鐘的時間之後, 您應該會看到做為執行如果 Flannel pod 網路部署的所有 pod。

```bash
kubectl get pods --all-namespaces
```

![文字](media/kube-master.png)

Flannel DaemonSet 應該也可以套用節點選取器。

```bash
kubectl get ds -n kube-system
```

![文字](media/kube-daemonset.png)
> [!tip]  
> 混淆？ 以下是這些預先套用預設叢集子網路的 2 個步驟的完整[範例 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml)針對 Flannel v0.9.1 `10.244.0.0/16`。

## <a name="configuring-a-tor-switch"></a>設定 ToR 交換器 ##
> [!NOTE]
> 如果您選擇[Flannel 做為您的網路解決方案](#flannel-in-host-gateway-mode)，您可以略過本節。
您實際節點以外，就會發生 ToR 開關的設定。 如需有關這的詳細資訊，請參閱[正式的 Kubernetes 文件](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)。


## <a name="next-steps"></a>後續步驟 ## 
在此區段中，我們會討論如何選取的網路功能的方案。 現在您已經準備好進行步驟 4:

> [!div class="nextstepaction"]
> [加入 Windows 工作者](./joining-windows-workers.md)