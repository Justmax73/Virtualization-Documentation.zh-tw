---
title: "將 Windows Dockerfiles 最佳化"
description: "將 Windows 容器的 Dockerfiles 最佳化。"
keywords: "docker, 容器"
author: PatrickLang
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
translationtype: Human Translation
ms.sourcegitcommit: 54eff4bb74ac9f4dc870d6046654bf918eac9bb5
ms.openlocfilehash: 2077f7cf0428e08ce915470ac4cc3b0ccc9c6369

---
# 將 Windows Dockerfiles 最佳化

有數種方法可用來將 Docker 建置流程及產生的 Docker 映像最佳化。 本文件詳述 Docker 建置流程的運作方式，並示範幾種可用來將映像建立與 Windows 容器最佳化的策略。

## Docker 建置

### 映像層

在檢查 Docker 建置最佳化之前，請務必了解 Docker 建置的運作方式。 Docker 建置流程期間使用了 Dockerfile，且每個可採取動作的指令都依序在其自身的暫存容器中執行。 結果就為每個可採取動作的指令產生了新的映像層。 

看看下列 Dockerfile。 在此範例中，使用 `windowsservercore` 基本 OS 映像並安裝了 IIS，然後建立了一個簡單的網站。

```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

從這個 Dockerfile 中，您可能會以為產生出的映像包含兩個層，一個用於容器 OS 映像，而另一個包含 IIS 和網站，但情況並非如此。 新的映像由多個層建構而成，每一個層相依於先前的層。 若要將此視覺化，可以對新的映像執行 `docker history` 命令。 如此會顯示映像包含四個層，即基底和其他三個圖層，各個皆用於 Docerkfile 中的其中一項指令。

```none
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

其中每個層都可以對應至 Dockerfile 的一項指令。 底層 (此範例中為 `6801d964fda5`) 代表基本 OS 映像。 向上一層可以看到 IIS 安裝。 再上一層則包含新的網站，以此類推。

可以將 Dockerfiles 寫入以將映像層最小化、將建置效能最佳化，同時將外觀方面的項目 (例如可讀性) 最佳化。 最後，有許多方式可以完成相同的映像建置工作。 了解 Dockerfile 的格式會如何影響建置時間和產生出的映像，將有助提升自動化體驗。 

## 將映像大小最佳化

在建置 Docker 容器映像時，映像大小可能會是重要的因素。 容器映像在登錄和主機之間移動、進行匯出和匯入，而且最終會耗用空間。 可以在 Docker 建置流程期間使用幾種策略，將映像大小降到最低。 本節將詳細說明部分專屬於 Windows 容器的策略。 

如需有關 Dockerfile 最佳做法的詳細資訊，請參閱 [Best practices for writing Dockerfiles on Docker.com]( https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) (Docker.com 上寫入 Dockerfiles 的最佳做法)。

### 群組相關的動作

因為每個 `RUN` 指令會在容器映像中建立新的層，將動作分組至一個 `RUN` 指令可以減少層的數目。 雖然將層最小化可能不會對映像大小造成太大的影響，但將相關聯的動作分組卻會有明顯影響，您會在後續的範例看到相關示範。

下列兩個範例將示範相同的作業，這會使容器映像具有相同功能，不過這兩個 Dockerfiles 則是以不同的方式建構。 同時也會對生產出的映像進行比較。  

第一個範例會下載並安裝 Python for Windows，然後移除下載的安裝檔加以清空。 每個動作都會依自己的 `RUN` 指令執行。

```none
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

產生出的映像包含三個額外的層，每個 `RUN` 指令皆有一個層。

```none
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

作為比較，以下是相同的作業，但所有步驟都執行相同的 `RUN` 指令。 請注意，`RUN` 指示中的每個步驟都位於 Dockerfile 上的一個新行，'\' 字元則用於自動換行。 

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

這裡產生出的映像包含一個用於 `RUN` 指令的額外層。

```none
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB                
```

### 移除多餘的檔案

