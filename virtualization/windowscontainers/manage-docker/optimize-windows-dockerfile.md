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
ms.openlocfilehash: ae633c7ba5d9672335addcc582988fc47c13ed79
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910148"
---
# <a name="optimize-windows-dockerfiles"></a>將 Windows Dockerfiles 最佳化

有許多方法可優化 Docker 組建程式和產生的 Docker 映射。 本文說明 Docker build 程式的運作方式，以及如何以最佳方式建立 Windows 容器的映射。

## <a name="image-layers-in-docker-build"></a>Docker 組建中的映射圖層

您必須先瞭解 Docker build 的運作方式，才能將 Docker build 優化。 Docker 建置流程期間使用了 Dockerfile，且每個可採取動作的指令都依序在其自身的暫存容器中執行。 結果就為每個可採取動作的指令產生了新的映像層。

例如，下列範例 Dockerfile 會使用 `mcr.microsoft.com/windows/servercore:ltsc2019` 基本 OS 映射、安裝 IIS，然後建立簡單的網站。

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

您可能會預期此 Dockerfile 會產生包含兩個層的映射，一個用於容器 OS 映射，第二個包含 IIS 和網站。 不過，實際的影像有許多層級，而且每個圖層都相依于其前面。

為了更清楚說明，讓我們對我們的範例 Dockerfile 所進行的映射執行 `docker history` 命令。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

輸出顯示此影像有四個層級：基底層和三個額外的圖層會對應到 Dockerfile 中的每個指令。 底層 (此範例中為 `6801d964fda5`) 代表基本 OS 映像。 其中一層是 IIS 安裝。 再上一層則包含新的網站，以此類推。

您可以撰寫 dockerfile 來最小化映射層、將組建效能優化，以及透過可讀性優化協助工具。 最後，有許多方式可以完成相同的映像建置工作。 瞭解 Dockerfile 的格式如何影響組建時間，以及它所建立的映射可改善自動化體驗。

## <a name="optimize-image-size"></a>優化影像大小

視您的空間需求而定，在建立 Docker 容器映射時，映射大小可能是很重要的因素。 容器映像在登錄和主機之間移動、進行匯出和匯入，而且最終會耗用空間。 本節將告訴您如何在 Windows 容器的 Docker 組建程式期間，將映射大小降至最低。

如需 Dockerfile 最佳做法的詳細資訊，請參閱[在 Docker.com 上撰寫 dockerfile 的最佳作法](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)。

### <a name="group-related-actions"></a>群組相關的動作

由於每個 `RUN` 指令會在容器映射中建立新的圖層，因此將動作分組為一個 `RUN` 指令可以減少 Dockerfile 中的層級數目。 雖然將層最小化可能不會對映像大小造成太大的影響，但將相關聯的動作分組卻會有明顯影響，您會在後續的範例看到相關示範。

在本節中，我們將比較兩個執行相同動作的範例 Dockerfile。 不過，一個 Dockerfile 的每個動作都有一個指示，另一個則是群組在一起的相關動作。

下列未分組的範例 Dockerfile 會下載適用于 Windows 的 Python、安裝它，並在安裝完成後移除下載的安裝程式檔案。 在此 Dockerfile 中，會為每個動作提供自己的 `RUN` 指令。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

第二個範例是執行完全相同作業的 Dockerfile。 不過，所有相關動作都已群組在單一 `RUN` 指令之下。 `RUN` 指令中的每個步驟都位於 Dockerfile 的新行，而 '\\' 字元則用來換行。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

產生的影像只有一個額外的 `RUN` 指令層。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>移除多餘的檔案

如果您的 Dockerfile 中有一個不需要的檔案（例如安裝程式），則您可以將它移除以減少影像大小。 這需與將檔案複製到映像層的步驟同時進行。 這麼做可防止檔案在較低層級的影像層中保存。

在下列範例 Dockerfile 中，會下載、執行 Python 套件，然後將其移除。 這全在一項 `RUN` 作業期間完成，並產生出單一映像層。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>優化組建速度

### <a name="multiple-lines"></a>多行

您可以將作業分割成多個個別的指示，以優化 Docker 組建速度。 多個 `RUN` 作業會增加快取效率，因為每個 `RUN` 指令都會建立個別層。 如果已在不同的 Docker 組建作業中執行相同的指令，則會重複使用此快取作業（映射層），因而減少 Docker 組建執行時間。

在下列範例中，會下載並安裝 Apache 和 Visual Studio 重新發佈套件，然後藉由移除不再需要的檔案來進行清除。 這一切都是透過單一 `RUN` 指令來完成。 如果其中任何一個動作已更新，則會重新執行所有動作。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

產生的映射有兩個層，一個用於基本 OS 映射，另一個則包含來自單一 `RUN` 指令的所有作業。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

藉由比較，以下是分成三個 `RUN` 指令的相同動作。 在此情況下，每個 `RUN` 指令都會在容器映射層中快取，而且只需要在後續 Dockerfile 組建上重新執行已變更的指示。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

產生的映射是由四個圖層所組成;基本 OS 映射的一個層級，以及三個 `RUN` 指示。 因為每個 `RUN` 指令都是在自己的層級中執行，所以在不同的 Dockerfile 中，此 Dockerfile 的任何後續執行或相同的指令集將會使用快取的影像層，以減少組建時間。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

當您使用映射快取時，如何排序指示是很重要的，如下一節所示。

### <a name="ordering-instructions"></a>排序指示

Dockerfile 是從上到下進行處理，每個指令會和快取層進行比較。 當指令沒有快取層時，此指令和所有後續的指令會在新容器映像層中進行處理。 有鑑於此，指令的放置順序非常重要。 在 Dockerfile 上層放置會保持固定的指令。 在 Dockerfile 下層放置可能會有所改變的指令。 這樣做可以降低取消現有快取的可能性。

下列範例會示範 Dockerfile 指令順序如何影響快取的效率。 這個簡單的範例 Dockerfile 有四個編號資料夾。  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

產生的映射具有五個層，一個用於基本 OS 映射，而每個 `RUN` 指令。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

下一個 Dockerfile 現在已稍微修改過，第三個 `RUN` 指令已變更為新的檔案。 當 Docker 建置針對此 Dockerfile 執行時，前三項指令 (和上個範例中的指令完全相同) 會使用快取映像層。 不過，因為變更的 `RUN` 指令不會被快取，所以會針對已變更的指令和所有後續的指示建立新的圖層。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

當您比較新影像的影像識別碼與本節第一個範例中的內容時，您會注意到，從下到上的前三個層是共用的，但第四個和第五個是唯一的。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>表面優化

### <a name="instruction-case"></a>指示案例

Dockerfile 指令不區分大小寫，但慣例是使用大寫。 這會藉由區分指令呼叫和指令作業來改善可讀性。 下列兩個範例會比較 uncapitalized 和大寫的 Dockerfile。

以下是 uncapitalized Dockerfile：

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

以下是使用大寫的相同 Dockerfile：

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>換行

長時間和複雜的作業可以使用反斜線 `\` 字元分隔成多行。 下列 Dockerfile 會安裝 Visual Studio 可轉散發套件、移除安裝程式檔案，然後建立設定檔。 這三項作業都是在單行中指定。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

命令可以使用反斜線來細分，讓一個 `RUN` 指令中的每個作業都在自己的行上指定。

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>進一步閱讀和參考

[Windows 上的 Dockerfile](manage-windows-dockerfile.md)

[在 Docker.com 上撰寫 Dockerfile 的最佳做法](https://docs.docker.com/engine/reference/builder/)
