---
title: "疑難排解 Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "部署 Kubernetes 和加入 Windows 節點時常見問題的解決方案。"
keywords: "kubernetes, 1.9, linux, 編譯"
ms.openlocfilehash: b6be43f1afabdf8ef9c2ddc6f46ed5ac43a9e7a5
ms.sourcegitcommit: 2e8f1fd06d46562e56c9e6d70e50745b8b234372
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/14/2018
---
# <a name="troubleshooting-kubernetes"></a>疑難排解 Kubernetes #
此頁面逐步解說 Kubernetes 設定、網路及部署的數個常見問題。

> [!tip]
> 藉由提出 PR 到[我們的文件存放庫](https://github.com/MicrosoftDocs/Virtualization-Documentation/)，建議常見問題集項目。


## <a name="common-deployment-errors"></a>常見的部署錯誤 ##
偵錯 Kubernetes 主機可分成三大類 (依可能性)：

  - Kubernetes 系統容器有問題。
  - `kubelet` 執行方式有問題。
  - 系統有問題。


執行 `kubectl get pods -n kube-system` 以查看 Kubernetes 建立的 Pod；這可能會提供哪些發生當機或無法正確啟動的深入解析。 然後執行 `docker ps -a` 以查看所有支援這些 Pod 的原始容器。 最後，在懷疑可能造成問題上的容器執行 `docker logs [ID]`，以查看處理序的原始輸出。


### <a name="permission-denied-errors"></a>_「權限遭拒」_錯誤 ###
確認指令碼有可執行檔的權限：

```bash
chmod +x [script name]
```

此外，特定的指令碼必須以進階使用者權限執行 (例如 `kubelet`)，而且前面應該加上 `sudo`。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>無法連線至位於 `https://[address]:[port]` 的 API 伺服器 ###
這個錯誤通常表示憑證問題。 請確定您已正確產生組態檔，其中的 IP 位址符合您的主機 IP 位址，而且您已將它複製到 API 伺服器裝載的目錄。

如果遵循[我們的指示](./creating-a-linux-master)，這會是在 `~/kube/kubelet/`；否則，請參考 API 伺服器的資訊清單檔以檢查裝載點。


## <a name="common-networking-errors"></a>常見的網路錯誤 ##
網路上或主機上可能有其他限制，防止節點之間特定類型的通訊。 請確定：

  - 您已正確設定網路拓撲
  - 允許來自 Pod 的流量
  - 允許 HTTP 流量，如果您要部署 Web 服務
  - 未捨棄 ICMP 封包


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>常見 Windows 錯誤 ##

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pod 在持續運作一段時間後順利停止解析 DNS 查詢 ###
這是網路堆疊中已知會影響某些設定的問題。此問題要透過 Windows 維護快速處理。


### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>我的 Kubernetes Pod 停滯在「ContainerCreating」 ###
此問題可能有很多成因，但其中一個最常見的原因是 Pause 映像設定錯誤。 這是下一個問題的高度徵兆。


### <a name="when-deploying-docker-containers-keep-restarting"></a>進行部署時，請 Docker 容器不斷重新啟動 ###
確認 Pause 影像與您的作業系統版本相容。 [相關指令](./getting-started-kubernetes-windows.md)假設作業系統與容器的版本都是 1709。 如果您的 Windows 是較新的版本 (例如測試人員組建)，就必須相應調整該映像。 如需了解映像，請參閱 Microsoft 的 [Docker 儲存機制](https://hub.docker.com/u/microsoft/)。 什麼都別管，Pause 映像 Dockerfile 和範例服務反正都預期此映像會標記為 `microsoft/windowsservercore:latest`。


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>我的 Windows Pod 無法存取 Linux 主機，反之亦然 ###
如果您使用 Hyper-V 虛擬機器，請確定網路介面卡上已啟用 MAC 詐騙。


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 節點無法使用服務 IP 存取我的服務 ###
這是 Windows 目前網路堆疊的已知限制。 只有 Pod 才可以參考服務 IP。


### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>啟動 Kubelet 時找不到網路介面卡 ###
Windows 網路堆疊需要虛擬介面卡，才能讓 Kubernetes 網路功能運作。 如果下列命令未傳回任何結果 (在 Admin Shell 中)，表示虛擬網路建立作業 &mdash; 讓 Kubelet 運作所需的必要條件 &mdash; 失敗：

```powershell
Get-HnsNetwork | ? Name -ieq "l2bridge"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

請參閱 `start-kubelet.ps1` 指令碼的輸出，以了解虛擬網路建立期間是否發生錯誤。

