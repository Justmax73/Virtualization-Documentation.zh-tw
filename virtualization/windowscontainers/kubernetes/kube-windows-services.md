---
title: 為 Windows 服務執行 Kubernetes
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: 如何為 Windows 服務執行 Kubernetes 元件。
keywords: kubernetes，1.13，windows，開始使用
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 6c68edda6e2017640b0a490c3c30f063c81698b3
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120587"
---
# <a name="kubernetes-components-as-windows-services"></a>Kubernetes 元件做為 Windows 服務 

某些使用者可能會想要設定處理程序，例如 flanneld.exe、 kubelet.exe、 kube proxy.exe 或其他 Windows 服務身分執行。 這可以為帶來額外的容錯能力權益，例如自動重新啟動時未預期的處理程序或節點損毀的程序。


## <a name="prerequisites"></a>必要條件
1. 您已下載到[nssm.exe](https://nssm.cc/download) `c:\k`目錄
2. 您已經加入您的叢集節點並先前在您的節點上執行[install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1)或[start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1)指令碼

## <a name="registering-windows-services"></a>註冊 Windows 服務
您可以執行[的範例指令碼](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1)，將會註冊名哪些使用 nssm.exe `kubelet`， `kube-proxy`，並`flanneld.exe`做為 Windows 服務，在背景中執行：

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
IP 位址指派給 Windows 節點。 您可以使用`ipconfig`若要尋找這。

|  |  | 
|---------|---------|
|參數     | `-ManagementIP`        |
|預設值    | n.A.        |


# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
網路模式`l2bridge`(flannel 主機 gw) 或`overlay`(flannel vxlan) 做為[網路解決方案](./network-topologies.md)選擇。

> [!Important] 
> `overlay` 網路功能模式 (flannel vxlan) 需要 Kubernetes v1.14 二進位檔案或更新版本。

|  |  | 
|---------|---------|
|參數     | `-NetworkMode`        |
|預設值    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
[叢集子網路範圍](./getting-started-kubernetes-windows.md#cluster-subnet-def)。

|  |  | 
|---------|---------|
|參數     | `-ClusterCIDR`        |
|預設值    | `10.244.0.0/16`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Kubernetes DNS 服務 IP](./getting-started-kubernetes-windows.md#kube-dns-def)。

|  |  | 
|---------|---------|
|參數     | `-KubeDnsServiceIP`        |
|預設值    | `10.96.0.10`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
其中 kubelet 和 kube proxy 記錄檔會被重新導向至其各自的輸出檔目錄。

|  |  | 
|---------|---------|
|參數     | `-LogDir`        |
|預設值    | `C:\k`        |

---


> [!TIP] 
> 應該出現錯誤，請洽詢[疑難排解區段](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>手動方法
應該[上方參照的指令碼](#registering-windows-services)不為此區段，您的工作提供一些*範例命令*，可用來登錄手動逐步說明這些服務。

> [!TIP] 
> 如需有關如何設定的詳細資訊請參閱[Kubelet 和 kube proxy 現在可以執行做為 Windows 服務](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)`kubelet`和`kube-proxy`做為原生 Windows 服務，提供執行`sc`。

### <a name="register-flanneldexe"></a>註冊 flanneld.exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>註冊 kubelet.exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>註冊 kube proxy.exe (l2bridge / 主機 gw)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>註冊 kube proxy.exe (重疊 / vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```