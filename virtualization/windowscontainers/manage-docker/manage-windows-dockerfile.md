---
title: "Dockerfile 與 Windows 容器"
description: "建立 Windows 容器 的 Dockerfiles。"
keywords: "docker, 容器"
author: PatrickLang
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 8c5e89cd3afcb109fd3eda2da7bcd1b2c7f48b88
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/21/2017
---
# Windows 上的 Dockerfile

Docker 引擎會包含自動建立容器映像的工具。 雖然可以使用 `docker commit` 命令手動建立容器映像，然而採用自動化映像建立程序提供許多優點，包括：

- 將容器映像儲存為程式碼。
- 快速、精確地為維護和升級等用途重新建立容器映像。
- 容器映像和開發週期之間的持續整合。

驅動這項自動化的 Docker 元件是 Dockerfile 和 `docker build` 命令。

- **Dockerfile** - 文字檔案，包含建立新容器映像所需的指令。 這些指令包含將用作基底的現有映像識別碼、在映像建立程序中執行的命令，以及部署容器映像新執行個體時所執行的命令。
- **Docker 建置** - Docker 引擎命令，其使用 Dockerfile 並觸發映像建立程序。

這份文件將介紹如何使用 Dockerfile 與 Windows 容器、討論語法並詳細說明常用的 Dockerfile 指令。

在這整份文件中，將會討論容器映像和容器映像層的概念。 如需映像和映像層的詳細資訊，請參閱[映像快速入門指南](../quick-start/quick-start-images.md)。

