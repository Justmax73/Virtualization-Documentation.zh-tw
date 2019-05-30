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
ms.openlocfilehash: c08fa4d0a89bddeddd0f0a918345c33a6e2ab893
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680988"
---
# <a name="dockerfile-on-windows"></a>Windows 上的 Dockerfile

Docker 引擎會包含自動建立容器映像的工具。 雖然您可以手動建立容器映像，執行`docker commit`命令時，然而採用自動化映像建立程序有許多好處，包括：

- 將容器映像儲存為程式碼。
- 快速、精確地為維護和升級等用途重新建立容器映像。
- 容器映像和開發週期之間的持續整合。

驅動這項自動化的 Docker 元件是 Dockerfile 和 `docker build` 命令。

Dockerfile 是文字檔案，其中包含建立新的容器映像所需的指示。 這些指令包含將用作基底的現有映像識別碼、在映像建立程序中執行的命令，以及部署容器映像新執行個體時所執行的命令。

Docker 建置為 Docker 引擎命令，其使用 Dockerfile 並觸發映像建立程序。

此主題將說明如何使用 Windows 容器的 Dockerfiles、 了解其基本語法，以及最常見的 Dockerfile 指令為何。

本文件會討論容器映像和容器映像層的概念。 如果您想要深入了解映像和映像層，請參閱[映像快速入門指南](../quick-start/quick-start-images.md)。

