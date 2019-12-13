---
title: 在 Windows Server 上部署 Windows 容器
description: 在 Windows Server 上部署 Windows 容器
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 09/09/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 6e3996af36b4a710f9a12b3a1371138b053a43d8
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909898"
---
# <a name="container-host-deployment-windows-server"></a>容器主機部署： Windows Server

部署 Windows 容器主機有不同的步驟，視作業系統和主機系統類型 (實體或虛擬) 而定。 這份文件詳細說明將 Windows 容器主機部署至實體或虛擬系統上的 Windows Server 2016 或 Windows Server Core 2016 的步驟。

## <a name="install-docker"></a>安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。

為了安裝 Docker，我們將使用[OneGet 提供者 PowerShell 模組](https://github.com/OneGet/MicrosoftDockerProvider)。 提供者將會在您的電腦上啟用容器功能，並安裝 Docker，這將需要重新開機。

開啟已提升許可權的 PowerShell 會話，然後執行下列 Cmdlet。

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

## <a name="install-a-specific-version-of-docker"></a>安裝特定版本的 Docker

適用于 Windows Server 的 Docker EE 目前有兩個可用的通道：

* `17.06`-如果您使用的是 Docker Enterprise Edition （Docker 引擎、UCP、DTR），請使用此版本。 `17.06` 是預設值。
* `18.03`-如果您要單獨執行 Docker EE 引擎，請使用此版本。

若要安裝特定版本，請使用 `RequiredVersion` 旗標：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

安裝特定的 Docker EE 版本可能需要更新先前安裝的 DockerMsftProvider 模組。 若要更新：

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>更新 Docker

如果您需要將 Docker EE 引擎從較早的通道更新為較新的通道，請同時使用 `-Update` 和 `-RequiredVersion` 旗標：

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>安裝基底容器映射

使用 Windows 容器之前，必須先安裝基本映像。 目前已經有以 Windows Server Core 或 Nano Server 做為容器作業系統的基本映像。 如需 Docker 容器映像的詳細資訊，請參閱 [docker.com 上建置自己的映像](https://docs.docker.com/engine/tutorials/dockerimages/)。

隨著 Windows Server 2019 的發行，Microsoft 來源的容器映射會移至名為 Microsoft Container Registry 的新登錄。 Microsoft 所發佈的容器映射應該繼續透過 Docker Hub 探索。 對於隨 Windows Server 2019 和更新發行的新容器映射，您應該尋找從 MCR 提取它們。 針對在 Windows Server 2019 之前發佈的舊版容器映射，您應該繼續從 Docker 的登錄中提取它們。

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 和更新版本

若要安裝「Windows Server Core」基底映射，請執行下列動作：

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

若要安裝「Nano Server」基底映射，請執行下列動作：

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 （版本1607-1803）

若要安裝 Windows Server Core 基本映像，請執行下列命令：

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:1607
```

若要安裝 Nano Server 基本映像，請執行下列命令：

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1803
```

> 請閱讀 Windows 容器 OS 映射的 EULA，其可在此找到- [eula](../images-eula.md)。

## <a name="hyper-v-isolation-host"></a>Hyper-v 隔離主機

您必須擁有 Hyper-v 角色，才能執行 Hyper-v 隔離。 如果 Windows 容器主機本身為 Hyper-V 虛擬機器，則必須先啟用巢狀虛擬化，才能安裝 Hyper-V 角色。 如需巢狀虛擬化的詳細資訊，請參閱[巢狀虛擬化](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)。

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

若要使用 PowerShell 啟用 Hyper-v 功能，請在提升許可權的 PowerShell 會話中執行下列 Cmdlet。

```PowerShell
Install-WindowsFeature hyper-v
```