若為使用之後就不需要的檔案 (例如安裝程式)，便可移除該檔案以減少映像的大小。 這需與將檔案複製到映像層的步驟同時進行。 如此可避免將檔案保存至較低層級的映像層中。

此範例下載並執行了 Python 封裝，並在執行完成後將可執行檔移除。 這全在一項 `RUN` 作業期間完成，並產生出單一映像層。

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## 將建置速度最佳化

### 多行

在將 Docker 建置速度最佳化時，將作業分隔成多個獨立的指令可能比較有利。 分成多個 `RUN` 作業會提升快取的效率。 由於獨立層是為各個 `RUN` 指令所建立，因此若已在不同的 Docker 建置作業中執行相同的步驟，便會重複使用此快取作業 (映像層)。 結果會減少該 Docker 建置執行階段。

在下列範例中，會下載並安裝 Apache 和 Visual Studio 轉散發套件，然後再清除不需要的檔案。 全都只需使用一個 `RUN` 指令完成。 如果其中任何一項動作進行更新，則會重新執行所有動作。

```none
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredistexe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force
```

產生出的映像包含兩個層，一個用於基本 OS 映像，而第二個則包含單一 `RUN` 指令的所有作業。

```none
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

作為對比，以下將相同的動作分成三個 `RUN` 指令。 在此情況下，每個 `RUN` 指令會在容器映像層中快取，而只有做出變更的指令需要在後續 Dockerfile 組建上重新執行。

```none
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

產生出的映像包含四個層，一個用於基本 OS 映像，然後其他層分別供給各個 `RUN` 指令使用。 因為各個 `RUN` 指令皆在自己的層中執行，所以此 Dockerfile 的任何後續執行或不同 Dockerfile 中相同一組指示，將會使用快取映像層，從而降低建置時間。 搭配映像快取使用時，指令順序非常重要，如需詳細資料，請參閱本文件的下一章節。

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

### 指令順序

Dockerfile 是從上到下進行處理，每個指令會和快取層進行比較。 當指令沒有快取層時，此指令和所有後續的指令會在新容器映像層中進行處理。 有鑑於此，指令的放置順序非常重要。 在 Dockerfile 上層放置會保持固定的指令。 在 Dockerfile 下層放置可能會有所改變的指令。 這樣做可以降低取消現有快取的可能性。

此範例的目的是示範 Dockerfile 指令順序如何影響快取的效率。 在這個簡易的 Dockerfile 中，會建立四個編號的資料夾。  

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```
產生出的映像具有五個層，一個用於基本 OS 映像，然後其他層分別供給各個 `RUN` 指令使用。

```none
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B    
```

Docker 檔案現在已經過些微修改。 請注意，第三個 `RUN` 指令已變更。 當 Docker 建置針對此 Dockerfile 執行時，前三項指令 (和上個範例中的指令完全相同) 會使用快取映像層。 不過，因為變更的 `RUN` 指令尚未經快取，因此建立了新的層供其本身與所有後續指令使用。

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

將新映像的映像識別碼和上個範例中的映像相比較，您會發現前三個層 (從下至上) 為共用，而第四和第五個層則為唯一。

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## 外觀最佳化

### 指令案例

Dockerfile 指令不區分大小寫，不過依慣例會使用大寫。 如此可區別指令呼叫和指令作業，有助於改善可讀性。 下列的兩個範例會示範這個概念。 

小寫字母：
```none
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```
大寫字母： 
```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### 自動換行

冗長且複雜的作業可以使用反斜線 `\` 字元分隔成多行。 下列 Dockerfile 會安裝 Visual Studio 可轉散發套件、移除安裝程式檔案，然後建立設定檔。 這三項作業都是在單行中指定。

```none
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```
命令可以改寫成由個別的 `RUN` 指令在其本身的行中指定各項作業。 

```none
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## 進一步閱讀與參考

[Windows 上的 Dockerfile] (manage-windows-dockerfile.md)

[Best practices for writing Dockerfiles on Docker.com (Docker.com 上撰寫 Dockerfiles 的最佳做法)](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Jan17_HO4-->


