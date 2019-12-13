---
title: 以 Windows 服務的形式執行 Kubernetes
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: 如何以 Windows 服務的形式執行 Kubernetes 元件。
keywords: kubernetes，1.14，windows，使用者入門
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: cd5026a244b57b5c70d4abfe076839130315a4f5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909798"
---
# <a name="kubernetes-components-as-windows-services"></a>以 Windows 服務的形式 Kubernetes 元件 

某些使用者可能會想要設定像是 flanneld、kubelet、kube-proxy 或其他進程，以 Windows 服務的形式執行。 這會帶來額外的容錯優勢，例如在非預期的進程或節點損毀時自動重新開機進程。


## <a name="prerequisites"></a>必要條件
1. 您已將[nssm](https://nssm.cc/download)下載至 `c:\k` 目錄
2. 您已將節點加入叢集，並在您的節點上執行[install. ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1)或[start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)腳本

## <a name="registering-windows-services"></a>註冊 Windows 服務
您可以執行[範例腳本](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1)，它會使用 nssm 來註冊 `kubelet`、`kube-proxy`和 `flanneld.exe` 在背景中以 Windows 服務的形式執行：

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementiptabmanagementip"></a>[ManagementIP](#tab/ManagementIP)
指派給 Windows 節點的 IP 位址。 您可以使用 `ipconfig` 來尋找此。

|  |  | 
|---------|---------|
|參數     | `-ManagementIP`        |
|預設值    | 馬丁        |


# <a name="networkmodetabnetworkmode"></a>[NetworkMode](#tab/NetworkMode)
選擇做為[網路解決方案](./network-topologies.md)的網路模式 `l2bridge` （flannel 主機-gw）或 `overlay` （flannel vxlan）。

> [!Important] 
> `overlay` 網路模式（flannel vxlan）需要 Kubernetes v 1.14 二進位檔或更新版本。

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


# <a name="kubednsserviceiptabkubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[KUBERNETES DNS 服務 IP](./getting-started-kubernetes-windows.md#kube-dns-def)。

|  |  | 
|---------|---------|
|參數     | `-KubeDnsServiceIP`        |
|預設值    | `10.96.0.10`        |


# <a name="logdirtablogdir"></a>[LogDir](#tab/LogDir)
Kubelet 和 kube proxy 記錄會重新導向至其個別輸出檔案的目錄。

|  |  | 
|---------|---------|
|參數     | `-LogDir`        |
|預設值    | `C:\k`        |

---


> [!TIP] 
> 萬一發生錯誤，請參閱[疑難排解一節](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>手動方法
如果[上述的參考腳本](#registering-windows-services)無法供您使用，本節會提供一些*範例命令*，可供您以手動方式逐步註冊這些服務。

> [!TIP] 
> 如需如何設定 `kubelet` 和 `kube-proxy` 以透過 `sc`以原生 Windows 服務的方式執行的詳細資訊，請參閱[Kubelet 和 kube](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) 。

### <a name="register-flanneldexe"></a>註冊 flanneld .exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>註冊 kubelet .exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>註冊 kube-proxy （l2bridge/主機-gw）
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>註冊 kube-proxy （重迭/vxlan）
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```