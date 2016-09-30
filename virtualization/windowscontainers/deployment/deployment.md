---
title: "在 Windows Server 上部署 Windows 容器"
description: "在 Windows Server 上部署 Windows 容器"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: 4d7e8fb1fcbb7e9680b7d5bd143ef6d59e45035e

---

# 容器主機部署 - Windows Server

部署 Windows 容器主機有不同的步驟，視作業系統和主機系統類型 (實體或虛擬) 而定。 這份文件詳細說明將 Windows 容器主機部署至實體或虛擬系統上的 Windows Server 2016 或 Windows Server Core 2016 的步驟。

## 安裝容器功能

容器功能必須先啟用，才能使用 Windows 容器。 若要這麼做，請在提升權限的 PowerShell 工作階段中執行下列命令。

```none
Install-WindowsFeature containers
```

功能安裝完成時，請重新啟動電腦。

```none
Restart-Computer -Force
```

## 安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 針對此練習，兩者都會安裝。

下載 Docker 引擎與用戶端的 zip 封存。

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

將該 zip 封存展開到 Program Files; 該封存內容已在 docker 目錄中。

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

若要將 Docker 安裝為 Windows 服務，請執行下列命令。

```none
dockerd --register-service
```

安裝之後，就可以啟動服務。

```none
Start-Service Docker
```

## 安裝基礎容器映像

使用 Windows 容器之前，必須先安裝基礎映像。 目前已經有以 Windows Server Core 或 Nano Server 做為基礎作業系統的基礎映像。 如需 Docker 容器映像的詳細資訊，請參閱 [docker.com 上建置自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

若要安裝 Windows Server Core 基礎映像，請執行下列步驟：

```none
docker pull microsoft/windowsservercore
```

若要安裝 Nano Server 基礎映像，請執行下列步驟：

```none
docker pull microsoft/nanoserver
```

> 請參閱 Windows 容器 OS 映像授權條款，位於這裡：[授權條款](../Images_EULA.md)。

## Hyper-V 容器主機

若要執行 Hyper-V 容器，需具備 Hyper-V 角色。 如果 Windows 容器主機本身為 Hyper-V 虛擬機器，則必須先啟用巢狀虛擬化，才能安裝 Hyper-V 角色。 如需巢狀虛擬化的詳細資訊，請參閱[巢狀虛擬化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。

### 巢狀虛擬化

下列指令碼可設定容器主機的巢狀虛擬化。 此指令碼要在父 HYPER-V 機器上執行。 執行這個指令碼時，容器主機的虛擬機器務必要關閉。

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### 啟用 Hyper-V 角色

若要使用 PowerShell 啟用 Hyper-V 功能，請在提升權限的 PowerShell 工作階段中執行下列命令。

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=Sep16_HO4-->


