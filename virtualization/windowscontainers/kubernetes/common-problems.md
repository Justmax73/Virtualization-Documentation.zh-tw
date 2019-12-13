---
title: 疑難排解 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: 部署 Kubernetes 和加入 Windows 節點時常見問題的解決方案。
keywords: kubernetes，1.14，linux，編譯
ms.openlocfilehash: 471731ec50c7c03816a956bd7aae859ad218be6d
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910448"
---
# <a name="troubleshooting-kubernetes"></a>疑難排解 Kubernetes #
此頁面逐步解說 Kubernetes 設定、網路及部署的數個常見問題。

> [!tip]
> 藉由提出 PR 到[我們的文件存放庫](https://github.com/MicrosoftDocs/Virtualization-Documentation/)，建議常見問題集項目。

此頁面會細分成下列類別：
1. [一般問題](#general-questions)
2. [常見的網路錯誤](#common-networking-errors)
3. [常見的 Windows 錯誤](#common-windows-errors)
4. [常見的 Kubernetes 主要錯誤](#common-kubernetes-master-errors)

## <a name="general-questions"></a>一般問題 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>如何? 知道，Windows 上的 ps1 成功完成了嗎？ ###
您應該會看到 kubelet、kube-proxy 和（如果您選擇 Flannel 做為您的網路解決方案） flanneld 在節點上執行的主機代理程式進程，並在個別的 PoSh 視窗中顯示執行中的記錄檔。 此外，您的 Windows 節點在 Kubernetes 叢集中應列為「就緒」。

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>我可以設定在背景中執行所有這項作業，而不是 PoSh 視窗嗎？ ###
從 Kubernetes 版本1.11 開始，kubelet & kube-proxy 可以當做原生[Windows 服務](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)執行。 您也可以一律使用替代服務管理員（例如[nssm](https://nssm.cc/) ），在背景中一律為您執行這些進程（flanneld、kubelet & kube-proxy）。 如需範例步驟，請參閱[Kubernetes 上的 Windows 服務](./kube-windows-services.md)。

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>我在執行 Kubernetes 進程做為 Windows 服務時遇到問題 ###
若要進行初始疑難排解，您可以在[nssm](https://nssm.cc/)中使用下列旗標，將 stdout 和 stderr 重新導向至輸出檔案：
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
如需其他詳細資訊，請參閱官方[nssm 使用量](https://nssm.cc/usage)檔。

## <a name="common-networking-errors"></a>常見的網路錯誤 ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>跨叢集節點的負載平衡器會以不一致的情況進行檢測 ###
在 Windows 上，kube 會針對叢集中的每個 Kubernetes 服務建立 HNS 負載平衡器。 在（預設值） kube-proxy 設定中，叢集中包含多個（通常是 100 +）負載平衡器的節點可能會用盡可用的暫時 TCP 埠（也稱為 動態埠範圍，預設會涵蓋埠49152到65535）。 這是因為每個節點上為每個（非 DSR）負載平衡器保留的埠數目過多。 此問題可能會透過 kube proxy 中的錯誤，例如：
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

使用者可以藉由執行[CollectLogs](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1)腳本並查閱 `*portrange.txt` 的檔案來識別此問題。

`CollectLogs.ps1` 也會模擬 HNS 配置邏輯，以在暫時的 TCP 埠範圍中測試埠集區配置可用性，並在 `reservedports.txt`中報告成功/失敗。 腳本會保留10個 64 TCP 暫時埠的範圍（以模擬 HNS 行為）、計算保留成功 & 失敗，然後釋放已配置的埠範圍。 小於10的成功數位表示暫時集區的可用空間不足。 `reservedports.txt`中也會產生大約有多少 64-區塊埠保留空間的 heuristical 摘要。

若要解決這個問題，您可以採取幾個步驟：
1.  若為永久解決方案，kube 的 proxy 負載平衡應設定為[DSR 模式](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710)。 DSR 模式完全在較新的[Windows Server Insider build 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) （或更高版本）上執行，並可供使用。
2. 因應措施是，使用者也可以使用命令（例如 `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>`）來增加可用的暫時埠預設 Windows 設定。 *警告：* 覆寫預設的動態埠範圍，可能會對主機上其他進程/服務產生依賴非暫時範圍的可用 TCP 埠的結果，因此應謹慎選取此範圍。
3. 非 DSR 模式的負載平衡器使用智慧型通訊埠集區共用進行了擴充性增強，其排程是透過 Q1 2020 中的累積更新來發行。

### <a name="hostport-publishing-is-not-working"></a>HostPort 發佈無法運作 ###
目前無法使用 [Kubernetes `containers.ports.hostPort`] 欄位發佈通訊埠，因為 Windows CNI 外掛程式並不接受此欄位。 請使用 NodePort 發佈來發行節點上的埠。

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>我看到錯誤，例如「Win32 中的 hnsCall 失敗：磁片磁碟機中有錯誤的磁片。」 ###
對 HNS 物件進行自訂修改，或安裝新的 Windows Update （在不卸載舊的 HNS 物件的情況下）時，就會發生此錯誤。 這表示在更新與目前安裝的 HNS 版本不相容之前，先前已建立的 HNS 物件。

在 Windows Server 2019 （和以下）中，使用者可以藉由刪除 HNS 資料檔來刪除 HNS 物件。 
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

Windows Server 1903 版的使用者可以前往下列登錄位置，並刪除以網路名稱（例如 `vxlan0` 或 `cbr0`）開頭的任何 Nic：
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>我的 Flannel 主機上的容器-Azure 上的 gw 部署無法連線到網際網路 ###
在 Azure 上的主機-gw 模式中部署 Flannel 時，封包必須經過 Azure 實體主機 vSwitch。 使用者應該為指派給節點的每個子網，設計「虛擬應用裝置」類型的[使用者定義路由](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined)。 這可以透過 Azure 入口網站來完成（請參閱[這裡](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)的範例），或經由 `az` Azure CLI。 以下是一個名為 "MyRoute" 的範例 UDR，其使用具有 IP 10.0.0.4 和個別 pod 子網 10.244.0.0/24 的節點的 az 命令：
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> 如果您要自行在 Azure 或來自其他雲端提供者的 IaaS Vm 上部署 Kubernetes，您也可以改為使用重迭[網路](./network-topologies.md#flannel-in-vxlan-mode)。

### <a name="my-windows-pods-cannot-ping-external-resources"></a>我的 Windows pod 無法 ping 外部資源 ###
Windows pod 目前沒有針對 ICMP 通訊協定進行程式設計的輸出規則。 不過，支援 TCP/UDP。 嘗試展示叢集外部資源的連線能力時，請以對應的 `curl <IP>` 命令取代 `ping <IP>`。

如果您仍面臨問題，很可能您[cni](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)中的網路設定應該會有一些額外的注意。 您隨時都可以編輯此靜態檔案，設定將會套用至任何新建立的 Kubernetes 資源。

為什麼？
其中一個 Kubernetes 網路需求（請參閱[Kubernetes 模型](https://kubernetes.io/docs/concepts/cluster-administration/networking/)）是為了在內部沒有 NAT 的情況下進行叢集通訊。 為了遵守這項需求，我們對所有不會出現輸出 NAT 的通訊都有一個[例外](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)狀況。 不過，這也表示您必須排除您嘗試從例外情況查詢的外部 IP。 只有這樣才會正確 SNAT'ed 來自 Windows pod 的流量，以接收來自外界的回應。 就這一點而言，您在 `cni.conf` 中的例外，看起來應該如下所示：
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>我的 Windows 節點無法存取 NodePort 服務 ###
來自節點本身的本機 NodePort 存取將會失敗。 此為已知的限制狀況。 NodePort 存取會從其他節點或外部用戶端工作。

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>一段時間之後，會刪除容器的 Vnic 和 HNS 端點 ###
當 `hostname-override` 參數未傳遞至[kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)時，可能會造成此問題。 若要解決此問題，使用者必須將主機名稱傳遞至 kube-proxy，如下所示：
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>在 Flannel （vxlan）模式中，我的 pod 在重新加入節點後發生連線問題 ###
當先前刪除的節點重新加入叢集時，flannelD 會嘗試將新的 pod 子網指派給該節點。 使用者應該移除下列路徑中的舊 pod 子網設定檔：
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>啟動 start. ps1 之後，Flanneld 會停滯在「正在等候網路建立」 ###
此問題有許多報告正在進行調查;這很可能是設定 flannel 網路管理 IP 時的時間問題。 因應措施是直接重新開機，或手動重新開機它，如下所示：
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

目前還有一個[PR](https://github.com/coreos/flannel/pull/1042)可解決此問題。


### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>我的 Windows pod 因遺失/run/flannel/subnet.env 而無法啟動 ###
這表示 Flannel 未正確啟動。 您可以嘗試重新開機 flanneld，也可以手動將檔案從 Kubernetes 主機上的 `/run/flannel/subnet.env` 複製到 Windows 背景工作節點上的 `C:\run\flannel\subnet.env`，然後將 `FLANNEL_SUBNET` 資料列修改為指派的子網。 例如，如果已指派 node 子網 10.244.4.1/24：
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
讓 flanneld 為您產生此檔案更為安全。


### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>在 vSphere 上執行的 Kubernetes 叢集上，主機之間的 pod 對 pod 連線中斷 
由於 vSphere 和 Flannel 都會保留重迭網路的埠4789（預設 VXLAN 埠），因此封包最後可能會被攔截。 如果 vSphere 用於重迭網路，則應該將它設定為使用不同的埠，才能釋出4789。  


### <a name="my-endpointsips-are-leaking"></a>我的端點/Ip 洩漏 ###
目前有2個已知問題，可能會造成端點洩漏。 
1.  第一個[已知問題](https://github.com/kubernetes/kubernetes/issues/68511)是 Kubernetes 1.11 版中的問題。 請避免使用 Kubernetes 版本 1.11.0-1.11.2。
2. 第二個會導致端點流失的[已知問題](https://github.com/docker/libnetwork/issues/1950)，是端點儲存體中的並行問題。 若要接收修正，您必須使用 Docker EE 18.09 或更新版本。

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>因為「網路：無法配置範圍」錯誤，所以無法啟動我的 pod ###
這表示節點上的 IP 位址空間已用完。 若要清除任何[遺漏的端點](#my-endpointsips-are-leaking)，請在受影響的節點上遷移任何資源，& 執行下列命令：
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 節點無法使用服務 IP 存取我的服務 ###
這是 Windows 目前網路堆疊的已知限制。 不過 *，Windows pod 可以存取*服務 IP。

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>啟動 Kubelet 時找不到網路介面卡 ###
Windows 網路堆疊需要虛擬介面卡，才能讓 Kubernetes 網路功能運作。 如果下列命令未傳回任何結果 (在 Admin Shell 中)，表示虛擬網路建立作業 &mdash; 讓 Kubelet 運作所需的必要條件 &mdash; 失敗：

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

通常，在主機的網路介面卡不是「Ethernet」的情況下，修改 start. ps1 腳本的[介面名稱](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6)參數是很值得的。 否則，請參閱 `start-kubelet.ps1` 腳本的輸出，以瞭解在建立虛擬網路期間是否發生錯誤。 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pod 在持續運作一段時間後順利停止解析 DNS 查詢 ###
Windows Server 1803 版和更低版本的網路堆疊中有已知的 DNS 快取問題，有時可能會導致 DNS 要求失敗。 若要解決此問題，您可以使用下列登錄機碼，將最大 TTL 快取值設定為零：

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>我仍然發現問題。 我該怎麼做？ ### 
網路上或主機上可能有其他限制，防止節點之間特定類型的通訊。 請確定：
  - 您已正確設定您選擇的[網路拓撲](./network-topologies.md)
  - 允許來自 Pod 的流量
  - 允許 HTTP 流量，如果您要部署 Web 服務
  - 不會卸載來自不同通訊協定（ie ICMP 與 TCP/UDP）的封包

>[!TIP]
> 如需其他自助資源，請參閱[這裡的適用](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)于 Windows 的 Kubernetes 疑難排解指南。

## <a name="common-windows-errors"></a>常見的 Windows 錯誤 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>我的 Kubernetes Pod 停滯在「ContainerCreating」 ###
此問題可能有很多成因，但其中一個最常見的原因是 Pause 映像設定錯誤。 這是下一個問題的高度徵兆。


### <a name="when-deploying-docker-containers-keep-restarting"></a>進行部署時，請 Docker 容器不斷重新啟動 ###
確認 Pause 影像與您的作業系統版本相容。 這些[指示](./deploying-resources.md)假設 OS 和容器都是1803版。 如果您的 Windows 是較新的版本 (例如測試人員組建)，就必須相應調整該映像。 如需了解映像，請參閱 Microsoft 的 [Docker 儲存機制](https://hub.docker.com/u/microsoft/)。 什麼都別管，Pause 映像 Dockerfile 和範例服務反正都預期此映像會標記為 `:latest`。


## <a name="common-kubernetes-master-errors"></a>常見的 Kubernetes 主要錯誤 ##
偵錯 Kubernetes 主機可分成三大類 (依可能性)：

  - Kubernetes 系統容器有問題。
  - `kubelet` 執行方式有問題。
  - 系統有問題。

執行 `kubectl get pods -n kube-system` 以查看 Kubernetes 建立的 Pod；這可能會提供哪些發生當機或無法正確啟動的深入解析。 然後執行 `docker ps -a` 以查看所有支援這些 Pod 的原始容器。 最後，在懷疑可能造成問題上的容器執行 `docker logs [ID]`，以查看處理序的原始輸出。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>無法連線至位於 `https://[address]:[port]` 的 API 伺服器 ###
這個錯誤通常表示憑證問題。 請確定您已正確產生組態檔，其中的 IP 位址符合您的主機 IP 位址，而且您已將它複製到 API 伺服器裝載的目錄。

如果遵循[我們的指示](./creating-a-linux-master.md)，很好的地方可以找到：   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 否則，請參閱 API 伺服器的資訊清單檔案，以檢查掛接點。
