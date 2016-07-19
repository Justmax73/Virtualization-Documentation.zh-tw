---
title: "Windows 10 上的 Windows 容器"
description: "容器部署快速入門"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/07/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 5f42cae373b1f8f0484ffac82f5ebc761c37d050
ms.openlocfilehash: 9ef41ff031e8b7bc463e71f39ee6a3b8e4fd846e

---

# Windows 10 上的 Windows 容器

**這是初版內容，後續可能會變更。** 

本練習將逐步引導您了解 Windows 10 (測試人員組建 14372 及更高版本) 上 Windows 容器功能的基本部署與使用。 完成之後，您將會安裝容器角色，並部署簡單的 Hyper-V 容器。 開始本快速入門之前，請先熟悉基本的容器概念與術語。 這項資訊可在[快速入門簡介](./quick_start.md)中找到。 

本快速入門是針對 Windows 10 上的 Hyper-V 容器。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

**必要條件：**

- 一個執行 [Windows 10 測試人員版本](https://insider.windows.com/)的實體電腦系統。   
- 本快速入門可以在 Windows 10 虛擬機器上執行，但是需要啟用巢狀虛擬化。 [巢狀虛擬化指南](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)中可以找到詳細資訊。

## 1.安裝容器功能

容器功能必須先啟用，才能使用 Windows 容器。 要這麼做，請在提高權限的 PowerShell 工作階段中執行下列命令。 

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

因為 Windows 10 只支援 Hyper-V 容器，所以也必須啟用 Hyper-V 功能。 若要使用 PowerShell 啟用 Hyper-V 功能，請在提升權限的 PowerShell 工作階段中執行下列命令。

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

安裝完成時，請重新啟動電腦。

```none
Restart-Computer -Force
```

## 2.安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 針對此練習，兩者都會安裝。 若要這樣做，請執行以下命令。 

建立 Docker 可執行檔的資料夾。

```none
New-Item -Type Directory -Path $env:ProgramFiles\docker\
```

下載 Docker 精靈。

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/dockerd.exe -OutFile $env:ProgramFiles\docker\dockerd.exe
```

下載 Docker 用戶端。

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/docker.exe -OutFile $env:ProgramFiles\docker\docker.exe
```

將 Docker 目錄新增至系統路徑。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

重新啟動 PowerShell 工作階段，以便辨識修改過的路徑。

若要將 Docker 安裝為 Windows 服務，請執行下列命令。

```none
dockerd --register-service
```

安裝之後，就可以啟動服務。

```none
Start-Service Docker
```

## 3.安裝基礎容器映像

會從範本或映像部署 Windows 容器。 必須先下載容器基礎 OS 映像，才能部署容器。 下列命令會下載 Nano Server 基礎映像。
    
> 此程序適用於高於 14372 的 Windows 測試人員組建，並將於 ‘docker pull’ 運作後停止。

下載 Nano Server 基礎映像。 

```none
Start-BitsTransfer https://aka.ms/tp5/6b/docker/nanoserver -Destination nanoserver.tar.gz
```

安裝基礎映像。

```none  
docker load -i nanoserver.tar.gz
```

在這個階段，執行 `docker images` 會傳回一份已安裝的映像，在此案例中為 Nano Server 映像。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
```

在繼續之前，此映像必須加上「最新」版本的標籤。 若要這樣做，請執行以下命令。

```none
docker tag microsoft/nanoserver:10.0.14300.1030 nanoserver:latest
```

如需 Windows 容器映像的深入資訊，請參閱[管理容器映像](../management/manage_images.md)。

## 4.部署您的第一個容器

針對這個簡單的範例，已預先建立 .NET Core 映像。 使用 `docker pull` 命令下載此映像。

執行時，將會啟動容器、執行簡單的 .NET Core 應用程式，然後容器會結束。 

```none
docker pull microsoft/sample-dotnet
```

您可以使用 `docker images` 命令來驗證。

```none
docker 

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
microsoft/sample-dotnet  latest              28da49c3bff4        41 hours ago        918.3 MB
nanoserver               10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
nanoserver               latest              3f5112ddd185        3 weeks ago         810.2 MB
```

使用 `docker run` 命令執行容器。 下列範例指定 `--rm` 參數，這會指示 Docker 引擎刪除不再執行的容器。 

如需 Docker Run 命令的深入資訊，請參閱 [Docker.com 上的 Docker Run Reference]( https://docs.docker.com/engine/reference/run/)。

```none
docker run --isolation=hyperv --rm microsoft/sample-dotnet
```

**注意：**如果擲回逾時事件錯誤，請執行下列 PowerShell 指令碼，然後重試作業。

```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

`docker run` 命令的結果，是從 sample-dotnet 映像建立 Hyper-V 容器，然後執行範例應用程式 (將輸出傳到殼層)，接著停止並移除容器。 後續的 Windows 10 及容器快速啟動將會深入探究在 Windows 10 上的容器中建立及部署應用程式。

## 後續步驟

[Windows Server 上的 Windows 容器](./quick_start_windows_server.md)





<!--HONumber=Jul16_HO2-->


