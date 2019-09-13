---
title: Dockerfile 與 Windows 容器
description: 建立 Windows 容器 的 Dockerfiles。
keywords: docker, 容器
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 9fef74c029dc3efc220b1f9924d2695cdbaa61be
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129298"
---
# <a name="dockerfile-on-windows"></a>Windows 上的 Dockerfile

Docker 引擎包括可自動建立容器影像的工具。 雖然您可以透過執行`docker commit`命令來手動建立容器影像，但採用自動影像建立程式有許多優點，包括：

- 將容器映像儲存為程式碼。
- 快速、精確地為維護和升級等用途重新建立容器映像。
- 容器映像和開發週期之間的持續整合。

驅動這項自動化的 Docker 元件是 Dockerfile 和 `docker build` 命令。

Dockerfile 是文字檔，其中包含建立新容器影像所需的指示。 這些指令包含將用作基底的現有映像識別碼、在映像建立程序中執行的命令，以及部署容器映像新執行個體時所執行的命令。

Docker 建立是使用 Dockerfile 的 Docker 引擎命令，並觸發影像建立程式。

本主題將示範如何在 Windows 容器中使用 Dockerfiles，瞭解其基本語法，以及最常見的 Dockerfile 指示。

本檔將討論容器影像和容器影像圖層的概念。 如果您想要深入瞭解影像和影像層疊，請參閱[容器基本影像](../manage-containers/container-base-images.md)。

