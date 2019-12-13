---
title: 網路拓撲
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows 和 Linux 上支援的網路拓撲。
keywords: kubernetes，1.14，windows，使用者入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910308"
---
# <a name="network-solutions"></a>Network Solutions #

[設定 Kubernetes 主要節點](./creating-a-linux-master.md)之後，您就可以選擇網路解決方案。 有多種方式可讓虛擬[群集子網](./getting-started-kubernetes-windows.md#cluster-subnet-def)在節點之間路由傳送。 針對在 Windows 上的 Kubernetes，請選擇下列其中一個選項：

1. 使用 CNI 外掛程式（例如[Flannel](#flannel-in-vxlan-mode) ）為您設定重迭網路。
2. 使用 CNI 外掛程式（例如[Flannel](#flannel-in-host-gateway-mode) ）來為您程式（使用 l2bridge 網路模式）。
3. 設定智慧型[主要機架（ToR）交換器](#configuring-a-tor-switch)來路由子網。

> [!tip]  
> Windows 上有第四種網路解決方案，會利用 Open vSwitch （Ovs-es）和 Open 虛擬網路（OVN）。 記載這不在本檔的範圍內，但您可以閱讀[這些指示](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)加以設定。

## <a name="flannel-in-vxlan-mode"></a>Vxlan 模式中的 Flannel

Vxlan 模式中的 Flannel 可以用來設定可設定的虛擬重迭網路，其使用 VXLAN 通道在節點之間路由封包。

### <a name="prepare-kubernetes-master-for-flannel"></a>準備 Flannel 的 Kubernetes 主機
我們建議您在叢集中的[Kubernetes 主機](./creating-a-linux-master.md)上進行一些次要準備。 使用 Flannel 時，建議您啟用 iptables 鏈的橋接 IPv4 流量。 這可以使用下列命令來完成：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>下載 & 設定 Flannel ###
下載最新的 Flannel 資訊清單：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

有兩個區段您應該修改來啟用 vxlan 網路後端：

1. 在 `kube-flannel.yml`的 [`net-conf.json`] 區段中，再次檢查：
 * 叢集子網（例如 "10.244.0.0/16"）會視需要設定。
 * VNI 4096 設定于後端
 * 埠4789已在後端中設定
2. 在 `kube-flannel.yml`的 [`cni-conf.json`] 區段中，將網路名稱變更為 [`"vxlan0"`]。

套用上述步驟之後，您的 `net-conf.json` 看起來應該如下所示：
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
> VNI 必須設為4096和埠4789，Linux 上的 Flannel 才能與 Windows 上的 Flannel 交互操作。 即將推出其他 VNIs 的支援。 如需這些欄位的說明，請參閱[VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 。

您的 `cni-conf.json` 看起來應該如下所示：
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
> 如需上述選項的詳細資訊，請參閱適用于 Linux 的官方 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、 [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)和[bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)外掛程式檔。

### <a name="launch-flannel--validate"></a>啟動 Flannel & 驗證 ###
使用下列步驟啟動 Flannel：

```bash
kubectl apply -f kube-flannel.yml
```

接下來，由於 Flannel pod 是以 Linux 為基礎，因此，請將 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修補程式套用至僅以 linux 為目標的 `kube-flannel-ds` DaemonSet （我們稍後會在加入時于 Windows 上啟動 Flannel "flanneld" 主機代理程式進程）：

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果任何節點不是以 x86-64 為基礎，請將上方的 `-amd64` 取代為您的處理器架構。

幾分鐘後，如果已部署 Flannel pod 網路，您應該會看到所有 pod 都是執行中狀態。

```bash
kubectl get pods --all-namespaces
```

![文字](media/kube-master.png)

Flannel DaemonSet 也應該套用 NodeSelector `beta.kubernetes.io/os=linux`。

```bash
kubectl get ds -n kube-system
```

![文字](media/kube-daemonset.png)

> [!tip]  
> 針對其餘的 flannel-ds-* Daemonset，這些可以忽略或刪除，因為如果沒有符合該處理器架構的節點，它們就不會排程。

> [!tip]  
> 搞混了嗎？ 以下是[kube-flannel yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) for flannel v 0.11.0 的完整範例，其中已針對預設叢集子網 `10.244.0.0/16`預先套用這些步驟。

成功之後，請繼續進行[後續步驟](#next-steps)。

## <a name="flannel-in-host-gateway-mode"></a>在主機閘道模式中 Flannel

除了[Flannel vxlan](#flannel-in-vxlan-mode)，Flannel 網路功能的另一個選項是*主機閘道模式*（主機 gw），這需要使用目標節點的主機位址做為下一個躍點，將每個節點上的靜態路由程式設計到其他節點的 pod 子網。

### <a name="prepare-kubernetes-master-for-flannel"></a>準備 Flannel 的 Kubernetes 主機

我們建議您在叢集中的[Kubernetes 主機](./creating-a-linux-master.md)上進行一些次要準備。 使用 Flannel 時，建議您啟用 iptables 鏈的橋接 IPv4 流量。 這可以使用下列命令來完成：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>下載 & 設定 Flannel ###
下載最新的 Flannel 資訊清單：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

您需要變更一個檔案，才能在 Windows/Linux 上啟用主機 gw 網路功能。

在 kube-flannel 的 [`net-conf.json`] 區段中，再次檢查下列各項：
1. 使用的網路後端類型設定為 `host-gw`，而不是 `vxlan`。
2. 叢集子網（例如 "10.244.0.0/16"）會視需要設定。

套用2個步驟之後，您的 `net-conf.json` 看起來應該如下所示：
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>啟動 Flannel & 驗證 ###
使用下列步驟啟動 Flannel：

```bash
kubectl apply -f kube-flannel.yml
```

接下來，由於 Flannel pod 是以 Linux 為基礎，因此，請將 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修補程式套用 `kube-flannel-ds` 至僅限目標 linux （我們稍後會在 Windows 上啟動 Flannel "flanneld" 主機代理程式進程）：

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果任何節點不是以 x86-64 為基礎，請以所需的處理器架構取代上面的 `-amd64`。

幾分鐘後，如果已部署 Flannel pod 網路，您應該會看到所有 pod 都是執行中狀態。

```bash
kubectl get pods --all-namespaces
```

![文字](media/kube-master.png)

Flannel DaemonSet 也應該套用 NodeSelector。

```bash
kubectl get ds -n kube-system
```

![文字](media/kube-daemonset.png)

> [!tip]  
> 針對其餘的 flannel-ds-* Daemonset，這些可以忽略或刪除，因為如果沒有符合該處理器架構的節點，它們就不會排程。

> [!tip]  
> 搞混了嗎？ 以下是[kube-flannel yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) For flannel v 0.11.0 的完整範例，其中包含針對預設叢集子網 `10.244.0.0/16`預先套用的兩個步驟。

成功之後，請繼續進行[後續步驟](#next-steps)。

## <a name="configuring-a-tor-switch"></a>設定 ToR 交換器 ##
> [!NOTE]
> 如果您選擇 [ [Flannel] 做為網路解決方案](#flannel-in-host-gateway-mode)，則可以略過本節。
ToR 參數的設定會在實際節點之外發生。 如需這種情況的詳細資訊，請參閱[官方 Kubernetes](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)檔。


## <a name="next-steps"></a>後續步驟 ## 
在本節中，我們討論了如何挑選和設定網路解決方案。 現在您已準備好進行步驟4：

> [!div class="nextstepaction"]
> [加入 Windows worker](./joining-windows-workers.md)