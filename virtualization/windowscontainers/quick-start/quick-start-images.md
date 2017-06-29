---
title: "容器部署快速入門 - 映像"
description: "容器部署快速入門"
keywords: "docker, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 355daae1b673f0b05f08d0706664967a825de6f7
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
---
# <a name="container-images-on-windows-server"></a>Windows Server 上的容器映像

在先前的 Windows Server 快速入門中，已從預先建立的 .NET Core 範例建立 Windows 容器。 這項練習詳列手動建立自訂容器映像的方法，使用 Dockerfile 自動建立容器映像，並將容器映像儲存在 Docker Hub 公開登錄中。

本快速入門專屬於 Windows Server 2016 上的 Windows Server 容器，並使用 Windows Server Core 容器基本映像。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

**必要條件：**

- 執行 Windows Server 2016 的電腦系統 (實體或虛擬)。
- 設定此系統的 Windows 容器功能和 Docker。 如需這些步驟的逐步解說，請參閱 [Windows Server 上的 Windows 容器](./quick-start-windows-server.md)。
- Docker 識別碼，這會用以將容器映像推送至 Docker Hub。 如果您沒有 Docker 識別碼，請在 [Docker Cloud](https://cloud.docker.com/) 註冊一個。

## <a name="1-container-image---manual"></a>1.容器映像 - 手動

如欲得到最佳的體驗，請從 Windows 命令殼層 (cmd.exe) 逐步進行本練習。

手動建立容器映像的第一個步驟是部署容器。 針對此範例，請從預先建立的 IIS 映像部署 IIS 容器。 部署容器後，您將會在來自該容器的殼層工作階段中工作。 互動式工作階段會以 `-it` 旗標起始。 如需 Docker Run 命令的深入詳細資訊，請參閱 [Docker.com 上的 Docker Run Reference](https://docs.docker.com/engine/reference/run/)。 

> 由於 Windows Server Core 基礎映像大小的緣故，這個步驟可能需要一些時間。

```none
docker run -d --name myIIS -p 80:80 microsoft/iis
```

此時，容器將在背景執行。 容器隨附的預設命令 `ServiceMonitor.exe` 會監視 IIS 進度，且將於 IIS 停止時自動停止容器。 若要深入了解此映像的建立過程，請參閱 GitHub 上的 [Microsoft/docker-iis](https://github.com/Microsoft/iis-docker)。

接下來，在容器內啟動互動式 cmd。 這將讓您可在執行中的容器內執行命令，而無須停止 IIS 或 ServiceMonitor。

```none
docker exec -i myIIS cmd 
```

接下來，您可以對執行中的容器進行變更。 執行下列命令以移除 IIS 啟動顯示畫面。

```none
del C:\inetpub\wwwroot\iisstart.htm
```

並且執行下列命令，將預設 IIS 網站取代為新的靜態網站。

```none
echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

從不同的系統中，瀏覽至容器主機的 IP 位址。 現在您應會看見 ‘Hello World’ 應用程式。

**注意︰** 如果您正在使用 Azure，必須有允許流量通過連接埠 80 的網路安全性群組規則。 如需詳細資訊，請參閱[建立現有 NSG 中的規則](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)。

![](media/hello.png)

回到容器中，結束互動式容器工作階段。

```none
exit
```

修改過的容器現在可以擷取至新的容器映像。 若要這樣做，您必須使用容器名稱。 使用 `docker ps -a` 命令即可找到。

```none
docker ps -a

CONTAINER ID     IMAGE                             COMMAND   CREATED             STATUS   PORTS   NAMES
489b0b447949     microsoft/iis   "cmd"     About an hour ago   Exited           pedantic_lichterman
```

若要建立新容器映像，請使用 `docker commit` 命令。 Docker commit 的形式為 “docker commit 容器名稱 新映像名稱”。 請注意，請將此範例中的容器名稱取代為實際的容器名稱。

```none
docker commit pedantic_lichterman modified-iis
```

若要確認已建立新映像，請使用 `docker images` 命令。  

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
modified-iis        latest              3e4fdb6ed3bc        About a minute ago   10.17 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago          9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago          9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago          9.344 GB
```

現在可以部署此映像。 產生的容器將包含所有擷取的修改。

## <a name="2-container-image---dockerfile"></a>2.容器映像 - Dockerfile

透過最後一項練習，容器已手動建立、修改，然後擷取至新的容器映像中。 Docker 包含使用 Dockerfile 將此程序自動化的方法。 此練習最後將有幾乎相同的結果，但這一次程序將會自動進行。 這項練習需要 Docker 識別碼。 如果您沒有 Docker 識別碼，請在 [Docker Cloud]( https://cloud.docker.com/) 註冊一個。

在容器主機上建立目錄 `c:\build`，然後在此目錄中建立名為 `Dockerfile` 的檔案。 請注意 – 檔案不應該具有副檔名。

```none
powershell new-item c:\build\Dockerfile -Force
```

在記事本中開啟 Dockerfile。

```none
notepad c:\build\Dockerfile
```

將下列文字複製到 Dockerfile 中，並儲存檔案。 這些命令會指示 Docker 建立新的映像，並且使用 `microsoft/iis` 做為基礎。 Dockerfile 接著會執行 `RUN` 指示中指定的命令，在此情況下，index.html 檔案會更新為新內容。 

如需 Dockerfile 的詳細資訊，請參閱 [Windows 上的 Dockerfile](../manage-docker/manage-windows-dockerfile.md)。

```none
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

`docker build` 命令將會啟動映像建置程序。 `-t` 參數會指示建置程序將新映像命名為 `iis-dockerfile`。 **以 Docker 帳戶的使用者名稱取代 'user'**。 如果您沒有 Docker 的帳戶，請在 [Docker Cloud](https://cloud.docker.com/) 註冊一個。

```none
docker build -t <user>/iis-dockerfile c:\Build
```

完成之後，您可以使用 `docker images` 命令來驗證映像是否已建立。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

現在請使用下列命令部署容器，並再次以 Docker 識別碼取代使用者。

```none
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

在容器建立後，請瀏覽至容器主機的 IP 位址。 您應會看見 hello world 應用程式。

![](media/dockerfile2.png)

回到容器主機上，使用 `docker ps` 以取得容器的名稱，使用 `docker rm` 則可移除容器。 請注意，請將此範例中的容器名稱取代為實際的容器名稱。

取得容器名稱。

```none
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

移除容器。

```none
docker rm -f <container name>
```

## <a name="3-docker-push"></a>3.Docker Push

Docker 容器映像可儲存於容器登錄中。 映像一旦儲存於登錄中，即可擷取以供日後在多種不同容器主機中使用。 Docker 提供公開登錄，以在 [Docker Hub](https://hub.docker.com/) 儲存容器映像。

在這項練習中，自訂 hello world 映像會推送至您在 Docker Hub 的帳戶。

首先，使用 `docker login command` 登入您的 Docker 帳戶。

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

登入後，即可將容器映像推送至 Docker Hub。 若要這樣做，請使用 `docker push` 命令。 **以您的 Docker 識別碼取代 'user’**。 

```none
docker push <user>/iis-dockerfile
```

現在可以使用 `docker pull`，將容器映像從 Docker Hub 下載至任何 Windows 容器主機。 在本教學課程中，我們會刪除現有映像，然後從 Docker Hub 加以提取。 

```none
docker rmi <user>/iis-dockerfile
```

執行 `docker images` 會顯示映像已經移除。

```none
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

最後可以使用 Docker 提取，將映像提取回容器主機。 以 Docker 帳戶的使用者名稱取代使用者。 

```none
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>後續步驟

[Windows 10 上的 Windows 容器](./quick-start-windows-10.md)
