---
title: Windows Server 上的 Windows 容器
description: 容器部署快速入門
keywords: docker, 容器
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: 665e6186ff5f9530f12ba3d0a400d82bcac755a7
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999195"
---
# <a name="windows-containers-on-windows-server"></a>Windows Server 上的 Windows 容器

此練習將逐步引導您在 Windows Server 2019 和 Windows Server 2016 上進行基本的部署和使用 Windows 容器功能。

在這個快速入門中, 您將完成下列作業:

1. 啟用 Windows Server 中的 [容器] 功能
2. 安裝 Docker
3. 執行簡單的 Windows 容器

如果您需要熟悉一下容器，可以在[關於容器](../about/index.md)中找到這項資訊。

此 [快速入門] 專用於 Windows Server 2019 和 Windows Server 2016 上的 Windows Server 容器。 在此頁面左側的目錄中，可以找到其他的快速入門文件及 Windows 10 中的容器。

## <a name="prerequisites"></a>必要條件

請確定您符合下列需求:
- 一種執行 Windows Server 2019 的電腦系統 (物理或虛擬)。 如果您使用的是 Windows Server 2019 測試人員預覽版, 請更新至[Window server 2019 評估](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )。

> 若要讓 Windows 容器功能正常運作, 需要進行重要更新。 請先安裝所有更新，再循序完成本教學課程。

若要在 Azure 部屬，可使用此[範本](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)簡化流程。

<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="install-docker"></a>安裝 Docker

若要安裝 Docker, 我們將使用[OneGet 提供者 PowerShell 模組](https://github.com/oneget/oneget), 搭配提供者執行安裝--在此情況下是[MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider)。 提供者可在您的電腦上啟用容器功能。 您同時會安裝 Docker，並需要重新開機。 需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。

開啟提高權限的 PowerShell 工作階段，並執行下列命令。

首先，從 [PowerShell 資源庫](https://www.powershellgallery.com/packages/DockerMsftProvider)安裝 Docker-Microsoft PackageManagement 提供者。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

接著，您使用 PackageManagement PowerShell 模組安裝最新版 Docker。

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

當 PowerShell 詢問是否要信任封裝來源 'DockerDefault' 時，輸入 `A` 以繼續安裝。 安裝完成時，請重新啟動電腦。

```powershell
Restart-Computer -Force
```

> [!TIP]
> 如果您想要稍後更新 Docker:
>  - 若要查看已安裝的版本，請使用 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 若要尋找最新版本，請使用 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 準備就緒後，請使用 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` 進行升級，後面接著 `Start-Service Docker`

## <a name="install-windows-updates"></a>安裝 Windows Updates

請確定您的 Windows Server 系統為最新狀態，方法是執行︰

```console
sconfig
```

這會顯示文字型的設定功能表，您可從中選擇選項 6，以下載並安裝更新︰

```console
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

當出現提示時，請選擇選項 A 下載所有的更新。

## <a name="deploy-your-first-container"></a>部署您的第一個容器

在這項練習中，您將從 Docker Hub 登錄下載預先建立的 .NET 範例映像，並部署執行 .Net Hello World 應用程式的簡單容器。  

使用 `docker run` 部署 .Net 容器。 這也會下載容器映像，並可能花費幾分鐘的時間。 視您的 Windows Server 主機版本而定, 請執行下列命令。

#### <a name="windows-server-2019"></a>Windows Server 2019

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-1809
```

#### <a name="windows-server-2016"></a>WindowsServer 2016

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-sac2016
```

容器隨即啟動並列印 hello world 訊息，然後結束。

```console
         Hello from .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....


**Environment**
Platform: .NET Core
OS: Microsoft Windows 10.0.17763
```

如需 Docker Run 命令的深入資訊，請參閱 [Docker.com 上的 Docker Run Reference](https://docs.docker.com/engine/reference/run/)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [瞭解如何自動化容器組建及儲存影像](./quick-start-images.md)
