---
title: 網路拓撲
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows 和 Linux 支援的網路拓撲。
keywords: kubernetes、1.14、windows、快速入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 42cb47ba4f3e22163869d094bd0c9cff415a43b0
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2019
ms.locfileid: "9884979"
---
# <a name="network-solutions"></a>網路解決方案 #

完成[設定 Kubernetes 主節點](./creating-a-linux-master.md)後, 您就可以選擇網路解決方案了。 有多種方式可讓虛擬[群集子網](./getting-started-kubernetes-windows.md#cluster-subnet-def)可跨節點進行路由。 在 Windows 今日選取下列其中一個 Kubernetes 選項:

1. 使用 CNI 外掛程式 (例如 [ [Flannel](#flannel-in-vxlan-mode) ]) 來為您設定重迭的網路。
2. 使用 CNI 外掛程式 (例如 [ [Flannel](#flannel-in-host-gateway-mode) ]) 來為您計畫路線 (使用 l2bridge 網路模式)。
3. 設定智慧[的機架 (ToR) 開關](#configuring-a-tor-switch)來傳送子網。

> [!tip]  
> Windows 上有四個網路解決方案, 可利用開啟的 vSwitch (OvS), 並開啟虛擬網路 (OVN)。 記錄這份檔超出範圍, 但您可以閱讀[這些指示](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay)來設定。

## <a name="flannel-in-vxlan-mode"></a>在 vxlan 模式中 Flannel

Vxlan 模式中的 Flannel 可用來設定可設定的虛擬重迭網路, 該網路使用 VXLAN 隧道在節點之間路由包。

### <a name="prepare-kubernetes-master-for-flannel"></a>為 Flannel 準備 Kubernetes 主版
我們建議在群集中的[Kubernetes 主機](./creating-a-linux-master.md)上進行一些次要準備。 使用 Flannel 時, 建議您啟用橋接的 IPv4 流量至 iptables 鏈。 您可以使用下列命令完成這項作業:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>下載 & 設定 Flannel ###
下載最新的 Flannel 資訊清單:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

您應該修改的兩個區段, 以啟用 vxlan 網路後端:

1. 在您`net-conf.json` `kube-flannel.yml`的 [] 區段中, 仔細檢查:
 * 群集子網 (例如 "10.244.0.0/16") 是視需要設定。
 * VNI 的4096是在後端設定
 * 在後端中設定埠4789
2. 在您`cni-conf.json` `kube-flannel.yml`的 [] 區段中, 將 [網路`"vxlan0"`名稱] 變更為。

套用上述步驟之後, 您`net-conf.json`應該看起來如下:
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
> VNI 必須設定為 4096, 以及在 Linux 上 Flannel 的埠 4789, 才能與 Windows 上的 Flannel 互動。 即將推出其他 VNIs 的支援。 如需這些欄位的說明, 請參閱[VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) 。

您`cni-conf.json`應該看起來如下:
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
> 如需上述選項的詳細資訊, 請參閱適用于 Linux 的官方 CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)、 [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)及[bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference)外掛程式文檔。

### <a name="launch-flannel--validate"></a>啟動 Flannel & 驗證 ###
啟動 Flannel 使用:

```bash
kubectl apply -f kube-flannel.yml
```

接下來, 因為 Flannel 盒是 Linux, 所以請將 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修補程式套用`kube-flannel-ds`至 DaemonSet, 僅限目標 Linux (我們稍後會在 Windows 上啟動 Flannel "flanneld" 主機-代理程式處理):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果有任何節點不是 x86 基礎, 請`-amd64`以您的處理器架構取代。

幾分鐘之後, 如果已部署 Flannel pod 網路, 就會看到所有的箱都在執行中。

```bash
kubectl get pods --all-namespaces
```

![文字](media/kube-master.png)

Flannel DaemonSet 也應該套用 NodeSelector `beta.kubernetes.io/os=linux` 。

```bash
kubectl get ds -n kube-system
```

![文字](media/kube-daemonset.png)

> [!tip]  
> 對於其餘的 flannel-ds-* DaemonSets, 這些可以忽略或刪除, 因為如果沒有節點符合該處理器架構, 就不會排程這些專案。

> [!tip]  
> 混淆? 以下是適用于 Flannel v 0.11.0 的完整[範例 kube-flannel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) , 並針對預設的群集子網`10.244.0.0/16`預先套用這些步驟。

成功後, 請繼續執行[下一個步驟](#next-steps)。

## <a name="flannel-in-host-gateway-mode"></a>Flannel 中的 [主機-閘道模式]

除了[Flannel vxlan](#flannel-in-vxlan-mode)之外, 其他 Flannel 網路選項是*主機閘道模式*(主機-gw), 它必須使用目標節點的主機位址做為下一個躍點, 以將每個節點的靜態路由程式設計為其他節點的 pod 子網。

### <a name="prepare-kubernetes-master-for-flannel"></a>為 Flannel 準備 Kubernetes 主版

我們建議在群集中的[Kubernetes 主機](./creating-a-linux-master.md)上進行一些次要準備。 使用 Flannel 時, 建議您啟用橋接的 IPv4 流量至 iptables 鏈。 您可以使用下列命令完成這項作業:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>下載 & 設定 Flannel ###
下載最新的 Flannel 資訊清單:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

若要在兩個 Windows/Linux 啟用主機 gw 網路, 您需要變更一個檔案。

在 kube-flannel `net-conf.json`的 [] 區段中, 仔細檢查:
1. 所使用的網路後端類型是 [設定`host-gw`為] `vxlan`, 而不是。
2. 群集子網 (例如 "10.244.0.0/16") 是視需要設定。

套用2個步驟之後, 您`net-conf.json`應該看起來如下:
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
啟動 Flannel 使用:

```bash
kubectl apply -f kube-flannel.yml
```

接著, 因為 Flannel 盒是 Linux, 所以請將我們的 Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml)修補程式`kube-flannel-ds`套用至 DaemonSet, 只使用 linux (我們稍後會在 Windows 上啟動 Flannel "flanneld" 主機-代理程式處理):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> 如果有任何節點不是以 x86 為基礎`-amd64` , 請以所需的處理器架構取代。

幾分鐘之後, 如果已部署 Flannel pod 網路, 就會看到所有的箱都在執行中。

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
> 對於其餘的 flannel-ds-* DaemonSets, 這些可以忽略或刪除, 因為如果沒有節點符合該處理器架構, 就不會排程這些專案。

> [!tip]  
> 混淆? 以下是適用于 Flannel v 0.11.0 的完整[範例 kube-flannel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) , 並針對預設的群集子網`10.244.0.0/16`預先套用了下列2個步驟。

成功後, 請繼續執行[下一個步驟](#next-steps)。

## <a name="configuring-a-tor-switch"></a>設定 ToR 開關 ##
> [!NOTE]
> 如果您選擇 [ [Flannel] 做為網路解決方案](#flannel-in-host-gateway-mode), 您可以略過本節。
ToR 切換的設定會出現在您的實際節點以外。 如需更多相關資訊, 請參閱[正式 Kubernetes](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology)檔。


## <a name="next-steps"></a>後續步驟 ## 
在本節中, 我們將說明如何挑選及設定網路化解決方案。 現在您已準備好進行步驟 4:

> [!div class="nextstepaction"]
> [加入 Windows 工作人員](./joining-windows-workers.md)