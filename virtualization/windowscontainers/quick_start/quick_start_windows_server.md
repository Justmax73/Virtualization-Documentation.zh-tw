---
title: "Windows Server 上的 Windows 容器"
description: "容器部署快速入門"
keywords: "docker, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: f0584aecd1148300e4075dd6e2e84e0ff92d50d5
ms.openlocfilehash: aa1af44908d623b43ea0095552d7d95b5a5236ac

---

# Windows Server 上的 Windows 容器

本練習將引導您以基本方式部署及使用 Windows Server 2016 上的 Windows 容器功能。 在本練習中，您會安裝容器角色並部署簡易的 Windows Server 容器。 開始本快速入門之前，請先熟悉基本的容器概念與術語。 您可在[快速入門簡介](./quick_start.md)中找到這項資訊。

本快速入門是針對 Windows Server 2016 上的 Windows Server 容器。 在此頁面左側的目錄中，可以找到其他的快速入門文件及 Windows 10 中的容器。

**必要條件：**

執行 Windows Server 2016 的電腦系統 (實體或虛擬)。 如果您使用的是 Windows Server 2016 TP5，請更新為 [Window Server 2016 評估版](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 )。 

> Windows 容器功能需要重大更新才能運作。 請先安裝所有更新，再循序完成本教學課程。

若要在 Azure 部屬，可使用此[範本](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)簡化流程。<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## 1.安裝 Docker

我們將使用 [OneGet 提供者 PowerShell 模組](https://github.com/oneget/oneget)安裝 Docker。 提供者可讓您的電腦使用容器功能。 您同時會安裝 Docker，並需要重新開機。 需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。

開啟提高權限的 PowerShell 工作階段，並執行下列命令。

首先安裝 OneGet PowerShell 模組。

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

接著使用 OneGet 安裝最新版的 Docker。
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

當 PowerShell 詢問是否要信任封裝來源 'DockerDefault' 時，輸入 A 以繼續安裝。 安裝完成時，請重新啟動電腦。

```none
Restart-Computer -Force
```

## 2.安裝 Windows Updates

請確定您的 Windows Server 系統為最新狀態，方法是執行︰

```none
sconfig
```

這會顯示文字型的設定功能表，您可從中選擇選項 6，以下載並安裝更新︰

```none
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

## 3.部署您的第一個容器

在這項練習中，您將從 Docker Hub 登錄下載預先建立的 .NET 範例映像，並部署執行 .Net Hello World 應用程式的簡單容器。  

使用 `docker run` 部署 .Net 容器。 這也會下載容器映像，並可能花費幾分鐘的時間。

```none
docker run microsoft/sample-dotnet
```

容器隨即啟動並列印 hello world 訊息，然後結束。

```none
       Welcome to .NET Core!
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
```

如需 Docker Run 命令的深入資訊，請參閱 [Docker.com 上的 Docker Run Reference]( https://docs.docker.com/engine/reference/run/)。

## 後續步驟

[Windows Server 上的容器映像](./quick_start_images.md)

[Windows 10 上的 Windows 容器](./quick_start_windows_10.md)



<!--HONumber=Nov16_HO2-->


