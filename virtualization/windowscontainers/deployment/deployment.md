---
title: 在 Windows Server 上部署 Windows 容器
description: 在 Windows Server 上部署 Windows 容器
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
---

# 容器主機部署 - Windows Server

**這是初版內容，後續可能會變更。** 

部署 Windows 容器主機有不同的步驟，視作業系統和主機系統類型 (實體或虛擬) 而定。 這份文件詳細說明將 Windows 容器主機部署至實體或虛擬系統上的 Windows Server 2016 或 Windows Server Core 2016 的步驟。

## 安裝容器功能

容器功能必須先啟用，才能使用 Windows 容器。 若要這麼做，請在提升權限的 PowerShell 工作階段中執行下列命令。 

```none
Install-WindowsFeature containers
```

功能安裝完成時，請重新啟動電腦。

## 安裝 Docker

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

將 Docker 目錄新增至系統路徑。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
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

## 安裝基本容器映像

必須先下載容器基本 OS 映像，才能部署容器。 下列範例會下載 Windows Server Core 基本 OS 映像。 安裝 Nano Server 基本映像時，也需完成此相同程序。 安裝 Nano Server 基本映像時，也需完成此相同程序。 如需 Windows 容器映像的詳細資訊，請參閱[管理容器映像](../management/manage_images.md)。
    
首先，安裝容器映像套件提供者。

```none
Install-PackageProvider ContainerImage -Force
```

接下來，安裝 Windows Server Core 映像。 此程序可能需要一些時間，因此您可以休息一下，等下載完成後再繼續。

```none 
Install-ContainerImage -Name WindowsServerCore    
```

基本映像安裝完成後，Docker 服務必須重新啟動。

```none
Restart-Service docker
```

最後，此映像必須加上「最新」版本的標籤。 若要這樣做，請執行以下命令。

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

## Hyper-V 容器主機

若要部署 Hyper-V 容器，需具備 Hyper-V 角色。 如果 Windows 容器主機本身為 Hyper-V 虛擬機器，則必須先啟用巢狀虛擬化，才能安裝 Hyper-V 角色。 如需巢狀虛擬化的詳細資訊，請參閱[巢狀虛擬化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。

### 巢狀虛擬化

下列指令碼可設定容器主機的巢狀虛擬化。 您必須在裝載容器主機虛擬機器的 Hyper-V 機器上執行此指令碼。 執行這個指令碼時，容器主機的虛擬機器務必要關閉。

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



<!--HONumber=May16_HO4-->


