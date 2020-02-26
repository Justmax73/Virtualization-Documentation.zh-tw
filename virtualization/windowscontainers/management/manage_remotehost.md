---
title: Windows Docker 主機的遠端管理
description: 如何安全管理執行 Windows Server 的遠端 Docker 主機。
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 02/14/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 0cc1b621-1a92-4512-8716-956d7a8fe495
ms.openlocfilehash: 2e1fec6aa7149c801b1c72a0f8a346ca879015c2
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439515"
---
# <a name="remote-management-of-a-windows-docker-host"></a>Windows Docker 主機的遠端管理

即使在缺乏 `docker-machine` 的情況下，您仍然可在 Windows Server 2016 VM 上建立遠端存取 Docker 主機。

步驟相當直接易懂︰

* 使用 [dockertls](https://hub.docker.com/r/stefanscherer/dockertls-windows/) 建立伺服器上的憑證。 如果您要使用 IP 位址建立憑證，建議您使用靜態 IP，以避免 IP 位址變更時需重新建立憑證。

* 重新開機 docker 服務 `Restart-Service Docker`
* 建立允許輸入流量的 NSG 規則，以便使用連接埠 Docker 的 TLS 連接埠 2375 和 2376。 請注意，對於安全連線只需允許 2376。  
  入口網站應該會顯示如下的 NSG 設定︰  
  ![NGS](media/nsg.png)  
  
* 允許透過 Windows 防火牆的輸入連線。 
```
New-NetFirewallRule -DisplayName 'Docker SSL Inbound' -Profile @('Domain', 'Public', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2376
```
* 將電腦上您使用者的 docker 資料夾檔案 `ca.pem`、'cert.pem' 及 'key.pem' (例如`c:\users\chris\.docker`) 複製到您的本機電腦。 例如，您可以從 RDP 工作階段使用 ctrl-c 加上 ctrl-v 將檔案複製貼上。 
* 確認您可以連線到遠端 Docker 主機。 執行
```
docker -D -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify --tlscacert=c:\
users\foo\.docker\client\ca.pem --tlscert=c:\users\foo\.docker\client\cert.pem --tlskey=c:\users\foo\.doc
ker\client\key.pem ps
```


## <a name="troubleshooting"></a>疑難排解
### <a name="try-connecting-without-tls-to-determine-your-nsg-firewall-settings-are-correct"></a>嘗試不使用 TLS 進行連線來判斷您的 NSG 防火牆設定是否正確
連線錯誤通常會顯示為如下錯誤︰
```
error during connect: Get https://wsdockerhost.southcentralus.cloudapp.azure.com:2376/v1.25/version: dial tcp 13.85.27.177:2376: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

新增未加密的連線，方法是將 
```
{
    "tlsverify":  false,
}
```
新增至 `c"\programdata\docker\config\daemon.json` 然後重新啟動服務。

使用如下的命令列連線到遠端主機︰
```
docker -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify=0 version
```

### <a name="cert-problems"></a>憑證問題
使用非針對 IP 位址或 DNS 名稱所建立的憑證存取 Docker 主機，將導致發生錯誤︰
```
error during connect: Get https://w.x.y.c.z:2376/v1.25/containers/json: x509: certificate is valid for 127.0.0.1, a.b.c.d, not w.x.y.z
```
請確保 w.x.y.z 是主機公用 IP 的 DNS 名稱，且任一 DNS 名稱符合憑證的[通用名稱](https://www.ssl.com/faqs/common-name/)，而該名稱為 `SERVER_NAME` 環境變數或提供給 dockertls 之 `IP_ADDRESSES` 變數中的其中一個 IP 位址

### <a name="cryptox509-warning"></a>crypto/x509 警告
您可能會收到警告 
```
level=warning msg="Unable to use system certificate pool: crypto/x509: system root pool is not available on Windows"
```
此為善意警告。