如需 Dockerfile 的完整概觀，請參閱 [Dockerfile reference at docker.com]( https://docs.docker.com/engine/reference/builder/) (docker.com 上的 Dockerfile 參考)。

## Dockerfile 簡介

### 基本語法

Dockerfile 在最基本的形式中可以極度簡易。 下列範例會建立新的映像，其中包括 IIS 和 ‘hello world’ 站台。 這個範例包含會說明每個步驟的註解 (以 `#` 表示)。 這篇文章的後續章節將會更詳細地介紹 Dockerfile 語法規則和 Dockerfile 指令。

> 請注意，建立的 Dockerfile 不能有副檔名。 在 Windows 中要達到這項要求，您可以使用自選的編輯器建立檔案，然後使用加上引號的 "Dockerfile" 標記法儲存該檔案。

```none
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
MAINTAINER jshelton@contoso.com

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

如需適用於 Windows 的 Dockerfiles 其他範例，請參閱 \[Dockerfile for Windows Repository (適用於 Windows 存放庫的 Dockerfile)\] (https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)。

## 指示

Dockerfile 指令為 Docker 引擎提供建立容器映像所需的步驟。 這些指令會依序逐一執行。 以下是一些基本 Dockerfile 指令的詳細資料。 如需 Dockerfile 指令的完整清單，請參閱 \[Dockerfile Reference on Docker.com (Docker.com 上的 Dockerfile 參考)\] (https://docs.docker.com/engine/reference/builder/)。

### FROM

`FROM` 指令會設定新映像建立程序期間所使用的容器映像。 比方說，在使用指令 `FROM microsoft/windowsservercore` 時，所產生的映像衍生自 (而且會相依於) Windows Server Core 基本 OS 映像。 如果指定的映像不存在於正在執行 Docker 建置流程的系統上，Docker 引擎會嘗試從公用或私用的映像登錄下載映像。

**格式**

FROM 指令可接受的格式為：

```
FROM <image>
```

**範例**

```
FROM microsoft/windowsservercore
```

如需 FROM 指令的詳細資訊，請參閱 [FROM Reference on Docker.com]( https://docs.docker.com/engine/reference/builder/#from) (Docker.com 上的 FROM 參考)。

### RUN

`RUN` 指令指定要執行並擷取至新容器映像的命令。 這些命令可以包含安裝軟體、建立檔案和目錄，以及建立環境設定等項目。

**格式**

RUN 指令可接受的格式為：

```none
# exec form

RUN ["<executable", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec 和 Shell 形式的差別在於 `RUN` 指令的執行方式。 使用 Exec 形式時會明確執行指定的程式。

以下範例使用 Exec 形式。

```none
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

檢查產生出的映像，所執行的命令為 `powershell New-Item c:/test`。

```none
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

做為對比，下列範例會執行相同的作業，但會使用殼層表單。

```none
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

這會導致 `cmd /S /C powershell New-Item c:\test` 的執行指令。

```none
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

**Windows 考量**

在 Windows 中，當使用 exec 格式的 `RUN` 指令時，反斜線必須逸出。

```none
RUN ["powershell", "New-Item", "c:\\test"]
```

若目標程式為 Windows Installer，則需另外透過 `/x:<directory>` 旗標擷取安裝程式，才能啟動實際 (無訊息) 安裝程序。 除此之外，還必須等候命令結束。 否則，處理程序會提前結束，而不會安裝任何項目。 如需詳細資訊，請參閱下面的範例。

**範例**

此範例使用 DISM 在容器映像中安裝 IIS。
```none
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

此範例會安裝 Visual Studio 可轉散發套件。 注意：`Start-Process` 及 `-Wait` 參數的用途在於執行安裝程式。 這可確保安裝確實完成之後，才會繼續執行 Docerkfile 中的下一個步驟。

```none
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

如需 RUN 指令的詳細資訊，請參閱 [RUN Reference on Docker.com]( https://docs.docker.com/engine/reference/builder/#run) (Docker.com 上的 RUN 參考)。

### 複製

`COPY` 指令會將檔案和目錄複製到容器的檔案系統。 檔案和目錄必須位在相對於 Dockerfile 的路徑。

**格式**

`COPY` 指令可接受的格式為：

```none
COPY <source> <destination>
```

如果來源或目的地其中一項包含空白字元，請在路徑前後方加上方括弧和雙引號。

```none
COPY ["<source>", "<destination>"]
```

**Windows 考量**

在 Windows 中，目的格式必須使用正斜線。 比方說，這些是有效的 `COPY` 指令。

```none
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

不過，以下所舉的項目將無法運作。

```none
COPY test1.txt c:\temp\
```

**範例**

本範例會將來源目錄的內容加入容器映像中名為 `sqllite` 的目錄。
```none
COPY source /sqlite/
```

這個範例會將以 config 開頭的所有檔案加入容器映像的 `c:\temp` 目錄中。
```none
COPY config* c:/temp/
```

如需 `COPY` 指令的詳細資訊，請參閱 [Docker.com 上的 COPY 參考]( https://docs.docker.com/engine/reference/builder/#copy)。

### 新增

ADD 指令與 COPY 指令非常類似，但是前者包含其他功能。 除了將檔案從主機複製到容器映像，`ADD` 指令也可以從具有 URL 規格的遠端位置複製檔案。

**格式**

`ADD` 指令可接受的格式為：

```none
ADD <source> <destination>
```

如果來源或目的地其中一項包含空白字元，請在路徑前後方加上方括弧和雙引號。

```none
ADD ["<source>", "<destination>"]
```

**Windows 考量**

在 Windows 中，目的格式必須使用正斜線。 比方說，這些是有效的 `ADD` 指令。

```none
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

不過，以下所舉的項目將無法運作。

```none
ADD test1.txt c:\temp\
```

此外，`ADD` 指令在 Linux 上會於複製時展開壓縮的封裝。 Windows 不提供這項功能。

**範例**

本範例會將來源目錄的內容加入容器映像中名為 `sqllite` 的目錄。
```none
ADD source /sqlite/
```

這個範例會將以 config 開頭的所有檔案加入容器映像的 `c:\temp` 目錄中。
```none
ADD config* c:/temp/
```

本範例會將適用於 Windows 的 Python 下載至容器映像的 `c:\temp` 目錄。
```none
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

如需 `ADD` 指令的詳細資訊，請參閱 [ADD Reference on Docker.com]( https://docs.docker.com/engine/reference/builder/#add) (Docker.com 上的 ADD 參考)。

### WORKDIR

`WORKDIR` 指令會為其他 Dockerfile 指令設定工作目錄，例如 `RUN`、`CMD`，也會為執行中的容器映像執行個體設定工作目錄。

**格式**

`WORKDIR` 指令可接受的格式為：

```none
WORKDIR <path to working directory>
```

**Windows 考量**

在 Windows 中，如果工作目錄包含一個反斜線，則必須逸出。

```none
WORKDIR c:\\windows
```

**範例**

```none
WORKDIR c:\\Apache24\\bin
```

如需 `WORKDIR` 指令的詳細資訊，請參閱 [WORKDIR Reference on Docker.com]( https://docs.docker.com/engine/reference/builder/#workdir) (Docker.com 上的 WORKDIR 參考)。

### CMD

`CMD` 指令會設定部署容器映像執行個體時要執行的預設命令。 例如，如果容器會裝載 NGINX 網頁伺服器，則 `CMD` 可能包含啟動網頁伺服器的指令，如 `nginx.exe`。 如果 Dockerfile 中指定了多個 `CMD` 指令，則只會評估最後的指令。

**格式**

`CMD` 指令可接受的格式為：

```none
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

**Windows 考量**

在 Windows 中，`CMD` 指令中指定的檔案路徑必須使用正斜線或具有逸出的反斜線 `\\`。 比方說，這些是有效的 `CMD` 指令。

```none
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```
不過，以下所舉的項目將無法運作。

```none
CMD c:\Apache24\bin\httpd.exe -w
```

如需 `CMD` 指令的詳細資訊，請參閱 [CMD Reference on Docker.com](https://docs.docker.com/engine/reference/builder/#cmd) (Docker.com 上的 CMD 參考)。

## 逸出字元

在許多情況下，Dockerfile 指示必須跨多行列出，而這全都仰賴逸出字元。 預設的 Dockerfile 逸出字元為反斜線 `\`。 因為反斜線也是 Windows 的檔案路徑分隔符號，所以可能會造成問題。 若要變更預設的逸出字元，可以使用剖析器指示詞。 如需剖析器指示詞的詳細資訊，請參閱 [Docker.com 上的剖析器指示詞](https://docs.docker.com/engine/reference/builder/#parser-directives)。

下列範例顯示單一個 RUN 指示如何使用預設的逸出字元跨多行列出。

```none
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

若要修改此逸出字元，可以將逸出剖析器指示詞置於 Dockerfile 的第一行。 如下列範例所示。

> 請注意，只有 `\` 及 `` ` `` 兩個值可以用為逸出字元。

```none
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

如需逸出的剖析器指示詞的詳細資訊，請參閱 [Docker.com 上的逸出剖析器指示詞](https://docs.docker.com/engine/reference/builder/#escape)。

## Dockerfile 中的 PowerShell

### PowerShell 命令

可以使用 `RUN` 作業在 Dockerfile 中執行 PowerShell 命令。

```none
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### REST 呼叫

從 Web 服務收集資訊或檔案時，PowerShell 和 `Invoke-WebRequest` 命令可能會有幫助。 比方說，如果建置包含 Python 的映像，就可以使用下列的範例。 請考慮將 `$ProgressPreference` 設定成 `SilentlyContinue` 以促使更快速的下載。

```none
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> Invoke-WebRequest 在 Nano Server 中也能運作

您也可以選擇使用 .NET WebClient 程式庫，在映像建立程序期間使用 PowerShell 下載檔案。 這樣會增進下載效能。 下列範例會使用 WebClient 媒體櫃下載 Python 軟體。

```none
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> 目前 Nano Server 不支援 Invoke-WebRequest

### PowerShell 指令碼

在某些情況下，在映像建立程序期間將指令碼複製到使用的容器，然後從容器中執行可能會有幫助。 請注意 - 這會限制任何映像層快取，並降低 Dockerfile 的可讀性。

此範例使用 `ADD` 指令，從組建電腦中將指令碼複製到容器中。 接著使用 RUN 指令來執行此指令碼。

```
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## Docker 建置

一旦建立了 Dockerfile 並儲存到磁碟後，就可以執行 `docker build` 建立新的映像。 `docker build` 命令會使用幾個選擇性參數和通往 Dockerfile 的路徑。 如需 Docker 組建的完整文件，包括所有版本選項的清單，請參閱 [Docker.com 上的組建參考](https://docs.docker.com/engine/reference/commandline/build/#build)。

```none
Docker build [OPTIONS] PATH
```
例如，下列命令會建立名為 ‘iis’ 的映像。

```none
docker build -t iis .
```

起始建置流程時，輸出會指出狀態並傳回任何擲回的錯誤。

```none
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM micrsoft/windowsservercore
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

結果是產生新的容器映像，在此範例中名為 'iis'。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## 進一步閱讀與參考

\[將 Dockerfiles 和適用於 Windows 的 Docker 建置最佳化\] (optimize-windows-dockerfile.md)

[Dockerfile Reference on Docker.com (Docker.com 上的 Dockerfile 參考)](https://docs.docker.com/engine/reference/builder/)
