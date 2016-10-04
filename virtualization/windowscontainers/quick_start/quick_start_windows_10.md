---
title: "Windows 10 上的 Windows 容器"
description: "容器部署快速入門"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 0fae34a5a85678a25c47b0312650e67aa6cd7efd
ms.openlocfilehash: 74686f222e8eec1daacd45a9e388d94abf6381f4

---

# Windows 10 上的 Windows 容器

本練習將逐步引導您了解 Windows 10 Professional 或 Enterprise (Anniversary Edition) 上 Windows 容器功能的基本部署與使用。 完成之後，您將會安裝容器角色，並部署簡單的 Hyper-V 容器。 開始本快速入門之前，請先熟悉基本的容器概念與術語。 這項資訊可在[快速入門簡介](./quick_start.md)中找到。

本快速入門是針對 Windows 10 上的 Hyper-V 容器。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

**必要條件：**

- 執行 Windows 10 年度版 (專業版或企業版) 的一部實體電腦系統。   
- 本快速入門可以在 Windows 10 虛擬機器上執行，但是需要啟用巢狀虛擬化。 [巢狀虛擬化指南](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)中可以找到詳細資訊。

> Windows 容器功能需要重大更新才能運作。 請先安裝所有更新，再循序完成本教學課程。

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

> 如果您先前在具有 Technical Preview 5 容器基本映像的 Windows 10 使用 Hyper-V 容器，請務必重新啟用 OpLock。 請執行下列命令：  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2.安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 針對此練習，兩者都會安裝。 若要這樣做，請執行以下命令。

下載 Docker 引擎與用戶端的 zip 封存。

```none
Invoke-WebRequest "https://master.dockerproject.org/windows/amd64/docker-1.13.0-dev.zip" -OutFile "$env:TEMP\docker-1.13.0-dev.zip" -UseBasicParsing
```

將該 zip 封存展開到 Program Files; 該封存內容已在 docker 目錄中。

```none
Expand-Archive -Path "$env:TEMP\docker-1.13.0-dev.zip" -DestinationPath $env:ProgramFiles
```

將 Docker 目錄新增至系統路徑。

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot.
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

會從範本或映像部署 Windows 容器。 必須先下載容器基礎 OS 映像，才能部署容器。 下列命令會下載 Nano Server 基礎映像。

提取 Nano Server 基礎映像。

```none
docker pull microsoft/nanoserver
```

提取映像之後再執行 `docker images`，將會傳回已安裝之映像的清單。在此案例中為 Nano Server 映像。

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

如需 Windows 容器映像的深入資訊，請參閱[管理容器映像](../management/manage_images.md)。

> 請參閱 Windows 容器 OS 映像授權條款，位於這裡：[授權條款](../Images_EULA.md)。

## 4.部署您的第一個容器

在這個簡單的範例中，將會建立及部署 'Hello World' 容器映像。 請在提高權限的 Windows 命令殼層中執行這些命令，讓作業進行得更加順暢。

首先，請利用互動式工作階段，從 `nanoserver` 映像啟動容器。 容器啟動之後，會在容器中顯示命令殼層。  

```none
docker run -it microsoft/nanoserver cmd
```

我們將在容器中建立簡單的 'Hello World' 指令碼。

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

完成之後，請結束容器。

```none
exit
```

接下來，請使用修改後的容器建立新的容器映像。 如需容器的清單，請執行下列命令並記下容器的識別碼。

```none
docker ps -a
```

請執行下列命令來建立新的 'HelloWorld' 映像。 以您的容器識別碼取代 <containerid>。

```none
docker commit <containerid> helloworld
```

完成之後，您的自訂映像中就會包含 'Hello World' 指令碼。 您可以使用下列命令確認。

```none
docker images
```

最後，若要執行該容器，請使用 `docker run` 命令。

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

`docker run` 命令的結果，在從 'HelloWorld’ 映像建立 Hyper-V 容器，接著執行範例 'HelloWorld’ 指令碼 (將輸出傳送到殼層)，然後停止容器並加以移除。
後續的 Windows 10 及容器快速啟動將會深入探究在 Windows 10 上的容器中建立及部署應用程式。

## 後續步驟

[Windows Server 上的 Windows 容器](./quick_start_windows_server.md)



<!--HONumber=Sep16_HO5-->


