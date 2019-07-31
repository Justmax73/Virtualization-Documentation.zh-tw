---
title: 疑難排解 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: 部署 Kubernetes 和加入 Windows 節點時常見問題的解決方案。
keywords: kubernetes、1.14、linux、compile
ms.openlocfilehash: bdf1fd78bbbebcad3562872d9e71c961be6c64eb
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883001"
---
# <a name="troubleshooting-kubernetes"></a>疑難排解 Kubernetes #
此頁面逐步解說 Kubernetes 設定、網路及部署的數個常見問題。

> [!tip]
> 藉由提出 PR 到[我們的文件存放庫](https://github.com/MicrosoftDocs/Virtualization-Documentation/)，建議常見問題集項目。

此頁面分為下列類別:
1. [一般問題](#general-questions)
2. [常見的網路錯誤](#common-networking-errors)
3. [常見的 Windows 錯誤](#common-windows-errors)
4. [常見的 Kubernetes 主要錯誤](#common-kubernetes-master-errors)

## <a name="general-questions"></a>一般問題 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>如何知道開始! Windows 上的 ps1 已順利完成？ ###
您應該會看到 kubelet、kube-proxy, 以及 (如果您選擇 Flannel 做為網路解決方案) flanneld 在您的節點上執行的主機代理程式進程, 且在不同的 PoSh 視窗中顯示正在執行的記錄。 此外, 您的 Windows 節點在您的 Kubernetes 群集中應該會列為 "就緒"。

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>我是否可以設定在背景中執行所有這些作業, 而不是 PoSh 視窗？ ###
從 Kubernetes 版本1.11 開始, kubelet & kube-proxy 可以作為原生[Windows 服務](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)執行。 您也可以隨時使用替代服務管理員 (例如[nssm](https://nssm.cc/) ), 在背景中一直執行這些程式 (flanneld、kubelet & kube-proxy)。 如需詳細步驟, 請參閱[Kubernetes 上的 Windows 服務](./kube-windows-services.md)。

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>我無法執行 Kubernetes 處理常式做為 Windows 服務的問題 ###
針對初始疑難排解, 您可以在[nssm](https://nssm.cc/)中使用下列標誌, 將 stdout 和 stderr 重新導向到輸出檔:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
如需其他詳細資料, 請參閱正式[nssm 使用](https://nssm.cc/usage)檔。

## <a name="common-networking-errors"></a>常見的網路錯誤 ##

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>我在 Win32 中看到「hnsCall 失敗」之類的錯誤: 磁片磁碟機中有錯誤的磁片。 ###
當您對 HNS 物件進行自訂修改或安裝新的 Windows 更新, 而不需要撕裂舊的 HNS 物件時, 就會發生此錯誤。 它表示先前在更新與目前安裝的 HNS 版本不相容之前所建立的 HNS 物件。

在 Windows Server 2019 (及以下) 上, 使用者可以透過刪除 HNS 資料檔來刪除 HNS 物件。 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

使用者應該能夠直接刪除任何不相容的 HNS 端點或網路:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Windows Server 上的使用者, 版本1903可以移至下列登錄位置, 並從網路名稱 (例如`vxlan0` `cbr0`) 刪除任何 nic:
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```


### <a name="my-windows-pods-cannot-ping-external-resources"></a>我的 Windows 盒無法 ping 外部資源 ###
Windows 盒目前沒有為 ICMP 通訊協定預先設定的輸出規則。 不過, 支援 TCP/UDP。 當您嘗試示範與群集以外的資源的連線時, 請`ping <IP>`使用對應`curl <IP>`的命令加以取代。

如果您仍面臨問題, 您在[cni](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)中很可能是您的網路設定值得格外注意。 您隨時可以編輯這個靜態檔案, 設定就會套用到任何新近建立的 Kubernetes 資源。

為什麼？
其中一個 Kubernetes 網路需求 (請參閱[Kubernetes 模型](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) 是在沒有 NAT 的情況下發生的群集通訊。 若要服從此需求, 我們會針對所有不想要輸出 NAT 發生的通訊, 提供[例外](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)順序。 不過, 這也表示您需要排除您嘗試從例外例外中查詢的外部 IP。 只有在您的 Windows 箱中產生的流量, 才能正確 SNAT'ed, 以接收來自外部世界的回應。 在這個方面, 您的例外`cni.conf`順序看起來應該如下:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>我的 Windows 節點無法存取 NodePort 服務 ###
從節點本身進行本機 NodePort 存取將會失敗。 這是已知限制。 NodePort access 將可從其他節點或外部用戶端運作。

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>一段時間之後, 會刪除容器的 vNICs 和 HNS 端點 ###
當參數未傳遞至`hostname-override` [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)時, 可能會造成此問題。 若要解決這個問題, 使用者需要將主機名稱傳給 kube-proxy, 如下所示:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>在 Flannel (vxlan) 模式下, 在 rejoining 節點後, 我的盒有連接問題 ###
只要將先前刪除的節點重新加入群集, flannelD 將會嘗試將新的 pod 子網指派給該節點。 使用者應該在下列路徑中移除舊的 pod 子網設定檔:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>啟動 start. ps1 之後, Flanneld 停滯在「正在等待建立網路」 ###
我們正在調查此問題的許多報告;最可能是設定 flannel 網路管理 IP 的時間問題。 因應措施是直接重新開機 start. ps1 或手動重新開機, 如下所示:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

在目前審查中, 還有一個可解決此問題的[PR](https://github.com/coreos/flannel/pull/1042) 。


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>在 Flannel (主機-gw) 上, 我的 Windows 盒沒有網路連線能力 ###
如果您想要使用 l2bridge 進行網路功能 (亦即[flannel 主機閘道](./network-topologies.md#flannel-in-host-gateway-mode)), 您應該確保已針對 Windows 容器主機 vm (來賓) 啟用 MAC 位址欺騙功能。 若要達到這個目的, 您應該在裝載 Vm 的電腦上以系統管理員身分執行下列動作 (例如, Hyper-v 提供的範例):

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> 如果您使用的是以 VMware 為基礎的產品來符合您的虛擬化需求, 請參閱針對 MAC 欺騙需求啟用[混雜模式](https://kb.vmware.com/s/article/1004099)。

>[!TIP]
> 如果您是在其他雲端提供者的 Azure 或 IaaS Vm 上自行部署 Kubernetes, 您也可以改為使用重迭[網路](./network-topologies.md#flannel-in-vxlan-mode)。

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>我的 Windows 盒無法啟動, 因為遺失/run/flannel/subnet.env ###
這表示 Flannel 未正確啟動。 您可以嘗試重新開機 flanneld, 也可以從`/run/flannel/subnet.env` Kubernetes 主版手動將檔案複製到`C:\run\flannel\subnet.env` Windows worker 節點上, 然後將該`FLANNEL_SUBNET`列修改為指派的子網。 例如, 如果已指派節點子網 10.244.4.1/24:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
更安全的做法是讓 flanneld 為您產生此檔案。

### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>主機間的 pod 到 pod 連線在 vSphere 上的 Kubernetes 群集中中斷 
因為 vSphere 和 Flannel 都是重迭網路的埠 4789 (預設 VXLAN 埠), 所以資料包最終會遭到截獲。 如果 vSphere 是用於重迭網路, 則應該將它設定為使用不同的埠才能釋放4789。  


### <a name="my-endpointsips-are-leaking"></a>我的端點/Ip 正在洩漏 ###
存在2個目前已知問題, 可能會導致端點洩漏。 
1.  第一個[已知問題](https://github.com/kubernetes/kubernetes/issues/68511)是 Kubernetes 版本1.11 中的問題。 請避免使用 Kubernetes 版本 1.11.0-1.11.2。
2. 可能導致端點洩漏的第二個[已知問題](https://github.com/docker/libnetwork/issues/1950)是端點儲存中的併發問題。 若要接收修正程式, 您必須使用 Docker EE 18.09 或更新版本。

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>我的箱無法啟動, 因為「網路: 無法為範圍指派」錯誤 ###
這表示您節點上的 IP 位址空間已用完。 若要清除任何[洩漏的端點](#my-endpointsips-are-leaking), 請在受影響的節點上遷移任何資源 & 執行下列命令:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 節點無法使用服務 IP 存取我的服務 ###
這是 Windows 目前網路堆疊的已知限制。 Windows*盒* **** 可以存取服務 IP。

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>啟動 Kubelet 時找不到網路介面卡 ###
Windows 網路堆疊需要虛擬介面卡，才能讓 Kubernetes 網路功能運作。 如果下列命令未傳回任何結果 (在 Admin Shell 中)，表示虛擬網路建立作業 &mdash; 讓 Kubelet 運作所需的必要條件 &mdash; 失敗：

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

通常, 修改 start. ps1 腳本的[InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6)參數 (如果主機的網路介面卡不是 "Ethernet") 是必要的。 否則, 請參閱`start-kubelet.ps1`腳本的輸出, 查看虛擬網路建立期間是否有錯誤。 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pod 在持續運作一段時間後順利停止解析 DNS 查詢 ###
在 Windows Server、版本1803和下方的網路堆疊中, 可能會發生已知的 DNS 快取問題, 有時可能會造成 DNS 要求失敗。 若要解決此問題, 您可以使用下列登錄機碼, 將最大 TTL 快取值設為零:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>我仍在看到問題。 我該怎麼做？ ### 
網路上或主機上可能有其他限制，防止節點之間特定類型的通訊。 請確定：
  - 您已正確設定您選擇的[網路拓朴](./network-topologies.md)
  - 允許來自 Pod 的流量
  - 允許 HTTP 流量，如果您要部署 Web 服務
  - 未刪除來自不同通訊協定 (ie ICMP 與 TCP/UDP) 的資料包


## <a name="common-windows-errors"></a>常見的 Windows 錯誤 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>我的 Kubernetes Pod 停滯在「ContainerCreating」 ###
此問題可能有很多成因，但其中一個最常見的原因是 Pause 映像設定錯誤。 這是下一個問題的高度徵兆。


### <a name="when-deploying-docker-containers-keep-restarting"></a>進行部署時，請 Docker 容器不斷重新啟動 ###
確認 Pause 影像與您的作業系統版本相容。 [指示](./deploying-resources.md)會假設 OS 和容器都是版本1803。 如果您的 Windows 是較新的版本 (例如測試人員組建)，就必須相應調整該映像。 如需了解映像，請參閱 Microsoft 的 [Docker 儲存機制](https://hub.docker.com/u/microsoft/)。 什麼都別管，Pause 映像 Dockerfile 和範例服務反正都預期此映像會標記為 `:latest`。


## <a name="common-kubernetes-master-errors"></a>常見的 Kubernetes 主要錯誤 ##
偵錯 Kubernetes 主機可分成三大類 (依可能性)：

  - Kubernetes 系統容器有問題。
  - `kubelet` 執行方式有問題。
  - 系統有問題。

執行 `kubectl get pods -n kube-system` 以查看 Kubernetes 建立的 Pod；這可能會提供哪些發生當機或無法正確啟動的深入解析。 然後執行 `docker ps -a` 以查看所有支援這些 Pod 的原始容器。 最後，在懷疑可能造成問題上的容器執行 `docker logs [ID]`，以查看處理序的原始輸出。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>無法連線至位於 `https://[address]:[port]` 的 API 伺服器 ###
這個錯誤通常表示憑證問題。 請確定您已正確產生組態檔，其中的 IP 位址符合您的主機 IP 位址，而且您已將它複製到 API 伺服器裝載的目錄。

如果遵循[我們的指示](./creating-a-linux-master.md), 您可以在以下位置找到:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 否則, 請參閱 API 伺服器的資訊清單檔案, 以檢查掛接點。