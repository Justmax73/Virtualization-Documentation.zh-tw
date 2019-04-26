---
title: 網路拓撲
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: 在 Windows 和 Linux 上支援的網路拓撲。
keywords: kubernetes，1.13，windows，開始使用
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 9f96fcc80c533b74ab46d93beecc7ca8629ce395
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576129"
---
# <a name="network-solutions"></a>網路解決方案 #

一旦您已[安裝的 Kubernetes 主要節點](./creating-a-linux-master.md)您準備好挑選網路的解決方案。 有多個方式可以跨節點，讓虛擬[叢集子網路](./getting-started-kubernetes-windows.md#cluster-subnet-def)路由。 選擇其中一個下列選項適用於 Kubernetes Windows 上現今：

1. 使用 CNI 外掛程式，例如[Flannel](#flannel-in-vxlan-mode)設定為您的覆疊網路。
2. 為您使用程式路由到例如[Flannel](#flannel-in-host-gateway-mode) CNI 外掛程式。
3. 設定智慧[頂端 rack (ToR) 切換](#configuring-a-tor-switch)至路由子網路。

> [!tip]  
> 沒有第四個網路上運用開啟 vSwitch (OvS) 的 Windows 和開啟虛擬網路 (OVN) 的解決方案。 記載此超出範圍這份文件，但您可以閱讀[這些指示](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)來設定它。

## <a name="flannel-in-vxlan-mode"></a>Flannel vxlan 模式

Vxlan 模式 flannel 可用來安裝程式會使用 VXLAN 通道路由節點之間的封包的可設定的虛擬覆疊網路。

### <a name="prepare-kubernetes-master-for-flannel"></a>準備 flannel Kubernetes 主機
在我們的叢集中[Kubernetes 主機](./creating-a-linux-master.md)建議一些次要的準備。 建議使用 Flannel 時讓 iptables 鏈結的橋接的 IPv4 流量。 這可以使用下列命令：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>下載 & 設定 Flannel ###
下載最新的 Flannel 資訊清單：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

有兩個區段，您應該修改為啟用 vxlan 網路後端：

1. 在`net-conf.json`一節您`kube-flannel.yml`，再次檢查：
 * 叢集子網路 (例如 「 10.244.0.0/16 」) 會設定為所需。
 * VNI 4096 設在後端
 * 在後端設定連接埠 4789
2. 在`cni-conf.json`一節您`kube-flannel.yml`，網路將名稱變更為`"vxlan0"`。

套用上述步驟之後, 您`net-conf.json`應該看起來如下：
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]  
> VNI 必須設為 4096 和連接埠 4789 flannel linux 與 Windows 上的 Flannel 相互溝通。 即將推出對其他 VNIs 的支援。 這些欄位的說明，請參閱[VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 。

您`cni-conf.json`應該看起來如下：
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]  
> 如需有關上述選項的詳細資訊，請洽詢官方 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、[連接埠對應](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)，以及[橋接器](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)外掛程式的文件 Linux。

### <a name="launch-flannel--validate"></a>啟動的 Flannel & 驗證 ###
啟動 Flannel 使用：

```bash
kubectl apply -f kube-flannel.yml
```

接下來，由於 Flannel pod Linux 型，套用 Linux[節點選取器](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修補程式來`kube-flannel-ds`DaemonSet 只目標 Linux （我們將會啟動 Flannel 」 flanneld 「 主機代理程式上的處理序 Windows 稍後加入時）：

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果任何節點不是 x86 64，取代`-amd64`上述與您的處理器架構。

在幾分鐘的時間之後, 您應該會看到做為執行如果 Flannel pod 網路部署的所有 pod。

```bash
kubectl get pods --all-namespaces
```

![文字](media/kube-master.png)

Flannel DaemonSet 應該也有節點選取器`beta.kubernetes.io/os=linux`套用。

```bash
kubectl get ds -n kube-system
```

![文字](media/kube-daemonset.png)

> [!tip]  
> 剩餘的 flannel-ds-* DaemonSets，這些可以會忽略或刪除，因為它們不會排程如果沒有比對該處理器架構的節點。

> [!tip]  
> 混淆？ 以下是 Flannel v0.11.0 預先套用預設叢集子網路的這些步驟的完整[範例 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) `10.244.0.0/16`。

一旦成功，繼續[下一個步驟](#next-steps)。

## <a name="flannel-in-host-gateway-mode"></a>Flannel 在主機閘道模式

與[Flannel vxlan](#flannel-in-vxlan-mode)，一起 Flannel 網路功能的另一個選項是*主機閘道模式*(主機-gw)，這牽涉到其他節點的 pod 子網路使用目標節點的主機位址為下一個躍點到每個節點上的靜態路徑的程式設計。

### <a name="prepare-kubernetes-master-for-flannel"></a>準備 flannel Kubernetes 主機

在我們的叢集中[Kubernetes 主機](./creating-a-linux-master.md)建議一些次要的準備。 建議使用 Flannel 時讓 iptables 鏈結的橋接的 IPv4 流量。 這可以使用下列命令：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>下載 & 設定 Flannel ###
下載最新的 Flannel 資訊清單：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

還有一個檔案，您需要變更，以便啟用主機 gw 跨兩個 Windows/Linux 的網路功能。

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

### <a name="launch-flannel--validate"></a>啟動的 Flannel & 驗證 ###
啟動 Flannel 使用：

```bash
kubectl apply -f kube-flannel.yml
```

接下來，由於 Flannel pod Linux 型，套用到我們 Linux[節點選取器](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修補程式`kube-flannel-ds`DaemonSet 只目標 Linux （我們將會啟動 Flannel 」 flanneld 「 主機代理程式上的處理序 Windows 稍後加入時）：

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果任何節點不是 x86 64，取代`-amd64`上方所需的處理器架構。

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
> 剩餘的 flannel-ds-* DaemonSets，這些可以會忽略或刪除，因為它們不會排程如果沒有比對該處理器架構的節點。

> [!tip]  
> 混淆？ 以下是預先套用預設叢集子網路的這些 2 個步驟的完整[範例 kube flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml)適用於 Flannel v0.11.0 `10.244.0.0/16`。

一旦成功，繼續[下一個步驟](#next-steps)。

## <a name="configuring-a-tor-switch"></a>設定 ToR 交換器 ##
> [!NOTE]
> 如果您選擇[Flannel 做為您的網路解決方案](#flannel-in-host-gateway-mode)，您可以略過本節。
您實際的節點以外，就會發生 ToR 開關的設定。 如需有關這的詳細資訊，請參閱[正式的 Kubernetes 文件](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)。


## <a name="next-steps"></a>後續步驟 ## 
在此區段中，我們會討論如何選擇，並將設定的網路解決方案。 現在您已經準備好進行步驟 4:

> [!div class="nextstepaction"]
> [加入 Windows 工作者](./joining-windows-workers.md)