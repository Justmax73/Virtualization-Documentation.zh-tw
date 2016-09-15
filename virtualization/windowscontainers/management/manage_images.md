---
title: "Windows 容器映像"
description: "使用 Windows 容器建立和管理容器映像。"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
redirect_url: https://docs.docker.com/v1.8/userguide/dockerimages/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 49e29949fc91533cf1d3ed0aef47e829276772f4

---

# Windows 容器映像

**這是初版內容，後續可能會變更。** 

>Windows 容器是以 Docker 來管理。 Windows 容器文件為 [docs.docker.com](https://docs.docker.com/) 上文件的補充文件。

容器映像可用來部署容器。 這些映像可包含應用程式和所有的應用程式相依性。 例如，您可以開發已預先設定 Nano Server、IIS 和執行於 IIS 之應用程式的容器映像。 接著，此容器映像可儲存在容器登錄中供後續使用、部署在任何 Windows 容器主機上 (內部部署、雲端，甚至是容器服務)，也可以作為新容器映像的基底。

### 安裝映像

使用 Windows 容器之前，必須先安裝基礎映像。 目前已經有以 Windows Server Core 或 Nano Server 做為基礎作業系統的基礎映像。 如需支援設定的資訊，請參閱 [Windows 容器系統需求](../deployment/system_requirements.md)。

若要安裝 Windows Server Core 基礎映像，請執行下列步驟：

```none
docker pull microsoft/windowsservercore
```

若要安裝 Nano Server 基礎映像，請執行下列步驟：

```none
docker pull microsoft/nanoserver
```

### 列出映像

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        9 weeks ago         7.764 GB
microsoft/nanoserver          latest              3a703c6e97a2        9 weeks ago         969.8 MB
```

### 建立新的映像

您可以從任何現有容器建立新的容器映像。 若要這樣做，請使用 `docker commit` 命令。 下列範例會建立名為 ‘windowsservercoreiis’ 的新容器映像。

```none
docker commit 475059caef8f windowsservercoreiis
```

### 移除映像

如果有任何容器 (即便處於停止狀態) 具有對映像的相依性，容器映像即無法移除。

使用 docker 移除映像時，可依名稱或識別碼來參考映像。

```none
docker rmi windowsservercoreiis
```

### 映像相依性

若要使用 Docker 檢視映像相依性，可以使用 `docker history` 命令。

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Docker Hub 登錄包含可下載至容器主機上的預先建置映像。 這些映像在下載後，將可作為 Windows 容器應用程式的基底。

若要查看可從 Docker Hub 取得的映像清單，請使用 `docker search` 命令。 請注意，從 Docker Hub 提取這些依存容器映像前，必須先安裝 Windows Server Core 或 Nano Server 基礎 OS 映像。

這些映像大部分都具有 Windows Server Core 和 Nano Server 版本。 若要取得特定版本，只需加入 ":windowsservercore" 或 ":nanoserver" 標籤。 根據預設，除非只有 Nano Server 版本可用，否則「最新」標籤會傳回 Windows Server Core 版本。


```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

### Docker Pull

若要從 Docker Hub 下載映像，請使用 `docker pull`。 如需詳細資訊，請參閱 [Docker.com 上的 Docker Pull](https://docs.docker.com/engine/reference/commandline/pull/)。

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

現在，執行 `docker images` 時可以看見此映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

> 若 Docker Pull 失敗，請確定容器主機已套用最新版的累積更新。 如需 TP5 更新，請參閱 [KB3157663]( https://support.microsoft.com/en-us/kb/3157663)。

### Docker Push

容器映像也可上傳到 Docker Hub 或 Docker Trusted Registry。 上傳這些映像之後，就能供不同的 Windows 容器環境下載重複使用。

若要將容器映像上傳到 Docker Hub，請先登入 Registry。 如需詳細資訊，請參閱 [Docker.com 上的 Docker Login]( https://docs.docker.com/engine/reference/commandline/login/)。

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password:

Login Succeeded
```

登入您的 Docker Hub 或 Docker Trusted Registry 之後，請使用 `docker push` 上傳容器映像。 此容器映像可供名稱或識別碼參考。 如需詳細資訊，請參閱 [Docker.com 上的 Docker Push]( https://docs.docker.com/engine/reference/commandline/push/)。

```none
docker push username/containername

The push refers to a repository [docker.io/username/containername]
b567cea5d325: Pushed
00f57025c723: Pushed
2e05e94480e9: Pushed
63f3aa135163: Pushed
469f4bf35316: Pushed
2946c9dcfc7d: Pushed
7bfd967a5e43: Pushed
f64ea92aaebc: Pushed
4341be770beb: Pushed
fed398573696: Pushed
latest: digest: sha256:ae3a2971628c04d5df32c3bbbfc87c477bb814d5e73e2787900da13228676c4f size: 2410
```

此時容器映像已可使用，並可透過 `docker pull` 加以存取。






<!--HONumber=Sep16_HO2-->


