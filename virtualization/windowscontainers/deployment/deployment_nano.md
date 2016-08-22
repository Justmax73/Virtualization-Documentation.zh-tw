---
title: "在 Nano Server 上部署 Windows 容器"
description: "在 Nano Server 上部署 Windows 容器"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 07/06/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: fac57150de3ffd6c7d957dd628b937d5c41c1b35
ms.openlocfilehash: d2f19e96f06ba18ab7e23e62652f569265c6f43f

---

# 容器主機部署 - Nano Server

**這是初版內容，後續可能會變更。** 

本文將逐步部署內含 Windows 容器功能的基本 Nano Server。 此為進階主題，使用者應大致了解 Windows 和 Windows 容器。 如需 Windows 容器的簡介，請參閱 [Windows 容器快速入門](../quick_start/quick_start.md)。

## 準備 Nano Server

下一節將詳細說明最基本的 Nano Server 設定部署。 如需 Nano Server 部署和設定選項的詳盡說明，請參閱 Getting Started with Nano Server (https://technet.microsoft.com/en-us/library/mt126167.aspx) (開始使用 Nano Server)。

### 建立 Nano Server VM

請先從[這個位置](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)下載 Nano Server 評估 VHD。 透過這個 VHD 建立虛擬機器、啟動虛擬機器，並使用 Hyper-V 連線選項 (或依據所使用之虛擬化平台的對等項目) 連接到該虛擬機器。

接下來，您必須設定系統管理密碼。 若要這樣做，請在 Nano Server 修復主控台上按 `F11`。 隨即顯示 [變更密碼] 對話方塊。

### 建立遠端 PowerShell 工作階段

由於 Nano Server 沒有互動式登入功能，因此所有管理作業都需透過遠端 PowerShell 工作階段來完成。 若要建立遠端工作階段，請使用 Nano Server 修復主控台的網路區段，取得系統的 IP 位址，然後在遠端主機上執行下列命令。 將 IPADDRESS 取代為 Nano Server 系統的實際 IP 位址。

將 Nano Server 系統新增至信任的主機。

```none
set-item WSMan:\localhost\Client\TrustedHosts IPADDRESS -Force
```

建立遠端 PowerShell 工作階段。

```none
Enter-PSSession -ComputerName IPADDRESS -Credential ~\Administrator
```

完成這些步驟後，您即會進入 Nano Server 系統的遠端 PowerShell 工作階段。 除非另有說明，否則本文件其餘內容皆為透過遠端工作階段來執行。


## 安裝容器功能

Nano Server 套件管理提供者可讓您在 Nano Server 上安裝角色與功能。 請使用這個命令來安裝提供者。

```none
Install-PackageProvider NanoServerPackage
```

安裝套件提供者之後，請安裝容器的功能。

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

安裝這些容器功能之後，必須重新啟動 Nano Server。 

```none
Restart-Computer
```

備份好時，請重新建立遠端 PowerShell 連線。

## 安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 使用這些步驟安裝 Docker 引擎與用戶端。

在 Nano Server 上，為 Docker 可執行檔建立一個資料夾。

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

下載 Docker 引擎及用戶端，然後將其複製到容器主機上的 'C:\Program Files\docker\' 中。 

**注意** - Nano Server 目前不支援 `Invoke-WebRequest`，因此必須從遠端系統完成下載，然後複製到 Nano Server 主機。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

下載 Docker 用戶端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

下載 Docker 引擎和用戶端之後，請將其複製到 Nano Server 容器主機中的 'C:\Program Files\docker\' 資料夾。 Nano Server 防火牆必須設為允許連入的 SMB 連線。 您可以使用 PowerShell 或 Nano Server 修復主控台來完成這項作業。 

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

現在，您可以使用標準的 SMB 檔案複製方法來複製檔案。

將 dockerd.exe 檔案複製到主機後，請執行此命令以將 Docker 安裝為 Windows 服務。

```none
& $env:ProgramFiles'\docker\dockerd.exe' --register-service
```

啟動 Docker 服務。

```none
Start-Service Docker
```

## 安裝基礎容器映像

基本 OS 映像可作為任何 Windows Server 或 Hyper-V 容器的基底。 基本 OS 映像可作為 Windows Server Core 和 Nano Server 的基礎作業系統，而且可以使用容器映像提供者加以安裝。 如需 Windows 容器映像的詳細資訊，請參閱[管理容器映像](../management/manage_images.md)。

您可以使用下列命令來安裝容器映像提供者。

```none
Install-PackageProvider ContainerImage -Force
```

若要下載並安裝 Nano Server 基本映像，請執行下列命令：

```none
Install-ContainerImage -Name NanoServer
```

**注意：**目前只有 Nano Server 基本映像與 Nano Server 容器主機相容。

重新啟動 Docker 服務。

```none
Restart-Service Docker
```

將此 Nano Server 基底映像標記為最新版。

```none
& $env:ProgramFiles'\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## 管理 Nano Server 上的 Docker

若要獲得最佳體驗，最佳做法是透過遠端系統管理 Nano Server 上的 Docker。 若要這樣做，必須先完成下列項目。

### 準備容器主機

針對 Docker 連線在容器主機上建立防火牆規則。 如果是不安全的連線，為連接埠 `2375`；安全的連線則為連接埠 `2376`。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

將 Docker 引擎設為接受透過 TCP 的連入連線。

首先，在 Nano Server 主機上的 `c:\ProgramData\docker\config\daemon.json` 位置，建立 `daemon.json` 檔案。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

接著，執行下列命令，將連線設定新增至 `daemon.json` 檔案。 如此會將 Docker 引擎設為接受透過 TCP 連接埠 2375 的連入連線。 這是不安全的連線，因此並不建議您使用，但可用於隔離測試。 如需保護連線安全的詳細資訊，請參閱 Docker.com 上的 [Protect the Docker Daemon](https://docs.docker.com/engine/security/https/) (保護 Docker 精靈)。

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

重新啟動 Docker 服務。

```none
Restart-Service docker
```

### 準備遠端用戶端

在您要使用的遠端系統上建立目錄，以儲存 Docker 用戶端。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

將 Docker 用戶端下載至這個目錄中。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "$env:ProgramFiles\docker\docker.exe"
```

將 Docker 目錄加入系統路徑中。

```none
$env:Path += ";$env:ProgramFiles\Docker"
```

重新啟動 PowerShell 或命令工作階段，使其能辨識修改後的路徑。

完成後即可使用 `docker -H` 參數存取遠端 Docker 主機。

```none
docker -H tcp://<IPADDRESS>:2375 run -it nanoserver cmd
```

您可建立環境變數 `DOCKER_HOST`，並移除 `-H` 參數的需求。 您可以使用下列 PowerShell 命令來完成此作業：

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

使用此變數集時，此命令現在會看起來像這樣。

```none
docker run -it nanoserver cmd
```

## Hyper-V 容器主機

若要部署 Hyper-V 容器，容器主機上需具備 Hyper-V 角色。 如需 Hyper-V 容器的詳細資訊，請參閱 [Hyper-V 容器](../management/hyperv_container.md)。

若 Windows 容器主機本身即是 Hyper-V 虛擬機器，必須先啟用巢狀虛擬化。 如需巢狀虛擬化的詳細資訊，請參閱[巢狀虛擬化](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。


在 Nano Server 容器主機上，安裝 Hyper-V 角色。

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

安裝 HYPER-V 角色之後，必須重新啟動 Nano Server 主機。

```none
Restart-Computer
```


<!--HONumber=Aug16_HO3-->


