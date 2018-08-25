---
title: 在 Windows Server 上部署 Windows 容器
description: 在 Windows Server 上部署 Windows 容器
keywords: docker, 容器
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: b80dd0d231d0f9435b7cc1c5e2b35bbf5a59d793
ms.sourcegitcommit: a287211a0ed9cac7ebfe1718e3a46f0f26fc8843
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "2748884"
---
# <a name="container-host-deployment---windows-server"></a>容器主機部署 - Windows Server

部署 Windows 容器主機有不同的步驟，視作業系統和主機系統類型 (實體或虛擬) 而定。 這份文件詳細說明將 Windows 容器主機部署至實體或虛擬系統上的 Windows Server 2016 或 Windows Server Core 2016 的步驟。

## <a name="install-docker"></a>安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 

我們將使用 [OneGet 提供者 PowerShell 模組](https://github.com/OneGet/MicrosoftDockerProvider)安裝 Docker。 提供者會啟用您電腦上的 \[容器\] 功能並安裝 Docker，這會需要重新開機。 

開啟提高權限的 PowerShell 工作階段，並執行下列命令。

安裝 OneGet PowerShell 模組。

```PowerShell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

使用 OneGet 安裝最新版的 Docker。

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

安裝完成時，請重新啟動電腦。

```PowerShell
Restart-Computer -Force
```

## <a name="install-a-specific-version-of-docker"></a>安裝 Docker 特定版本

目前有兩個通道可供 Docker EE for Windows Server：

* `17.06` -如果您使用 Docker Enterprise Edition (Docker 引擎，UCP、 DTR) 使用這個版本。 `17.06` 預設值為。
* `18.03` -如果您正在執行 Docker EE 引擎單獨使用這個版本。

若要安裝的特定版本，請使用`RequiredVersion`標幟：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

安裝特定 Docker EE 版本可能需要更新至先前安裝的 DockerMsftProvider 模組。 若要更新：

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>更新 Docker

如果您需要從舊版的通道更新 Docker EE 引擎的更新版本的通道，同時使用這兩者`-Update`和`-RequiredVersion`標幟：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>安裝基礎容器映像

使用 Windows 容器之前，必須先安裝基本映像。 目前已經有以 Windows Server Core 或 Nano Server 做為容器作業系統的基本映像。 如需 Docker 容器映像的詳細資訊，請參閱 [docker.com 上建置自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

若要安裝 Windows Server Core 基本映像，請執行下列命令：

```PowerShell
docker pull microsoft/windowsservercore
```

若要安裝 Nano Server 基本映像，請執行下列命令：

```PowerShell
docker pull microsoft/nanoserver
```

> 請參閱 Windows 容器 OS 映像授權條款，位於這裡：[授權條款](../images-eula.md)。

## <a name="hyper-v-container-host"></a>Hyper-V 容器主機

若要執行 Hyper-V 容器，需具備 Hyper-V 角色。 如果 Windows 容器主機本身為 Hyper-V 虛擬機器，則必須先啟用巢狀虛擬化，才能安裝 Hyper-V 角色。 如需巢狀虛擬化的詳細資訊，請參閱[巢狀虛擬化]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)。

### <a name="nested-virtualization"></a>巢狀虛擬化

下列指令碼可設定容器主機的巢狀虛擬化。 此指令碼要在父 HYPER-V 機器上執行。 執行這個指令碼時，容器主機的虛擬機器務必要關閉。

```PowerShell
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>啟用 Hyper-V 角色

若要使用 PowerShell 啟用 Hyper-V 功能，請在提升權限的 PowerShell 工作階段中執行下列命令。

```PowerShell
Install-WindowsFeature hyper-v
```
