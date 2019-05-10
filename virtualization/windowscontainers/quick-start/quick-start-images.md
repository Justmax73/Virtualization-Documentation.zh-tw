---
title: 容器部署快速入門 - 映像
description: 容器部署快速入門
keywords: docker, 容器
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: db360bdd2b62667ab017549b3c179d11278abc19
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620796"
---
# <a name="automating-builds-and-saving-images"></a>自動化組建和儲存映像

在先前的 Windows Server 快速入門中，已從預先建立的 .NET Core 範例建立 Windows 容器。 這項練習示範如何根據 Dockerfile 建置您自己的容器映像和容器映像儲存在 Docker Hub 公開登錄中。

本快速入門是針對 Windows Server 2019 或 Windows Server 2016 上的 Windows Server 容器，並將會使用 Windows Server Core 容器基本映像。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

## <a name="prerequisites"></a>必要條件

請確定您符合下列需求：

- 一部電腦系統 （實體或虛擬） 正在執行的 Windows Server 2019 或 Windows Server 2016。
- 設定此系統的 Windows 容器功能和 Docker。 如需這些步驟的逐步解說，請參閱[Windows Server 上的 Windows 容器](./quick-start-windows-server.md)。
- Docker 識別碼，這會用以將容器映像推送至 Docker Hub。 如果您沒有 Docker 識別碼，請在 [Docker Cloud](https://cloud.docker.com/) 註冊一個。

## <a name="container-image---dockerfile"></a>容器映像-Dockerfile

雖然容器可以手動建立、修改，然後擷取至到新的容器映像中，但是 Docker 包含使用 Dockerfile 將此程序自動化的方法。 這項練習需要 Docker 識別碼。 如果您沒有 Docker 識別碼，請在 [Docker Cloud](https://cloud.docker.com/) 註冊一個。

在容器主機上建立目錄 `c:\build`，然後在此目錄中建立名為 `Dockerfile` 的檔案。 請注意 – 檔案不應該具有副檔名。

```console
powershell new-item c:\build\Dockerfile -Force
```

在記事本中開啟 Dockerfile。

```console
notepad c:\build\Dockerfile
```

將下列文字複製到 Dockerfile 中，並儲存檔案。 這些命令會指示 Docker 建立新的映像，並且使用 `microsoft/iis` 做為基礎。 Dockerfile 接著會執行 `RUN` 指示中指定的命令，在此情況下，index.html 檔案會更新為新內容。

如需 Dockerfile 的詳細資訊，請參閱 [Windows 上的 Dockerfile](../manage-docker/manage-windows-dockerfile.md)。

```dockerfile
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

`docker build` 命令將會啟動映像建置程序。 `-t` 參數會指示建置程序將新映像命名為 `iis-dockerfile`。 **以 Docker 帳戶的使用者名稱取代 'user'**。 如果您沒有 Docker 的帳戶，請在 [Docker Cloud](https://cloud.docker.com/) 註冊一個。

```console
docker build -t <user>/iis-dockerfile c:\Build
```

完成之後，您可以使用 `docker images` 命令來驗證映像是否已建立。

```console
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

現在請使用下列命令部署容器，並再次以 Docker 識別碼取代使用者。

```console
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

在容器建立後，請瀏覽至容器主機的 IP 位址。 您應會看見 hello world 應用程式。

![](media/dockerfile2.png)

回到容器主機上，使用 `docker ps` 以取得容器的名稱，使用 `docker rm` 則可移除容器。 請注意，請將此範例中的容器名稱取代為實際的容器名稱。

取得容器名稱。

```console
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

停止容器。

```console
docker stop <container name>
```

移除容器。

```console
docker rm -f <container name>
```

## <a name="docker-push"></a>3.docker Push

Docker 容器映像可儲存於容器登錄中。 映像一旦儲存於登錄中，即可擷取以供日後在多種不同容器主機中使用。 Docker 提供公開登錄，以在 [Docker Hub](https://hub.docker.com/) 儲存容器映像。

在這項練習中，自訂 hello world 映像會推送至您在 Docker Hub 的帳戶。

首先，使用 `docker login command` 登入您的 Docker 帳戶。

```console
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

登入後，即可將容器映像推送至 Docker Hub。 若要這樣做，請使用 `docker push` 命令。 **以您的 Docker 識別碼取代 'user’**。 

```console
docker push <user>/iis-dockerfile
```

當 Docker 推入最多 Docker Hub 每個層，docker 會跳過層已經存在於在 Docker Hub 中，或其他登錄 （外部層）。  例如，最新版的 Windows Server Core 會裝載於 Microsoft 容器登錄中或從私人公司的登錄，層級，會略過，然後不推入至 Docker Hub。

現在可以使用 `docker pull`，將容器映像從 Docker Hub 下載至任何 Windows 容器主機。 在本教學課程中，我們會刪除現有映像，然後從 Docker Hub 加以提取。 

```console
docker rmi <user>/iis-dockerfile
```

執行 `docker images` 會顯示映像已經移除。

```console
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

最後可以使用 Docker 提取，將映像提取回容器主機。 以 Docker 帳戶的使用者名稱取代使用者。 

```
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>後續步驟

如果您想要了解如何封裝範例 ASP.NET 應用程式，請瀏覽以下連結的 Windows 10 教學課程。

> [!div class="nextstepaction"]
> [Windows 10 上的容器](./quick-start-windows-10.md)
