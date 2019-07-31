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
ms.openlocfilehash: 056ab87189e8e423df5758be0f622a43b92c9056
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882951"
---
# <a name="optimize-windows-dockerfiles"></a>將 Windows Dockerfiles 最佳化

您可以透過多種方式來優化 Docker 建立程式和產生的 Docker 影像。 本文說明 Docker 組建程式的運作方式, 以及如何以最佳方式建立 Windows 容器的影像。

## <a name="image-layers-in-docker-build"></a>Docker 建立中的圖像圖層

您必須知道 Docker 組建的運作方式, 才能優化您的 Docker 建立。 Docker 建置流程期間使用了 Dockerfile，且每個可採取動作的指令都依序在其自身的暫存容器中執行。 結果就為每個可採取動作的指令產生了新的映像層。

例如, 下列範例 Dockerfile 會使用`windowsservercore`基本 OS 影像、安裝 IIS, 然後建立簡單的網站。

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

您可能會認為這個 Dockerfile 會產生一個含有兩個圖層的影像, 一個用於容器 OS 影像, 另一個則包含 IIS 與網站。 不過, 實際影像有許多層級, 而每個圖層都依賴于它前面的圖層。

若要讓這個更清楚, 請讓`docker history`我們針對我們的範例 Dockerfile 執行命令。

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

[輸出] 會顯示這個影像有四個圖層: 基底圖層和三個其他圖層, 分別對應至 Dockerfile 中的每個指示。 底層 (此範例中為 `6801d964fda5`) 代表基本 OS 映像。 [一層] 是 [IIS 安裝]。 再上一層則包含新的網站，以此類推。

Dockerfiles 可以寫入以最小化影像圖層、優化組建效能, 以及透過可讀性優化協助工具。 最後，有許多方式可以完成相同的映像建置工作。 瞭解 Dockerfile 的格式會如何影響組建時間, 以及它所建立的影像會改善自動化體驗。

## <a name="optimize-image-size"></a>優化影像大小

根據您的空間需求而定, 影像大小在建立 Docker 容器影像時可能是一個重要的因素。 容器映像在登錄和主機之間移動、進行匯出和匯入，而且最終會耗用空間。 本節將說明如何在 Windows 容器的 Docker 建立程式期間將影像大小最小化。

如需有關 Dockerfile 最佳做法的其他資訊, 請參閱[在 Docker.com 上撰寫 Dockerfiles 的最佳做法](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)。

### <a name="group-related-actions"></a>群組相關的動作

因為每`RUN`個指令都會在容器影像中建立新的圖層, 所以`RUN`將動作群組成一個指令可減少 Dockerfile 中的圖層數。 雖然將層最小化可能不會對映像大小造成太大的影響，但將相關聯的動作分組卻會有明顯影響，您會在後續的範例看到相關示範。

在本節中, 我們將比較兩個執行相同專案的範例 Dockerfiles。 不過, 一個 Dockerfile 的每個動作都有一個指示, 而另一個則是將其相關動作群組在一起。

下列取消群組範例 Dockerfile 下載適用于 Windows 的 Python、安裝該檔案, 並在安裝完成後移除已下載的安裝程式檔。 在此 Dockerfile 中, 會為每個動作`RUN`指定自己的指示。

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

第二個範例是執行完全相同的操作的 Dockerfile。 不過, 所有相關動作都已群組在單一`RUN`指示下。 `RUN`指令中的每個步驟都是在 Dockerfile 的新行中, 而 "\ \" 字元則用於自動換行。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

產生的圖像只有一個額外的圖層可`RUN`供指示。

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>移除多餘的檔案

如果您的 Dockerfile (例如安裝程式) 中有您不需要的檔案, 您可以移除該檔案以減少影像大小。 這需與將檔案複製到映像層的步驟同時進行。 如此一來, 就能避免檔案持續在較低層級的影像圖層中。

在下列範例中 Dockerfile, 將會下載並執行並移除 Python 封裝。 這全在一項 `RUN` 作業期間完成，並產生出單一映像層。

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>優化組建速度

### <a name="multiple-lines"></a>多個線條

您可以將作業分割成多個個別的指示, 以優化 Docker 組建速度。 由於`RUN`每個`RUN`指令都會建立個別層, 所以多個作業會提高快取的效能。 如果已在不同的 Docker 建立作業中執行相同的指令, 則會重複使用這個快取的操作 (影像圖層), 導致放大的 Docker 組建執行時間減少。

在下列範例中, Apache 與 Visual Studio 的重新發佈套件都是透過移除不再需要的檔案來下載、安裝及清理。 這只會以單一`RUN`指示完成。 如果更新這些動作中的任何一個, 所有動作將會重新執行。

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

產生的圖像有兩個層, 一個用於基作業系統影像, 另一個包含來自單一`RUN`指令的所有操作。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

依比較, 以下是分為三個`RUN`指示的相同動作。 在這種情況下`RUN` , 每個指令都是在容器影像圖層中緩存, 只需要在後續的 Dockerfile 組建上重新執行。

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

產生的圖像由四個圖層組成;基本 OS 影像的一個圖層, 以及這三`RUN`個指示。 因為每`RUN`個指令都是在自己的層級執行, 所以任何後續在不同的 Dockerfile 中執行此 Dockerfile 或相同的指令集, 都會使用快取的影像圖層, 以減少組建時間。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

使用影像快取時, 您對指示的排序方式很重要, 如下節所示。

### <a name="ordering-instructions"></a>排序指示

Dockerfile 是從上到下進行處理，每個指令會和快取層進行比較。 當指令沒有快取層時，此指令和所有後續的指令會在新容器映像層中進行處理。 有鑑於此，指令的放置順序非常重要。 在 Dockerfile 上層放置會保持固定的指令。 在 Dockerfile 下層放置可能會有所改變的指令。 這樣做可以降低取消現有快取的可能性。

下列範例示範 Dockerfile 指令排序如何影響快取效果。 這個簡單的範例 Dockerfile 有四個編號的資料夾。  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

產生的影像有五個層, 一個用於基作業系統影像和每個`RUN`指示。

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

下一個 Dockerfile 現在已稍加修改, 第三個`RUN`指示變更為新的檔案。 當 Docker 建置針對此 Dockerfile 執行時，前三項指令 (和上個範例中的指令完全相同) 會使用快取映像層。 不過, 因為變更的`RUN`指令不會被快取, 所以會針對變更的指令及所有後續的指示建立新的圖層。

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

當您將新影像的影像識別碼與此區段第一個範例中的圖像識別碼進行比較時, 您會注意到從下到上的前三個層都是共用的, 但第四個和第五個是唯一的。

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>修飾優化

### <a name="instruction-case"></a>指示例

Dockerfile 指令不區分大小寫, 但慣例是使用大寫。 這可讓您在指令通話與指示作業間區分來改善可讀性。 下列兩個範例會比較 uncapitalized 和大寫 Dockerfile。

下列是 uncapitalized Dockerfile:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

下列是使用大小寫的相同 Dockerfile:

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>換行

您可以透過反斜線`\`字元將長且複雜的作業分割成多個線條。 下列 Dockerfile 會安裝 Visual Studio 可轉散發套件、移除安裝程式檔案，然後建立設定檔。 這三項作業都是在單行中指定。

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

您可以使用反斜線來中斷命令, 讓單一`RUN`指令的每個運算都在自己的行上指定。

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>進一步的閱讀與參考

[Windows 上的 Dockerfile](manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles on Docker.com (Docker.com 上撰寫 Dockerfiles 的最佳做法)](https://docs.docker.com/engine/reference/builder/)
