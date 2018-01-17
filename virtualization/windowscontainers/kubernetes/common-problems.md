---
title: "疑難排解 Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "部署 Kubernetes 和加入 Windows 節點時常見問題的解決方案。"
keywords: "kubernetes, 1.9, linux, 編譯"
ms.openlocfilehash: 73b44ffd12fba58ac4ef38352c012061a6817945
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/05/2017
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

此外，特定的指令碼必須以系統管理員權限執行 (例如 `kubelet`)，而且前面應該加上 `sudo`。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>無法連線至位於 `https://[address]:[port]` 的 API 伺服器 ###
這個錯誤通常表示憑證問題。 請確定您已正確產生組態檔，其中的 IP 位址符合您的主機 IP 位址，而且您已將它複製到 API 伺服器裝載的目錄。

如果遵循[我們的指示](./creating-a-linux-master)，這會是在 `~/kube/kubelet/`；否則，請參考 API 伺服器的資訊清單檔以檢查裝載點。


## <a name="common-networking-errors"></a>常見的網路錯誤 ##
網路上或主機上可能有其他限制，防止節點之間特定類型的通訊。 請確定：

  - 允許來自 Pod 的流量
  - 允許 HTTP 流量，如果您要部署 Web 服務
  - 未捨棄 ICMP 封包


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>常見 Windows 錯誤 ##


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>我的 Windows Pod 無法存取 Linux 主機，反之亦然。 ###
如果您使用 Hyper-V 虛擬機器，請確定網路介面卡上已啟用 MAC 詐騙。


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>我的 Windows 節點無法使用服務 IP 存取我的服務。 ###
這是 Windows 目前網路堆疊的已知限制。
