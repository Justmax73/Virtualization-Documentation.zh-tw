---
title: "Windows Server 上的 Windows 容器"
description: "容器部署快速入門"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: ac962391cd3b82be2dd18b145ee5e6d7a483a91a
ms.openlocfilehash: 334f19fa645ad50eb59ad61890842f0b6a43dce2

---

# Windows Server 上的 Windows 容器

本練習將引導進行 Windows Server 上的 Windows 容器功能基本部署和使用。 完成之後，您將會安裝容器角色，並部署簡單的 Windows Server 容器。 開始本快速入門之前，請先熟悉基本的容器概念與術語。 這項資訊可在[快快速入門簡介](./quick_start.md)中找到。

本快速入門是針對 Windows Server 2016 上的 Windows Server 容器。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

**必要條件：**

執行 Windows Server 2016 的電腦系統 (實體或虛擬)。 如果您使用的是 Windows Server 2016 TP5，請更新為 [Window Server 2016 評估版](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 )。 

> Windows 容器功能需要重大更新才能運作。 請先安裝所有更新，再循序完成本教學課程。

## 1.安裝容器功能

容器功能必須先啟用，才能使用 Windows 容器。 若要這麼做，請在提升權限的 PowerShell 工作階段中執行下列命令。

```none
Install-WindowsFeature containers
```

功能安裝完成時，請重新啟動電腦。

```none
Restart-Computer -Force
```

## 2.安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 針對此練習，兩者都會安裝。

以 ZIP 封存的形式下載 Commercially Supported Docker Engine 候選版及用戶端。

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker-1.12.2.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

將該 zip 封存展開到 Program Files。

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

將 Docker 目錄新增至系統路徑。

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

若要將 Docker 安裝為 Windows 服務，請執行下列命令。

```none
dockerd.exe --register-service
```

安裝之後，就可以啟動服務。

```none
Start-Service docker
```

## 3.部署您的第一個容器

在這項練習中，您將從 Docker Hub 登錄下載預先建立的 .NET 範例映像，並部署執行 .Net Hello World 應用程式的簡單容器。  

使用 `docker run` 部署 .NET 容器。 這也會下載容器映像，並可能花費幾分鐘的時間。

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



<!--HONumber=Oct16_HO2-->


