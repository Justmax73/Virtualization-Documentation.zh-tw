---
title: 將 Windows Dockerfiles 最佳化
description: 將 Windows 容器的 Dockerfiles 最佳化。
keywords: docker, 容器
author: cwilhit
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: d897560061fae23fda6f88ebdad6dd804da9a8f1
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610338"
---
# <a name="optimize-windows-dockerfiles"></a>將 Windows Dockerfiles 最佳化

有許多方式可以將 Docker 建置流程及產生的 Docker 映像最佳化。 這篇文章說明 Docker 建置流程的運作方式，以及如何最佳方式建立適用於 Windows 容器映像。

## <a name="image-layers-in-docker-build"></a>在 Docker 組建的映像層

您可以將您的 Docker 建置最佳化之前，您將需要知道如何 Docker 建置的運作方式。 Docker 建置流程期間使用了 Dockerfile，且每個可採取動作的指令都依序在其自身的暫存容器中執行。 結果就為每個可採取動作的指令產生了新的映像層。

例如，以下範例 Dockerfile 使用`windowsservercore`基本 OS 映像，安裝 IIS，並接著會建立一個簡單的網站。

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

您可能會預期這個 Dockerfile 會產生兩個層，一個用於容器 OS 映像，並第二個，其中包括 IIS 和網站的影像。 不過，實際的映像有許多層級，而且每個層級取決於它前一。

若要讓這更加清楚，讓我們執行`docker history`命令針對映像我們所做的 Dockerfile 的範例。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

輸出會顯示我們此映像具有四個層級： 基本層級，並會對應至 Dockerfile 中每個指令的三個額外層。 底層 (此範例中為 `6801d964fda5`) 代表基本 OS 映像。 向上一層是 IIS 安裝。 再上一層則包含新的網站，以此類推。

可以將 Dockerfiles 寫入以映像層最小化、 將建置效能最佳化透過可讀性的協助工具。 最後，有許多方式可以完成相同的映像建置工作。 了解如何在 Dockerfile 的格式會影響建置時間和它會建立映像提升自動化體驗。

## <a name="optimize-image-size"></a>將映像大小最佳化

根據您的空間需求，映像大小可以是重要的因素，在建置 Docker 容器映像。 容器映像在登錄和主機之間移動、進行匯出和匯入，而且最終會耗用空間。 本章節會告訴您如何在 Windows 容器的 Docker 建置流程期間，映像大小降到最低。

如需有關 Dockerfile 最佳做法的詳細資訊，請參閱[撰寫 Docker.com 上的 Dockerfiles 的最佳做法](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)。

### <a name="group-related-actions"></a>群組相關的動作

因為每個`RUN`指令會建立新的層中的容器映像，將動作分組至一個`RUN`指令可以減少層 Dockerfile 中的數目。 雖然將層最小化可能不會對映像大小造成太大的影響，但將相關聯的動作分組卻會有明顯影響，您會在後續的範例看到相關示範。

在此區段中，我們會比較兩個範例執行相同的動作的 Dockerfiles。 不過，一個 Dockerfile 都有一個指令，每個動作，而另有其相關的動作，群組在一起。

下列群組的範例 Dockerfile 會用於 Windows 的 Python 下載、 安裝它，並完成安裝後會移除下載的安裝檔。 在這個 Dockerfile 中，每個動作會獲得自己`RUN`指令。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

產生出的映像包含三個額外的層，每個 `RUN` 指令皆有一個層。

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

第二個範例是 Dockerfile 執行相同的作業。 不過，所有相關的動作有分組單一`RUN`指令。 每個步驟`RUN`指令是在 Dockerfile 新的一行，雖然 ' \\' 字元用於自動換行。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

產生的映像具有只有一個額外的層級的`RUN`指令。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>移除多餘的檔案

是否已使用的是您 Dockerfile，例如安裝程式，您不需要在其之後的檔案，您可以移除它，以減少映像大小。 這需與將檔案複製到映像層的步驟同時進行。 這樣可避免將檔案保存至較低層級的映像層。

在下列範例 Dockerfile 中，Python 套件是下載，執行，然後移除。 這全在一項 `RUN` 作業期間完成，並產生出單一映像層。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>將建置速度最佳化

### <a name="multiple-lines"></a>多行

您可以分割成多個獨立的指令，若要將 Docker 建置速度最佳化的作業。 多個`RUN`作業會提升快取的效率，因為個別的層級會建立每個`RUN`指令。 如果在不同的 Docker 建置作業已經執行相同的指令，此快取的作業 （映像層） 則會重複使用，導致降低 Docker 建置執行階段。

在下列範例中，Apache 和 Visual Studio 轉散發套件下載、 安裝，並再清除藉由移除不再需要的檔案。 這所有完成具單一`RUN`指令。 如果任何一項動作進行更新，將會重新執行所有動作。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

產生的映像具有兩個層，一個用於基本 OS 映像，另一個包含所有的作業從單一`RUN`指令。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

透過比較，以下是相同動作分割成三個`RUN`指示。 在此情況下，每個`RUN`指令會快取在容器映像層中，與只有做出變更重新執行後續 Dockerfile 上需要有組建。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

產生的映像包含四個層級;適用於基本 OS 映像和每個三種一層`RUN`指示。 因為每個`RUN`指令執行自己的層中，此 Dockerfile 的任何後續執行或相同一組不同的 Dockerfile 中的指示，則會使用快取映像層，降低建置時間。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

順序指示的如何重要的是當使用影像的快取，因為您會看到下一節。

### <a name="ordering-instructions"></a>指令順序

Dockerfile 是從上到下進行處理，每個指令會和快取層進行比較。 當指令沒有快取層時，此指令和所有後續的指令會在新容器映像層中進行處理。 有鑑於此，指令的放置順序非常重要。 在 Dockerfile 上層放置會保持固定的指令。 在 Dockerfile 下層放置可能會有所改變的指令。 這樣做可以降低取消現有快取的可能性。

下列範例顯示 Dockerfile 指令順序如何影響快取的效率。 此簡單範例 Dockerfile 有四個已編號的資料夾。  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

產生的映像具有五個層，一個用於基本 OS 映像和每個`RUN`指示。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

這個下一步 Dockerfile 現在已經過稍微修改，與第三個`RUN`指令已變更為新的檔案。 當 Docker 建置針對此 Dockerfile 執行時，前三項指令 (和上個範例中的指令完全相同) 會使用快取映像層。 不過，因為已變更`RUN`指令不快取，變更的指令和所有後續的指示建立新的層供。

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

當您比較映像識別碼的新的映像的這一節的第一個範例中時，您會注意到前, 三個層從下至上為共用，但第四個和第五個為唯一。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>外觀最佳化

### <a name="instruction-case"></a>指令案例

Dockerfile 指令不區分大小寫，但依慣例會使用大寫。 如此可區別指令呼叫和指令作業改善可讀性。 下列兩個範例會比較 uncapitalized 和大寫的 Dockerfile。

以下是 uncapitalized 的 Dockerfile:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

以下是相同的 Dockerfile 使用大寫：

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>自動換行

冗長且複雜的作業可以由反斜線分隔成多行`\`字元。 下列 Dockerfile 會安裝 Visual Studio 可轉散發套件、移除安裝程式檔案，然後建立設定檔。 這三項作業都是在單行中指定。

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

命令可以因為落後將使用反斜線因此，每個作業`RUN`指令在其本身的行中指定。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>進一步閱讀與參考

[Windows 上的 Dockerfile](manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles on Docker.com (Docker.com 上撰寫 Dockerfiles 的最佳做法)](https://docs.docker.com/engine/reference/builder/)
