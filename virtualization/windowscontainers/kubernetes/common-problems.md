---
title: 疑難排解 Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: 部署 Kubernetes 和加入 Windows 節點時常見問題的解決方案。
keywords: kubernetes，1.12，linux，編譯
ms.openlocfilehash: a5e9369b000aa83aa7ec6ec9bb147f0fd844c820
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178911"
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

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>我是否可以設定而不是漂亮 windows 於背景中執行這一切？ ###
開始使用 Kubernetes 版本 1.11，讓 kubelet 與 kube proxy 可以執行做為原生[Windows 服務](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)。 您也一律可用來替代的服務管理員像[nssm.exe](https://nssm.cc/)一律為您在背景執行這些處理程序 (flanneld、 kubelet & kube proxy)。


## <a name="common-networking-errors"></a>常見的網路錯誤 ##

### <a name="my-windows-pods-do-not-have-network-connectivity"></a>我的 Windows pod 沒有網路連線 ###
如果您使用任何虛擬機器，請確定所有 VM 網路介面卡上已啟用 MAC 詐騙。 [反詐騙保護](./getting-started-kubernetes-windows.md#disable-anti-spoofing-protection)，如需詳細資訊，請參閱。


### <a name="my-windows-pods-cannot-ping-external-resources"></a>我的 Windows pod 無法 ping 外部資源 ###
Windows pod 沒有現今程式設計 ICMP 通訊協定的輸出規則。 但是，TCP/UDP 則支援。 當嘗試示範連線至叢集以外的資源，請取代`ping <IP>`使用對應的`curl <IP>`命令。

如果您仍然會遇到問題，很可能是您的網路設定中[cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)是值得一些額外的注意。 您可以隨時編輯此靜態的檔案，設定將會套用到新建立的任何 Kubernetes 資源。

為什麼？
Kubernetes 網路功能需求的其中一個 （請參閱[Kubernetes 模型](https://kubernetes.io/docs/concepts/cluster-administration/networking/)） 是叢集通訊內部 NAT 的情況下發生。 若要接受此需求，我們有所有通訊[ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)其中我們不希望發生的輸出 NAT。 不過，這也表示您需要將排除您嘗試從 ExceptionList 查詢的外部 IP。 然後只來自您的 Windows pod 的流量會 SNAT'ed 正確地從外界接收回應。 在這個問題，您在 ExceptionList`cni.conf`應該看起來如下：
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>在啟動 start.ps1 之後, Flanneld 停滯在同 」 等待 to be created 網路 」 ###
有許多此問題報告它所要調查;最有可能是為計時問題當 flannel 網路的管理 IP 設定。 因應措施是只要重新啟動 start.ps1 或重新啟動它以手動方式，如下所示：
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

也是目前可解決此問題，檢閱下方[PR](https://github.com/coreos/flannel/pull/1042) 。

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>我的 Windows pod 無法啟動因為缺少 /run/flannel/subnet.env ###
這表示 Flannel 無法正確啟動。 您可以嘗試重新啟動 flanneld.exe 或您可以將檔案複製手動從`/run/flannel/subnet.env`若要建立 Kubernetes 主機上`C:\run\flannel\subnet.env`Windows 背景工作節點上和修改`FLANNEL_SUBNET`列以不同的數字。 例如，如果節點的子網路 10.244.4.1/24 想要：
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>我的端點/Ip 流失 ###
有 2 個目前已知的問題可能會造成遺漏的端點。 
1.  [已知問題](https://github.com/kubernetes/kubernetes/issues/68511)的第一個是 Kubernetes 版本 1.11 中的問題。 請避免使用 Kubernetes 版本 1.11.0-1.11.2。
2. 第二個[已知問題](https://github.com/docker/libnetwork/issues/1950)，可能會造成遺漏的端點是在端點的儲存空間中的並行問題。 若要接收已修正問題，您必須使用 Docker EE 18.09 或更新版本。

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>我的 pod 無法啟動由於 」 網路： 為範圍配置失敗 」 錯誤 ###
這表示您的節點上的 IP 位址空間耗盡。 若要清除任何[遺漏的端點](#my-endpointsips-are-leaking)，請移轉受影響的程度的節點上的任何資源與執行下列命令：
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

請參閱 `start-kubelet.ps1` 指令碼的輸出，以了解虛擬網路建立期間是否發生錯誤。

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pod 在持續運作一段時間後順利停止解析 DNS 查詢 ###
已知的 DNS 快取中的 Windows Server 網路堆疊的問題，版本 1803年和下方，有時可能會造成 DNS 要求失敗。 若要解決此問題，您可以設定為零使用下列登錄機碼的最大 TTL 快取值：

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>我仍然看到問題。 我該怎麼辦？ ### 
網路上或主機上可能有其他限制，防止節點之間特定類型的通訊。 請確定：
  - 您已正確設定您所選擇的[網路拓撲](./network-topologies.md)
  - 允許來自 Pod 的流量
  - 允許 HTTP 流量，如果您要部署 Web 服務
  - 未捨棄從不同的通訊協定 (ie ICMP 與 TCP/UDP) 的封包


## <a name="common-windows-errors"></a>常見 Windows 錯誤 ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>我的 Kubernetes Pod 停滯在「ContainerCreating」 ###
此問題可能有很多成因，但其中一個最常見的原因是 Pause 映像設定錯誤。 這是下一個問題的高度徵兆。


### <a name="when-deploying-docker-containers-keep-restarting"></a>進行部署時，請 Docker 容器不斷重新啟動 ###
確認 Pause 影像與您的作業系統版本相容。 [指示](./deploying-resources.md)假設作業系統與容器版本 1803年。 如果您的 Windows 是較新的版本 (例如測試人員組建)，就必須相應調整該映像。 如需了解映像，請參閱 Microsoft 的 [Docker 儲存機制](https://hub.docker.com/u/microsoft/)。 什麼都別管，Pause 映像 Dockerfile 和範例服務反正都預期此映像會標記為 `:latest`。


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

 否則，請參考 API 伺服器的資訊清單檔，以檢查裝載點。