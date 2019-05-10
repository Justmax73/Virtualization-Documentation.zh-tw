---
title: 在 Windows Server 上部署 Windows 容器
description: 在 Windows Server 上部署 Windows 容器
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: f4c6b37c6e33593be0237bd4059435a99c2bdd86
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620826"
---
# <a name="container-host-deployment-windows-server"></a>容器主機部署： Windows Server

部署 Windows 容器主機有不同的步驟，視作業系統和主機系統類型 (實體或虛擬) 而定。 這份文件詳細說明將 Windows 容器主機部署至實體或虛擬系統上的 Windows Server 2016 或 Windows Server Core 2016 的步驟。

## <a name="install-docker"></a>安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。

若要安裝 Docker，我們將使用[OneGet 提供者 PowerShell 模組](https://github.com/OneGet/MicrosoftDockerProvider)。 提供者會啟用您電腦上的容器功能，並安裝 Docker，將會需要重新開機。

開啟提升權限的 PowerShell 工作階段並執行下列 cmdlet。

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

## <a name="install-a-specific-version-of-docker"></a>安裝 Docker 的特定版本

目前有兩種管道適用於 Docker EE 適用於 Windows Server 的：

* `17.06` -如果您使用 Docker Enterprise Edition (Docker 引擎，UCP，DTR)，請使用這個版本。 `17.06` 是預設值。
* `18.03` -如果您正在執行 Docker EE 引擎單獨使用這個版本。

若要安裝的特定版本，請使用`RequiredVersion`旗標：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

安裝特定的 Docker EE 版本可能需要先前已安裝 DockerMsftProvider 模組的更新。 若要更新：

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>更新 Docker

如果您需要從較舊版本的通道的 Docker EE 引擎更新到更新版本的通道，使用這兩者`-Update`和`-RequiredVersion`旗標：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>安裝基礎容器映像

使用 Windows 容器之前，必須先安裝基本映像。 目前已經有以 Windows Server Core 或 Nano Server 做為容器作業系統的基本映像。 如需 Docker 容器映像的詳細資訊，請參閱 [docker.com 上建置自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

Windows Server 2019 版本中，Microsoft 為來源的容器映像要移轉至新的登錄，稱為 Microsoft 容器登錄。 可透過 Docker Hub 探索到應該繼續在 Microsoft 發行的容器映像。 新容器映像發佈使用 Windows Server 2019 及更新版本中，您應該看從 MCR 提取它們。 針對舊版 Windows Server 2019 之前發佈的容器映像，您應該繼續提取它們從 Docker 的登錄。

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 和更新版本

若要安裝 Windows Server Core' 基本映像執行下列命令：

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

若要安裝 Nano Server 基礎映像，請執行下列命令：

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 （版本 1607年 1803年）

若要安裝 Windows Server Core 基本映像，請執行下列命令：

```PowerShell
docker pull microsoft/windowsservercore
```

若要安裝 Nano Server 基本映像，請執行下列命令：

```PowerShell
docker pull microsoft/nanoserver
```

> 請閱讀 Windows 容器 OS 映像授權條款，這可以在這裡找到 – [EULA](../images-eula.md)。

## <a name="hyper-v-isolation-host"></a>HYPER-V 隔離主機

您必須有執行 HYPER-V 隔離的 HYPER-V 角色。 如果 Windows 容器主機本身為 Hyper-V 虛擬機器，則必須先啟用巢狀虛擬化，才能安裝 Hyper-V 角色。 如需巢狀虛擬化的詳細資訊，請參閱[巢狀虛擬化](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)。

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

若要啟用使用 PowerShell 的 HYPER-V 功能，請在提升權限的 PowerShell 工作階段中執行下列 cmdlet。

```PowerShell
Install-WindowsFeature hyper-v
```