如需 Dockerfiles 的完整外觀，請參閱[Dockerfile 參考](https://docs.docker.com/engine/reference/builder/)。

## <a name="basic-syntax"></a>基本語法

Dockerfile 在最基本的形式中可以極度簡易。 下列範例會建立新的映像，其中包括 IIS 和 ‘hello world’ 站台。 這個範例包含會說明每個步驟的註解 (以 `#` 表示)。 這篇文章的後續章節將會更詳細地介紹 Dockerfile 語法規則和 Dockerfile 指令。

>[!NOTE]
>必須建立不含副檔名的 Dockerfile。 若要在 Windows 中執行這項作業，請使用您的選擇編輯器來建立檔案，然後將它儲存為 "Dockerfile" （包括引號）。

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

如需 Windows 版 Dockerfiles 的其他範例，請參閱[windows 知識庫 Dockerfile](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)。

## <a name="instructions"></a>指示

Dockerfile 指示會將 Docker 引擎提供給它建立容器影像所需的指示。 這些指示是依序執行一次。 下列範例是 Dockerfiles 中最常使用的指示。 如需 Dockerfile 指示的完整清單，請參閱[Dockerfile 參考](https://docs.docker.com/engine/reference/builder/)。

### <a name="from"></a>FROM

`FROM` 指令會設定新映像建立程序期間所使用的容器映像。 比方說，在使用指令 `FROM mcr.microsoft.com/windows/servercore` 時，所產生的映像衍生自 (而且會相依於) Windows Server Core 基本 OS 映像。 如果指定的映像不存在於正在執行 Docker 建置流程的系統上，Docker 引擎會嘗試從公用或私用的映像登錄下載映像。

FROM 指令的格式如下所示：

```dockerfile
FROM <image>
```

以下是 [寄件者] 命令的範例：

若要從 Microsoft 容器 Registry （MCR）下載 ltsc2019 版本的 windows server core：
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

如需詳細資訊，請參閱[FROM 參考](https://docs.docker.com/engine/reference/builder/#from)。

### <a name="run"></a>RUN

`RUN` 指令指定要執行並擷取至新容器映像的命令。 這些命令可以包含安裝軟體、建立檔案和目錄，以及建立環境設定等項目。

RUN 指令會如下所示：

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec 與 shell 格式之間的差異在於`RUN`指令的執行方式。 使用 Exec 形式時會明確執行指定的程式。

以下是 exec 形式的範例：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

產生的影像會執行`powershell New-Item c:/test`命令：

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

相反地，下列範例會在 shell 格式中執行相同的操作：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

產生的`cmd /S /C powershell New-Item c:\test`圖像有執行指示。

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>在 Windows 上使用 [執行] 的考慮

在 Windows 中，當使用 exec 格式的 `RUN` 指令時，反斜線必須逸出。

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

當目的程式是 Windows 安裝程式時，您必須透過`/x:<directory>`標誌來解壓縮安裝程式，才能啟動實際（緘默）安裝程式。 您也必須等到命令結束之後，才能執行任何其他動作。 否則，程式將會提前結束，但不會安裝任何內容。 如需詳細資訊，請參閱下面的範例。

#### <a name="examples-of-using-run-with-windows"></a>在 Windows 上使用 [執行] 的範例

下列範例 Dockerfile 使用 DISM 在容器影像中安裝 IIS：

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

此範例會安裝 Visual Studio 可轉散發套件。 `Start-Process` 而且該`-Wait`參數是用來執行安裝程式。 這可確保安裝完成後，才能移至 Dockerfile 中的下一個指示。

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

如需 RUN 指令的詳細資訊，請參閱[執行參照](https://docs.docker.com/engine/reference/builder/#run)。

### <a name="copy"></a>複製

此`COPY`指示會將檔案和目錄複寫到容器的檔案系統中。 檔案和目錄必須位於相對於 Dockerfile 的路徑中。

`COPY`指示的格式如下所示：

```dockerfile
COPY <source> <destination>
```

如果來源或目的地包含空格，請以方括弧和雙引號括住路徑，如下列範例所示：

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>在 Windows 中使用 [複製] 的考慮

在 Windows 中，目的格式必須使用正斜線。 例如，以下是有效`COPY`的指示：

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

同時，下列格式的反斜線將無法使用：

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>在 Windows 中使用 [複製] 的範例

下列範例會將來原始目錄的內容新增到容器影像中指定`sqllite`的目錄：

```dockerfile
COPY source /sqlite/
```

下列範例會將所有以 config 開頭的檔案新增至容器`c:\temp`影像的目錄：

```dockerfile
COPY config* c:/temp/
```

如需有關`COPY`指令的詳細資訊，請參閱[複製參照](https://docs.docker.com/engine/reference/builder/#copy)。

### <a name="add"></a>ADD

ADD 指令就像是 COPY 指令，但有更多的功能。 除了將檔案從主機複製到容器映像，`ADD` 指令也可以從具有 URL 規格的遠端位置複製檔案。

`ADD`指示的格式如下所示：

```dockerfile
ADD <source> <destination>
```

如果來源或目的地都包含空白，請以方括弧和雙引號括住路徑：

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>在 Windows 上執行 [新增] 的考慮

在 Windows 中，目的格式必須使用正斜線。 例如，以下是有效`ADD`的指示：

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

同時，下列格式的反斜線將無法使用：

```dockerfile
ADD test1.txt c:\temp\
```

此外，`ADD` 指令在 Linux 上會於複製時展開壓縮的封裝。 Windows 不提供這項功能。

#### <a name="examples-of-using-add-with-windows"></a>在 Windows 中使用 [新增] 的範例

下列範例會將來原始目錄的內容新增到容器影像中指定`sqllite`的目錄：

```dockerfile
ADD source /sqlite/
```

下列範例會將所有以 "config" 開頭的檔案，新增至`c:\temp`容器影像的目錄。

```dockerfile
ADD config* c:/temp/
```

下列範例會將適用于 Windows 的 Python 下載`c:\temp`到容器影像的目錄中。

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

如需有關`ADD`指令的詳細資訊，請參閱[ADD 引用](https://docs.docker.com/engine/reference/builder/#add)。

### <a name="workdir"></a>WORKDIR

`WORKDIR` 指令會為其他 Dockerfile 指令設定工作目錄，例如 `RUN`、`CMD`，也會為執行中的容器映像執行個體設定工作目錄。

`WORKDIR`指示的格式如下所示：

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>在 Windows 上使用 WORKDIR 的考慮

在 Windows 中，如果工作目錄包含一個反斜線，則必須逸出。

```dockerfile
WORKDIR c:\\windows
```

**範例**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

如需`WORKDIR`指示的詳細資訊，請參閱[WORKDIR 參考](https://docs.docker.com/engine/reference/builder/#workdir)。

### <a name="cmd"></a>CMD

`CMD` 指令會設定部署容器映像執行個體時要執行的預設命令。 例如，如果容器將是託管 NGINX 的網頁伺服器，則`CMD`可能會包含使用等`nginx.exe`命令來啟動 web 伺服器的指示。 如果 Dockerfile 中指定了多個 `CMD` 指令，則只會評估最後的指令。

`CMD`指示的格式如下所示：

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>在 Windows 上使用 CMD 的考慮

在 Windows 中，`CMD` 指令中指定的檔案路徑必須使用正斜線或具有逸出的反斜線 `\\`。 下列是有效`CMD`的指示：

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

不過，不含適當斜線的下列格式將無法使用：

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

如需有關`CMD`指令的詳細資訊，請參閱[CMD 參考](https://docs.docker.com/engine/reference/builder/#cmd)。

## <a name="escape-character"></a>逸出字元

在許多情況下，Dockerfile 指令將需要跨多個線條。 若要這樣做，您可以使用逸出字元。 預設的 Dockerfile 逸出字元為反斜線 `\`。 不過，由於反斜線也是 Windows 中的檔案路徑分隔符號，所以使用它來跨多條線路可能會造成問題。 若要解決這個情況，您可以使用 parser 指令來變更預設的逸出字元。 如需有關 parser 指令的詳細資訊，請參閱[parser 指令](https://docs.docker.com/engine/reference/builder/#parser-directives)。

下列範例顯示使用預設逸出字元跨多個線條的單一執行指令：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

若要修改此逸出字元，可以將逸出剖析器指示詞置於 Dockerfile 的第一行。 下列範例會顯示這種情況。

>[!NOTE]
>只有兩個值可以用來做為轉義`\`字元`` ` ``：與。

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

如需有關轉義器指令的詳細資訊，請參閱[轉義器指令](https://docs.docker.com/engine/reference/builder/#escape)。

## <a name="powershell-in-dockerfile"></a>Dockerfile 中的 PowerShell

### <a name="powershell-cmdlets"></a>PowerShell Cmdlet

您可以在 Dockerfile 中使用此`RUN`操作執行 PowerShell Cmdlet。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST 通話

從 web `Invoke-WebRequest`服務收集資訊或檔案時，PowerShell 的 Cmdlet 可能很有用。 例如，如果您建立包含 Python 的影像，您可以將設定`$ProgressPreference`為`SilentlyContinue`以獲得更快速的下載，如下列範例所示。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` 也可以在 Nano Server 中使用。

您也可以選擇使用 .NET WebClient 程式庫，在映像建立程序期間使用 PowerShell 下載檔案。 這樣會增進下載效能。 下列範例會使用 WebClient 媒體櫃下載 Python 軟體。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server 目前不支援 WebClient。

### <a name="powershell-scripts"></a>PowerShell 指令碼

在某些情況下，將腳本複製到您在影像建立過程中所使用的容器中可能會有説明，然後從容器中執行腳本。

>[!NOTE]
>這會限制任何影像圖層快取，並減少 Dockerfile 的可讀性。

此範例使用 `ADD` 指令，從組建電腦中將指令碼複製到容器中。 接著使用 RUN 指令來執行此指令碼。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker 建立

在 Dockerfile 已建立並儲存至磁片之後，您可以執行`docker build`以建立新的影像。 `docker build` 命令會使用幾個選擇性參數和通往 Dockerfile 的路徑。 如需 Docker 建立的完整檔，包括所有組建選項的清單，請參閱[組建參考](https://docs.docker.com/engine/reference/commandline/build/#build)。

`docker build`命令的格式如下所示：

```dockerfile
docker build [OPTIONS] PATH
```

例如，下列命令會建立名為 "iis" 的影像。

```dockerfile
docker build -t iis .
```

啟動組建程式後，輸出會指示狀態並傳回任何拋出的錯誤。

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
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

結果就是新的容器影像，在這個範例中稱為「iis」。

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>進一步的閱讀與參考

- [優化 Windows 版 Dockerfiles 和 Docker 組建](optimize-windows-dockerfile.md)
- [Dockerfile 參考](https://docs.docker.com/engine/reference/builder/)
