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
ms.sourcegitcommit: 41561cacc8c2531f1351154d85861f1712182c9a
ms.openlocfilehash: 091007ea301226ca98c93855d8c36b09f3e4d0be

---

# 容器主機部署 - Windows Server

部署 Windows 容器主機有不同的步驟，視作業系統和主機系統類型 (實體或虛擬) 而定。 這份文件詳細說明將 Windows 容器主機部署至實體或虛擬系統上的 Windows Server 2016 或 Windows Server Core 2016 的步驟。

## 安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 

我們將使用 [OneGet 提供者 PowerShell 模組](https://github.com/oneget/oneget)安裝 Docker。 提供者會啟用您電腦上的 [容器] 功能並安裝 Docker，這會需要重新開機。 

開啟提高權限的 PowerShell 工作階段，並執行下列命令。

首先安裝 OneGet PowerShell 模組。

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

接著使用 OneGet 安裝最新版的 Docker。

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

安裝完成時，請重新啟動電腦。

```none
Restart-Computer -Force
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



<!--HONumber=Oct16_HO2-->


