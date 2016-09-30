---
title: "在 Nano Server 上部署 Windows 容器"
description: "在 Nano Server 上部署 Windows 容器"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 185c83b69972765a72af2dbbf5d0c7d2551212ce
ms.openlocfilehash: 6ada7de02bbdfab8986fdfeeda60b6373a6e2d96

---

# 容器主機部署 - Nano Server

**這是初版內容，後續可能會變更。** 

本文將逐步部署內含 Windows 容器功能的基本 Nano Server。 此為進階主題，使用者應大致了解 Windows 和 Windows 容器。 如需 Windows 容器的簡介，請參閱 [Windows 容器快速入門](../quick_start/quick_start.md)。

## 準備 Nano Server

下一節將詳細說明最基本的 Nano Server 設定部署。 如需 Nano Server 部署和設定選項的詳盡說明，請參閱 Getting Started with Nano Server (https://technet.microsoft.com/en-us/library/mt126167.aspx) (開始使用 Nano Server)。

### 建立 Nano Server VM

請先從[這個位置](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)下載 Nano Server 評估 VHD。 透過這個 VHD 建立虛擬機器、啟動虛擬機器，並使用 Hyper-V 連線選項 (或依據所使用之虛擬化平台的對等項目) 連接到該虛擬機器。

### 建立遠端 PowerShell 工作階段

由於 Nano Server 沒有互動式登入功能，因此所有管理作業都需使用 PowerShell 從遠端系統來完成。

將 Nano Server 系統新增至遠端系統信任的主機。 以 Nano Server 的 IP 位址來取代此 IP 位址。

```none
Set-Item WSMan:\localhost\Client\TrustedHosts 192.168.1.50 -Force
```

建立遠端 PowerShell 工作階段。

```none
Enter-PSSession -ComputerName 192.168.1.50 -Credential ~\Administrator
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

需要先安裝 Docker 引擎，才能搭配使用 Windows 容器。 請使用下列步驟安裝 Docker 引擎。

首先，確定已經在 Nano Server 防火牆設定 SMB。 在 Nano Server 主機上執行此命令，即可完成此工作。

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

在 Nano Server 上，為 Docker 可執行檔建立一個資料夾。

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

下載 Docker 引擎及用戶端，然後將其複製到容器主機上的 'C:\Program Files\docker\' 中。 

> Nano Server 目前不支援 `Invoke-WebRequest`。 下載必須在遠端系統上完成，並將檔案複製到 Nano Server 主機。

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile .\docker.zip -UseBasicParsing
```

將下載的封裝解壓縮。 完成後，將會有包含 **dockerd.exe** 與 **docker.exe** 的目錄。 將這兩個檔案複製到 Nano Server 容器主機中的 **C:\Program Files\docker\** 資料夾。 

```none
Expand-Archive .\docker.zip
```

將 Docker 目錄新增至 Nano Server 上的系統路徑。

> 請務必切換回到遠端 Nano Server 工作階段。

```none
# For quick use, does not require shell to be restarted.
$env:path += “;C:\program files\docker”

# For persistent use, will apply even after a reboot.
setx PATH $env:path /M
```

將 Docker 安裝為 Windows 服務。

```none
dockerd --register-service
```

啟動 Docker 服務。

```none
Start-Service Docker
```

## 安裝基礎容器映像

基礎 OS 映像可作為任何 Windows Server 或 Hyper-V 容器的基底。 目前已經有以 Windows Server Core 與 Nano Server 做為基礎作業系統的基礎映像，並能使用 `docker pull` 來安裝。 如需 Docker 容器映像的詳細資訊，請參閱 [docker.com 上建置自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

若要下載並安裝 Windows Server 與 Nano Server 基礎映像，請執行下列命令。

```none
docker pull microsoft/nanoserver
```

```none
docker pull microsoft/windowsservercore
```

> 請參閱 Windows 容器 OS 映像授權條款，位於這裡：[授權條款](../Images_EULA.md)。

## 管理 Nano Server 上的 Docker

若要獲得最佳體驗，最佳做法是透過遠端系統管理 Nano Server 上的 Docker。 若要這樣做，必須先完成下列項目。

### 準備容器主機

針對 Docker 連線在容器主機上建立防火牆規則。 如果是不安全的連線，為連接埠 `2375`；安全的連線則為連接埠 `2376`。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2375
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

在您要使用的遠端系統上，下載 Docker 用戶端。

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

將壓縮過的封裝解壓縮。

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

執行下列兩個命令，以新增 Docker 目錄至系統路徑。

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

完成後即可使用 `docker -H` 參數存取遠端 Docker 主機。

```none
docker -H tcp://<IPADDRESS>:2375 run -it microsoft/nanoserver cmd
```

您可建立環境變數 `DOCKER_HOST`，並移除 `-H` 參數的需求。 您可以使用下列 PowerShell 命令來完成此作業：

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

使用此變數集時，此命令現在會看起來像這樣。

```none
docker run -it microsoft/nanoserver cmd
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



<!--HONumber=Sep16_HO4-->


