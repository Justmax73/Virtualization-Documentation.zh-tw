---
title: "Windows Server 上的 Windows 容器"
description: "容器部署快速入門"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: b3f273d230344cff28d4eab7cebf96bac14f68c2
ms.openlocfilehash: 808436ba179daa09fbc45ee7f7708a505bd1b4c8

---

# Windows Server 上的 Windows 容器

**這是初版內容，後續可能會變更。**

本練習將引導進行 Windows Server 上的 Windows 容器功能基本部署和使用。 完成之後，您將會安裝容器角色，並部署簡單的 Windows Server 容器。 開始本快速入門之前，請先熟悉基本的容器概念與術語。 這項資訊可在[快快速入門簡介](./quick_start.md)中找到。

本快速入門是針對 Windows Server 2016 上的 Windows Server 容器。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

**必要條件：**

一個執行 [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview) 的電腦系統 (實體或虛擬)。

Azure 提供完整設定的 Windows Server 映像。 若要使用此映像，請按一下下方的按鈕部署虛擬機器。 部署約需要 10 分鐘。 完成之後，請登入 Azure 虛擬機器，並直接跳到本教學課程的步驟四。 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## 1.安裝容器功能

容器功能必須先啟用，才能使用 Windows 容器。 若要這麼做，請在提升權限的 PowerShell 工作階段中執行下列命令。

```none
Install-WindowsFeature containers
```

功能安裝完成時，請重新啟動電腦。

```none
Restart-Computer -Force
```

## 2.安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 針對此練習，兩者都會安裝。

下載 Docker 引擎與用戶端的 zip 封存。

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

將該 zip 封存展開到 Program Files; 該封存內容已在 docker 目錄中。

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

將 Docker 目錄新增至系統路徑。 完成時，重新啟動 PowerShell 工作階段，以便識別修改過的路徑。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

若要將 Docker 安裝為 Windows 服務，請執行下列命令。

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

安裝之後，就可以啟動服務。

```none
Start-Service docker
```

## 3.安裝基礎容器映像

會從範本或映像部署 Windows 容器。 必須先下載基礎 OS 映像，才能部署容器。 下列命令會下載 Windows Server Core 基礎映像。

```none
docker pull microsoft/windowsservercore
```

此程序可能需要一些時間才能完成。您可以稍事休息，待提取作業完成之後再繼續作業。

提取映像之後再執行 `docker images`，將會傳回已安裝之映像的清單。在此案例中為 Windows Server Core 映像。

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        7 weeks ago         7.764 GB
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

**注意︰**如果您在 Azure 中進行作業，則需具備虛擬機器的外部 IP 位址，並設定好網路安全性。 如需詳細資訊，請參閱[建立現有 NSG 中的規則]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)。

![](media/iis1.png)

回到容器主機上，使用 `docker rm` 命令以移除容器。 請注意，請將此範例中的容器名稱取代為實際的容器名稱。

```none
docker rm -f grave_jang
```
## 後續步驟

[Windows Server 上的容器映像](./quick_start_images.md)

[Windows 10 上的 Windows 容器](./quick_start_windows_10.md)



<!--HONumber=Aug16_HO1-->


