---
title: Windows 容器映像
description: 使用 Windows 容器建立和管理容器映像。
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
---

# Windows 容器映像

**這是初版內容，後續可能會變更。** 

容器映像可用來部署容器。 這些映像可包含作業系統、應用程式和所有的應用程式相依性。 例如，您可以開發已預先設定 Nano Server、IIS 和執行於 IIS 之應用程式的容器映像。 接著，此容器映像可儲存在容器登錄中供後續使用、部署在任何 Windows 容器主機上 (內部部署、雲端，甚至是容器服務)，也可以作為新容器映像的基底。

容器映像分成兩種類型：

**基本 OS 映像** – 此類型由 Microsoft 提供，其中包含核心 OS 元件。 

**容器映像** – 從基本 OS 映像衍生出的自訂容器映像。

## 基本 OS 映像

### 安裝映像

您可以使用 ContainerImage PowerShell 模組尋找並安裝容器 OS 映像。 此模組必須先安裝，才可使用。 您可以使用下列命令來安裝此模組。 如需有關使用容器映像 OneGet PowerShell 模組的詳細資訊，請參閱 [Container Image Provider](https://github.com/PowerShell/ContainerProvider) (容器映像提供者)。 

```none
Install-PackageProvider ContainerImage -Force
```

安裝完成後，即可使用 `Find-ContainerImage` 傳回基本 OS 映像的清單。

```none
Find-ContainerImage

Name                 Version          Source           Summary
----                 -------          ------           -------
NanoServer           10.0.14300.1010  ContainerImag... Container OS Image of Windows Server 2016 Technical...
WindowsServerCore    10.0.14300.1000  ContainerImag... Container OS Image of Windows Server 2016 Technical...
```

若要下載並安裝 Nano Server 基本 OS 映像，請執行下列命令。 `-version` 參數是選擇性的。 若未指定基本 OS 映像版本，將會安裝最新版本。

```none
Install-ContainerImage -Name NanoServer -Version 10.0.14300.1010
```

同樣地，此命令也將下載並安裝 Windows Server Core 基本 OS 映像。 `-version` 參數是選擇性的。 若未指定基本 OS 映像版本，將會安裝最新版本。

```none
Install-ContainerImage -Name WindowsServerCore -Version 10.0.14300.1000
```

使用 `docker images` 命令驗證已安裝映像。 

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     40356b90dc80        2 weeks ago         793.3 MB
windowsservercore   10.0.14304.1000     7837d9445187        2 weeks ago         9.176 GB
```  

安裝之後，建議您為映像標上「最新」標籤。 可以在下方 [標籤] 區段找到這些指示的詳細解說。

> 如果基本 OS 映像已下載完畢，但在執行 `docker images` 時未顯示，請使用服務控制台小程式或依序使用命令 'sc stop docker' 和 'sc start docker'，以重新啟動 Docker 服務

### 標籤映像

依名稱參考容器映像時，Docker 引擎會搜尋最新版本的映像。 如果無法判斷最新版本，將會擲回下列錯誤。

```none
docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

安裝 Windows Server Core 或 Nano Server 基本 OS 映像後，將必須以「最新」版本加以標記。 若要這樣做，請使用 `docker tag` 命令。 

如需有關 `docker tag` 的詳細資訊，請參閱 [Tag, push, and pull you images on docker.com](https://docs.docker.com/mac/step_six/) (在 docker.com 上標記、推播及提取您的映像)。 

```none
docker tag <image id> windowsservercore:latest
```

在加上標籤後，`docker images` 的輸出會顯示相同映像的兩種版本，一個具有映像版本的標籤，另一個則有「最新」標籤。 現在可依名稱參考映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14300.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### 離線安裝

基本 OS 映像也可以在沒有網際網路連線的情況下安裝。 若要這樣做，請在具有網際網路連線的電腦上下載映像、將它複製到目標系統，然後使用 `Install-ContainerOSImages` 命令匯入映像。

在下載基本 OS 映像前，請執行下列命令，讓**連接至網路**的系統備有容器映像提供者。

```none
Install-PackageProvider ContainerImage -Force
```

從 PowerShell OneGet 封裝管理員傳回映像清單：

```none
Find-ContainerImage
```

輸出：

```none
Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.14300.1010         Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.14300.1000         Container OS Image of Windows Server 2016 Techn...
```

若要下載映像，請使用 `Save-ContainerImage` 命令。

```none
Save-ContainerImage -Name NanoServer -Path c:\container-image
```

現在可以將下載好的容器映像複製到**離線容器主機**，並使用 `Install-ContainerOSImage` 命令進行安裝。

```none
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### 將 OS 映像解除安裝

可使用 `Uninstall-ContainerOSImage` 命令將基本 OS 映像解除安裝。 下列範例會將 NanoServer 基本 OS 映像解除安裝。

```none
Uninstall-ContainerOSImage -FullName CN=Microsoft_NanoServer_10.0.14304.1003
```

## 容器映像

### 列出映像

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.14300.1000     6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.14300.1010     8572198a60f1        2 weeks ago          0 B
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

若要查看可從 Docker Hub 取得的映像清單，請使用 `docker search` 命令。 請注意，從 Docker Hub 提取這些依存容器映像前，必須先安裝 Windows Server Core 或 Nano Server 基本 OS 映像。

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





<!--HONumber=Jun16_HO3-->


