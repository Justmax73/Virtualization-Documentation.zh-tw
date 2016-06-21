---
title: Windows Server 上的 Windows 容器
description: 容器部署快速入門
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
---

# Windows Server 上的 Windows 容器

**這是初版內容，後續可能會變更。** 

本練習將引導進行 Windows Server 上的 Windows 容器功能基本部署和使用。 完成之後，您將會安裝容器角色，並部署簡單的 Windows Server 容器。 開始本快速入門之前，請先熟悉基本的容器概念與術語。 這項資訊可在[快快速入門簡介](./quick_start.md)中找到。 

本快速入門是針對 Windows Server 2016 上的 Windows Server 容器。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

**必要條件：**

- 一個執行 [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview) 的電腦系統 (實體或虛擬)。

## 1.安裝容器功能

容器功能必須先啟用，才能使用 Windows 容器。 要這麼做，請在提高權限的 PowerShell 工作階段中執行下列命令。 

```none
Install-WindowsFeature containers
```

功能安裝完成時，請重新啟動電腦。

## 2.安裝 Docker

需要 Docker，才能使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 針對此練習，兩者都會安裝。

建立 Docker 可執行檔的資料夾。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

下載 Docker 精靈。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

下載 Docker 用戶端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

將 Docker 目錄新增至系統路徑。 完成時，重新啟動 PowerShell 工作階段，以便辨識修改過的路徑。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

若要將 Docker 安裝為 Windows 服務，請執行下列命令。

```none
dockerd --register-service
```

安裝之後，就可以啟動服務。

```none
Start-Service Docker
```

## 3.安裝基礎容器映像

會從範本或映像部署 Windows 容器。 必須先下載基礎 OS 映像，才能部署容器。 下列命令會下載 Windows Server Core 基礎映像。 
    
首先，安裝容器映像套件提供者。

```none
Install-PackageProvider ContainerImage -Force
```

接下來，安裝 Windows Server Core 映像。 此程序可能需要一些時間，因此您可以休息一下，等下載完成後再繼續。

```none 
Install-ContainerImage -Name WindowsServerCore    
```

基礎映像安裝完成後，Docker 服務必須重新啟動。

```none
Restart-Service docker
```

在這個階段，執行 `docker images` 會傳回一份已安裝的映像，在此案例中為 Windows Server Core 映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
windowsservercore   10.0.14300.1000     dbfee88ee9fd        7 weeks ago         9.344 GB
```

在繼續之前，此映像必須加上「最新」版本的標籤。 若要這樣做，請執行以下命令。

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

如需 Windows 容器映像的深入資訊，請參閱[管理容器映像](../management/manage_images.md)。

## 4.部署您的第一個容器

針對此練習中，您將從 Docker Hub 登錄下載預先建立的 IIS 映像，並部署執行 IIS 的簡單容器。  

若要在 Docker Hub 搜尋 Windows 容器映像，請執行 `docker search Microsoft`。  

```none
docker search microsoft

NAME                                         DESCRIPTION                                     
microsoft/sample-django:windowsservercore    Django installed in a Windows Server Core ...   
microsoft/dotnet35:windowsservercore         .NET 3.5 Runtime installed in a Windows Se...   
microsoft/sample-golang:windowsservercore    Go Programming Language installed in a Win...   
microsoft/sample-httpd:windowsservercore     Apache httpd installed in a Windows Server...   
microsoft/iis:windowsservercore              Internet Information Services (IIS) instal...   
microsoft/sample-mongodb:windowsservercore   MongoDB installed in a Windows Server Core...   
microsoft/sample-mysql:windowsservercore     MySQL installed in a Windows Server Core b...   
microsoft/sample-nginx:windowsservercore     Nginx installed in a Windows Server Core b...  
microsoft/sample-python:windowsservercore    Python installed in a Windows Server Core ...   
microsoft/sample-rails:windowsservercore     Ruby on Rails installed in a Windows Serve...  
microsoft/sample-redis:windowsservercore     Redis installed in a Windows Server Core b...   
microsoft/sample-ruby:windowsservercore      Ruby installed in a Windows Server Core ba...   
microsoft/sample-sqlite:windowsservercore    SQLite installed in a Windows Server Core ...  
```

使用 `docker pull` 下載 IIS 映像。  

```none
docker pull microsoft/iis:windowsservercore
```

可以使用 `docker images` 命令驗證映像下載。 請注意，您會看到基礎映像 (windowsservercore) 和 IIS 映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

使用 `docker run` 部署 IIS 容器。

```none
docker run -d -p 80:80 microsoft/iis:windowsservercore ping -t localhost
```

這個命令會執行 IIS 映像作為背景服務 (-d)，並設定網路功能，使得容器主機的連接埠 80 對應至容器的連接埠 80。
如需 Docker Run 命令的深入資訊，請參閱 [Docker.com 上的 Docker Run Reference]( https://docs.docker.com/engine/reference/run/)。


可以使用 `docker ps` 命令看到執行中的容器。 記下容器名稱，這將用於後續的步驟。

```none
docker ps

CONTAINER ID    IMAGE                             COMMAND               CREATED              STATUS   PORTS                NAMES
9cad3ea5b7bc    microsoft/iis:windowsservercore   "ping -t localhost"   About a minute ago   Up       0.0.0.0:80->80/tcp   grave_jang
```

從不同的電腦開啟網頁瀏覽器，並輸入容器主機的 IP 位址。 如果所有項目都已正確設定，您應該會看到 IIS 啟動顯示畫面。 這是要由裝載於 Windows 容器中的 IIS 執行個體所提供。

![](media/iis1.png)

回到容器主機上，使用 `docker rm` 命令以移除容器。 請注意，請將此範例中的容器名稱取代為實際的容器名稱。

```none
docker rm -f grave_jang
```
## 後續步驟

[Windows Server 上的容器映像](./quick_start_images.md)

[Windows 10 上的 Windows 容器](./quick_start_windows_10.md)


<!--HONumber=May16_HO4-->


