---
title: 疑難排解 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: 部署 Kubernetes 和加入 Windows 節點時常見問題的解決方案。
keywords: kubernetes，1.14，linux，編譯
ms.openlocfilehash: fbb5b8474323a7d418de972bffbb9e005c94cb85
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622993"
---
# <a name="troubleshooting-kubernetes"></a>疑難排解 Kubernetes #
此頁面逐步解說 Kubernetes 設定、網路及部署的數個常見問題。

> [!tip]
> 藉由提出 PR 到[我們的文件存放庫](https://github.com/MicrosoftDocs/Virtualization-Documentation/)，建議常見問題集項目。

此頁面分成下列類別：
1. [一般問題](#general-questions)
2. [常見的網路錯誤](#common-networking-errors)
3. [常見 Windows 錯誤](#common-windows-errors)
4. [常見的 Kubernetes 主機錯誤](#common-kubernetes-master-errors)

## <a name="general-questions"></a>一般問題 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>如何了解 Windows 順利完成上的 start.ps1？ ###
您應該會看到 kubelet，kube proxy，和 （如果您選擇 Flannel 做為您的網路解決方案） flanneld 主機代理程式處理程序執行記錄檔中顯示您的節點上執行分隔漂亮 windows。 此外，您的 Windows 節點應該會列在 「 已準備好 」 在您的 Kubernetes 叢集。

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>我是否可以設定而不是漂亮 windows 在背景執行這一切？ ###
開始使用 Kubernetes 版本 1.11，可以做為原生[Windows 服務](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)執行 kubelet & kube proxy。 您也都可以使用替代的服務管理員像[nssm.exe](https://nssm.cc/)一律在背景中執行這些處理程序 （flanneld，讓 kubelet & kube proxy），為您。 [Kubernetes 上的 Windows 服務](./kube-windows-services.md)，例如步驟，請參閱。

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>我已作為 Windows 服務執行 Kubernetes 處理程序的問題 ###
初始疑難排解時，您可以使用下列旗標[nssm.exe](https://nssm.cc/)中，將 stdout 和 stderr 重新導向至輸出檔案：
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
如需詳細資訊，請參閱[nssm 使用量](https://nssm.cc/usage)的官方文件。

## <a name="common-networking-errors"></a>常見的網路錯誤 ##

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>我在這類看到錯誤 「 hnsCall 在 Win32 中失敗： 錯誤磁碟是磁碟機中。 」 ###
製作 HNS 物件或安裝新的 Windows Update 會造成不需要向下舊的 HNS 物件撕裂變更 HNS 的自訂修改時，會發生這個錯誤。 它會指出更新前先前已建立一個 HNS 物件是與目前安裝的 HNS 版本不相容。

在 Windows Server 2019 （以及下方），使用者可以刪除 HNS 物件刪除 HNS.data 檔案 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

使用者應該能夠直接刪除任何不相容的 HNS 端點或網路：
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Windows Server 上的使用者，版本 1903年可以移至下列登錄位置並刪除任何 Nic 從開始的網路名稱 (例如`vxlan0`或`cbr0`):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```


### <a name="my-windows-pods-cannot-ping-external-resources"></a>我的 Windows pod 無法 ping 外部資源 ###
Windows pod 沒有現今程式設計 ICMP 通訊協定的輸出規則。 但是，TCP/UDP 則支援。 當嘗試示範連線至叢集以外的資源，請取代`ping <IP>`使用對應的`curl <IP>`命令。

如果您仍然會遇到問題，很可能是您的網路設定中[cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)是值得一些額外的注意。 您可以隨時編輯此靜態的檔案，設定將會套用到新建立的任何 Kubernetes 資源。

為什麼？
Kubernetes 網路功能需求的其中一個 （請參閱[Kubernetes 模型](https://kubernetes.io/docs/concepts/cluster-administration/networking/)） 是在內部 NAT 的情況下發生的叢集通訊。 若要接受此需求，我們有適用於所有通訊[ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)其中我們不希望發生輸出 NAT。 不過，這也表示您需要將排除您嘗試從 ExceptionList 查詢的外部 IP。 然後只來自您的 Windows pod 的流量會 SNAT'ed 正確地從外界接收回應。 在這個問題，您在 ExceptionList`cni.conf`應該看起來如下：
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>我的 Windows 節點無法存取 NodePort 服務 ###
本機 NodePort 存取從其本身的節點將會失敗。 這是已知限制。 NodePort 存取將會從其他節點或外部的用戶端工作。

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>一些時間之後, vnic 與和容器既有的 HNS 端點會被刪除 ###
此問題可能因為當`hostname-override`參數不會傳遞至[kube proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)。 若要解決它，使用者必須將主機名稱傳遞至 kube proxy，如下所示：
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>Flannel (vxlan) 模式，我的 pod 之後重新加入節點有連線問題 ###
正在重新先前已刪除的節點加入叢集，每當 flannelD 會嘗試將新的 pod 子網路指派至節點。 使用者應該在下列路徑中移除舊的 pod 子網路設定檔：
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>在啟動 start.ps1 之後, Flanneld 停滯在同 」 等待 to be created 網路 」 ###
有許多此問題報告它所要調查;最有可能是為計時問題當管理 flannel 網路的 IP 設定。 因應措施是只要重新啟動 start.ps1 或重新啟動它以手動方式，如下所示：
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

也是目前可解決此問題，檢閱下方[PR](https://github.com/coreos/flannel/pull/1042) 。


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>Flannel (主機 gw)，在我的 Windows pod 沒有網路連線 ###
如果您想要使用 l2bridge 網路功能 (亦即[flannel 主機閘道](./network-topologies.md#flannel-in-host-gateway-mode))，您應該確定 Windows 容器主機 Vm （客體） 啟用改變 MAC 位址。 為了達成此目的，您應該執行下列命令以系統管理員身分裝載 Vm （hyper-v 所提供的範例） 在電腦上：

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> 如果您使用 VMware 為基礎的產品以符合您的虛擬化需求，請看起來到啟用 MAC 詐騙需求[混雜模式](https://kb.vmware.com/s/article/1004099)。

>[!TIP]
> 如果您正在部署 Azure 或 IaaS 虛擬機器上的 Kubernetes 從其他雲端提供者自行，您也可以使用[覆疊網路功能](./network-topologies.md#flannel-in-vxlan-mode)改為。

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>我的 Windows pod 無法啟動因為缺少 /run/flannel/subnet.env ###
這表示 Flannel 無法正確啟動。 您可以嘗試重新啟動 flanneld.exe 或您可以將檔案複製手動從`/run/flannel/subnet.env`若要建立 Kubernetes 主機上`C:\run\flannel\subnet.env`Windows 背景工作節點上和修改`FLANNEL_SUBNET`已指派的子網路的資料列。 例如，如果節點的子網路 10.244.4.1/24 已指派：
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
它是更安全，讓 flanneld.exe 為您產生此檔案。

### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>在中斷 vSphere 上執行我 Kubernetes 叢集上的 pod 對 pod 主機之間的連線能力 
VSphere 和 Flannel 都保留用於覆疊網路連接埠 4789 （預設 VXLAN 連接埠），因為可以結束被攔截的封包。 如果 vSphere 用於覆疊網路功能，您應該設定以釋出 4789 使用不同的連接埠。  


### <a name="my-endpointsips-are-leaking"></a>我的端點/Ip 流失 ###
有 2 個目前已知的問題可能會造成遺漏的端點。 
1.  [已知問題](https://github.com/kubernetes/kubernetes/issues/68511)的第一個是 Kubernetes 版本 1.11 中的問題。 請避免使用 Kubernetes 版本 1.11.0-1.11.2。
2. 第二個[已知問題](https://github.com/docker/libnetwork/issues/1950)，可能會造成遺漏的端點是在端點的儲存體的並行問題。 若要接收已修正問題，您必須使用 Docker EE 18.09 或更新版本。

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>我的 pod 無法啟動由於 」 網路： 為範圍配置失敗 」 錯誤 ###
這表示您的節點上的 IP 位址空間耗盡。 若要清除任何[遺漏端點](#my-endpointsips-are-leaking)，請移轉受影響的程度節點 & 執行下列命令中的任何資源：
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 節點無法使用服務 IP 存取我的服務 ###
這是 Windows 目前網路堆疊的已知限制。 Windows *pod* **都**能不過存取服務 IP。

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>啟動 Kubelet 時找不到網路介面卡 ###
Windows 網路堆疊需要虛擬介面卡，才能讓 Kubernetes 網路功能運作。 如果下列命令未傳回任何結果 (在 Admin Shell 中)，表示虛擬網路建立作業 &mdash; 讓 Kubelet 運作所需的必要條件 &mdash; 失敗：

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

通常是值得修改 start.ps1 指令碼，在其中的主機網路介面卡不是"Ethernet"的情況下的[預設](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6)參數。 否則，請參閱的輸出`start-kubelet.ps1`指令碼，以查看虛擬網路建立期間是否有錯誤。 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pod 在持續運作一段時間後順利停止解析 DNS 查詢 ###
已知的 DNS 快取中的 Windows Server 網路堆疊的問題，版本 1803年或下方，有時可能會造成 DNS 要求失敗。 若要解決此問題，您可以設定為零使用下列登錄機碼的最大 TTL 快取值：

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>我仍然看到問題。 我該怎麼辦？ ### 
網路上或主機上可能有其他限制，防止節點之間特定類型的通訊。 請確定：
  - 您已正確設定您選擇的[網路拓撲](./network-topologies.md)
  - 允許來自 Pod 的流量
  - 允許 HTTP 流量，如果您要部署 Web 服務
  - 從不同的通訊協定 (ie ICMP 與 TCP/UDP) 的封包都未捨棄


## <a name="common-windows-errors"></a>常見 Windows 錯誤 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>我的 Kubernetes Pod 停滯在「ContainerCreating」 ###
此問題可能有很多成因，但其中一個最常見的原因是 Pause 映像設定錯誤。 這是下一個問題的高度徵兆。


### <a name="when-deploying-docker-containers-keep-restarting"></a>進行部署時，請 Docker 容器不斷重新啟動 ###
確認 Pause 影像與您的作業系統版本相容。 [指示](./deploying-resources.md)假設作業系統與容器是版本 1803年。 如果您的 Windows 是較新的版本 (例如測試人員組建)，就必須相應調整該映像。 如需了解映像，請參閱 Microsoft 的 [Docker 儲存機制](https://hub.docker.com/u/microsoft/)。 什麼都別管，Pause 映像 Dockerfile 和範例服務反正都預期此映像會標記為 `:latest`。


## <a name="common-kubernetes-master-errors"></a>常見的 Kubernetes 主機錯誤 ##
偵錯 Kubernetes 主機可分成三大類 (依可能性)：

  - Kubernetes 系統容器有問題。
  - `kubelet` 執行方式有問題。
  - 系統有問題。

執行 `kubectl get pods -n kube-system` 以查看 Kubernetes 建立的 Pod；這可能會提供哪些發生當機或無法正確啟動的深入解析。 然後執行 `docker ps -a` 以查看所有支援這些 Pod 的原始容器。 最後，在懷疑可能造成問題上的容器執行 `docker logs [ID]`，以查看處理序的原始輸出。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>無法連線至位於 `https://[address]:[port]` 的 API 伺服器 ###
這個錯誤通常表示憑證問題。 請確定您已正確產生組態檔，其中的 IP 位址符合您的主機 IP 位址，而且您已將它複製到 API 伺服器裝載的目錄。

如果遵循[我們的指示](./creating-a-linux-master.md)，好放入來尋找這是：   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 否則，請參閱 API 伺服器的資訊清單檔案，以檢查裝載點。