如需完整 Dockerfiles 查看，請參閱[Dockerfile 參考](https://docs.docker.com/engine/reference/builder/)。

## <a name="basic-syntax"></a>基本語法

Dockerfile 在最基本的形式中可以極度簡易。 下列範例會建立新的映像，其中包括 IIS 和 ‘hello world’ 站台。 這個範例包含會說明每個步驟的註解 (以 `#` 表示)。 這篇文章的後續章節將會更詳細地介紹 Dockerfile 語法規則和 Dockerfile 指令。

>[!NOTE]
>Dockerfile 必須建立，不含副檔名。 若要在 Windows 中執行此動作，使用您選擇的編輯器建立檔案，然後使用 （包括加上引號）"Dockerfile"標記法儲存它。

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

Windows 的 Dockerfiles 其他範例，請參閱 [ [Dockerfile for Windows 存放庫](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples)。

## <a name="instructions"></a>指示

Dockerfile 指令提供 Docker 引擎它必須為其建立容器映像的指示。 這些指令會執行一一和順序。 下列範例是 Dockerfiles 中最常使用的指示。 Dockerfile 指令的完整清單，請參閱[Dockerfile 參考](https://docs.docker.com/engine/reference/builder/)。

### <a name="from"></a>FROM

`FROM` 指令會設定新映像建立程序期間所使用的容器映像。 比方說，在使用指令 `FROM microsoft/windowsservercore` 時，所產生的映像衍生自 (而且會相依於) Windows Server Core 基本 OS 映像。 如果指定的映像不存在於正在執行 Docker 建置流程的系統上，Docker 引擎會嘗試從公用或私用的映像登錄下載映像。

FROM 指令的格式會像這樣：

```dockerfile
FROM <image>
```

以下是從命令的範例：

若要從 Microsoft 容器登錄 (MCR) 下載 ltsc2019 版本 windows server core:
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

如需詳細資訊，請參閱的[FROM 參考](https://docs.docker.com/engine/reference/builder/#from)。

### <a name="run"></a>RUN

`RUN` 指令指定要執行並擷取至新容器映像的命令。 這些命令可以包含安裝軟體、建立檔案和目錄，以及建立環境設定等項目。

RUN 指令會像這樣：

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Exec 和 shell 形式之間的差異是如何在`RUN`指令的執行。 使用 Exec 形式時會明確執行指定的程式。

以下是 exec 形式的範例：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

產生的映像執行`powershell New-Item c:/test`命令：

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

作為對比，下列範例會執行相同的作業，在殼層表單中：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

產生的映像已執行的指令`cmd /S /C powershell New-Item c:\test`。

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>使用 Windows 執行使用的考量

在 Windows 中，當使用 exec 格式的 `RUN` 指令時，反斜線必須逸出。

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Windows 安裝程式的目標程式時，您將需要解壓縮透過安裝`/x:<directory>`旗標，才能啟動實際 （無訊息） 安裝程序。 您也必須等候命令結束之前執行其他動作。 否則，處理程序會提前結束，而不安裝任何項目。 如需詳細資訊，請參閱下面的範例。

#### <a name="examples-of-using-run-with-windows"></a>使用 Windows 執行使用的範例

下列範例 Dockerfile 使用 DISM 來安裝 IIS 容器映像：

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

此範例會安裝 Visual Studio 可轉散發套件。 `Start-Process` 和`-Wait`參數用來執行安裝程式。 這樣可確保在安裝完成之後再移到 Dockerfile 中的下一個指令。

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

如需 RUN 指令的詳細資訊，請參閱[執行參考](https://docs.docker.com/engine/reference/builder/#run)。

### <a name="copy"></a>複製

`COPY`指令會將檔案和目錄複製到容器的檔案系統。 檔案和目錄必須在相對於 Dockerfile 的路徑。

`COPY`指令的格式會像這樣：

```dockerfile
COPY <source> <destination>
```

如果來源或目的地包含空白字元，請在路徑方括弧和雙引號，如下列範例所示：

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>與 Windows 使用複製的考量

在 Windows 中，目的格式必須使用正斜線。 例如，這些都是有效的`COPY`指示：

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

同時，將無法運作反斜線具有下列格式：

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>與 Windows 使用複製的範例

下列範例會將來源目錄的內容目錄，名為`sqllite`中的容器映像：

```dockerfile
COPY source /sqlite/
```

下列範例會將以 config 開頭的所有檔案都新增`c:\temp`目錄的容器映像：

```dockerfile
COPY config* c:/temp/
```

如需詳細資訊，相關`COPY`指令，請參閱[COPY 參考](https://docs.docker.com/engine/reference/builder/#copy)。

### <a name="add"></a>ADD

ADD 指令是像與 COPY 指令，但具有更多的功能。 除了將檔案從主機複製到容器映像，`ADD` 指令也可以從具有 URL 規格的遠端位置複製檔案。

`ADD`指令的格式會像這樣：

```dockerfile
ADD <source> <destination>
```

如果來源或目的地包含空白字元，請在路徑方括弧和雙引號：

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>新增執行於 Windows 的考量

在 Windows 中，目的格式必須使用正斜線。 例如，這些都是有效的`ADD`指示：

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

同時，將無法運作反斜線具有下列格式：

```dockerfile
ADD test1.txt c:\temp\
```

此外，`ADD` 指令在 Linux 上會於複製時展開壓縮的封裝。 Windows 不提供這項功能。

#### <a name="examples-of-using-add-with-windows"></a>使用 Windows 新增使用的範例

下列範例會將來源目錄的內容目錄，名為`sqllite`中的容器映像：

```dockerfile
ADD source /sqlite/
```

下列範例會將所有開頭為 「 組態 」 的檔案新增到`c:\temp`目錄的容器映像。

```dockerfile
ADD config* c:/temp/
```

下列範例會針對 Windows 的 Python 下載至`c:\temp`目錄的容器映像。

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

如需詳細資訊，相關`ADD`指令，請參閱 [[加入參考](https://docs.docker.com/engine/reference/builder/#add)。

### <a name="workdir"></a>WORKDIR

`WORKDIR` 指令會為其他 Dockerfile 指令設定工作目錄，例如 `RUN`、`CMD`，也會為執行中的容器映像執行個體設定工作目錄。

`WORKDIR`指令的格式會像這樣：

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>使用 WORKDIR windows 考量

在 Windows 中，如果工作目錄包含一個反斜線，則必須逸出。

```dockerfile
WORKDIR c:\\windows
```

**範例**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

如需詳細資訊`WORKDIR`指令，請參閱[WORKDIR 參考](https://docs.docker.com/engine/reference/builder/#workdir)。

### <a name="cmd"></a>CMD

`CMD` 指令會設定部署容器映像執行個體時要執行的預設命令。 例如，如果容器會裝載 NGINX 網頁伺服器，`CMD`可能包含啟動網頁伺服器，透過像命令的指示`nginx.exe`。 如果 Dockerfile 中指定了多個 `CMD` 指令，則只會評估最後的指令。

`CMD`指令的格式會像這樣：

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>使用 CMD 與 Windows 的考量

在 Windows 中，`CMD` 指令中指定的檔案路徑必須使用正斜線或具有逸出的反斜線 `\\`。 下列項目都是有效的`CMD`指示：

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

不過，將無法運作不適當的斜線下列格式：

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

如需詳細資訊，相關`CMD`指令，請參閱[CMD 參考](https://docs.docker.com/engine/reference/builder/#cmd)。

## <a name="escape-character"></a>逸出字元

在許多情況下，Dockerfile 指示必須跨多行列出。 若要這樣做，您可以使用逸出字元。 預設的 Dockerfile 逸出字元為反斜線 `\`。 不過，因為反斜線也是在 Windows 中的檔案路徑分隔符號，使用它來跨多行列出可能會造成問題。 若要解決這個問題，您可以使用剖析器指示詞來變更預設的逸出字元。 如需剖析器指示詞的詳細資訊，請參閱[剖析器指示詞](https://docs.docker.com/engine/reference/builder/#parser-directives)。

下列範例顯示單一個 RUN 指示橫跨多行使用預設的逸出字元：

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

若要修改此逸出字元，可以將逸出剖析器指示詞置於 Dockerfile 的第一行。 這可以在下列範例所示。

>[!NOTE]
>只有兩個值可以用為逸出字元：`\`和`` ` ``。

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

如需逸出剖析器指示詞的詳細資訊，請參閱[逸出剖析器指示詞](https://docs.docker.com/engine/reference/builder/#escape)。

## <a name="powershell-in-dockerfile"></a>Dockerfile 中的 PowerShell

### <a name="powershell-cmdlets"></a>PowerShell Cmdlet

可使用 Dockerfile 中執行 PowerShell cmdlet`RUN`作業。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST 呼叫

PowerShell 的`Invoke-WebRequest`cmdlet 非常有用，從 web 服務收集資訊或檔案時。 例如，如果您要建置包含 Python 的影像，您可以設定`$ProgressPreference`到`SilentlyContinue`來達到更快速的下載項目，如下列範例所示。

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
>`Invoke-WebRequest` 也可以在 Nano Server 中運作。

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

在某些情況下，可能會很有幫助將指令碼複製到映像建立程序期間，您使用的容器，然後執行容器內的指令碼。

>[!NOTE]
>這會限制任何映像層快取，並降低 Dockerfile 的可讀性。

此範例使用 `ADD` 指令，從組建電腦中將指令碼複製到容器中。 接著使用 RUN 指令來執行此指令碼。

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker 建置

一旦您已建立了 Dockerfile 並儲存到磁碟，您可以執行`docker build`建立新的映像。 `docker build` 命令會使用幾個選擇性參數和通往 Dockerfile 的路徑。 如需 Docker 組建的完整文件，包括所有的清單建置選項，請參閱的[組建參考](https://docs.docker.com/engine/reference/commandline/build/#build)。

格式`docker build`命令會像這樣：

```dockerfile
docker build [OPTIONS] PATH
```

例如，下列命令會建立名為 「 iis。 「 映像

```dockerfile
docker build -t iis .
```

起始建置程序時輸出會指出狀態並傳回任何擲回的錯誤。

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

結果是新的容器映像，這在此範例中名為 「 iis 」。

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>進一步閱讀與參考

- [將 Dockerfiles 和 Docker 建置適用於 Windows](optimize-windows-dockerfile.md)
- [Dockerfile 參考](https://docs.docker.com/engine/reference/builder/)
