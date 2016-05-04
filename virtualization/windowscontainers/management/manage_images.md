---
author: neilpeterson
---

# 容器映像

**這是初版內容，後續可能會變更。**

容器映像可用來部署容器。 這些映像可包含作業系統、應用程式和所有的應用程式相依性。 例如，您可以開發已預先設定 Nano Server、IIS 和執行於 IIS 之應用程式的容器映像。 接著，此容器映像可儲存在容器登錄中供後續使用、部署在任何 Windows 容器主機上 (內部部署、雲端，甚至是容器服務)，也可以做為新容器映像的基底。

容器映像分成兩種類型：

- **基本 OS 映像** – 此類型由 Microsoft 提供，其中包含核心 OS 元件。
- **容器映像** – 從基本 OS 映像衍生出的自訂容器映像。

## 基本 OS 映像

### 安裝映像

您可以使用 ContainerProvider PowerShell 模組來尋找並安裝容器 OS 映像，以進行 PowerShell 和 Docker 管理。 此模組必須先安裝，才可使用。 您可以使用下列命令來安裝此模組。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

安裝完成後，即可使用 `Find-ContainerImage` 傳回基本 OS 映像的清單。

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

若要下載並安裝 Nano Server 基本 OS 映像，請執行下列命令。 `-version` 是選擇性參數。 若未指定基本 OS 映像版本，將會安裝最新版本。

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

同樣地，此命令也將下載並安裝 Windows Server Core 基本 OS 映像。 `-version` 是選擇性參數。 若未指定基本 OS 映像版本，將會安裝最新版本。

> **問題** Save-ContainerImage 和 Install-ContainerImage Cmdlet 可能無法在 PowerShell 遠端工作階段中與 WindowsServerCore 容器映像搭配運作。 **因應措施：**使用遠端桌面登入機器，並直接使用 Save-ContainerImage Cmdlet。

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

使用 `Get-ContainerImage` 命令確認已安裝映像。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

> **Install-ContainerImage** 會安裝基本 OS 映像，以用於受 PowerShell 或 Docker 管理的容器。 如果基本 OS 映像已下載完畢，但在執行 `docker images` 時未顯示，請使用服務控制台小程式或依序使用命令 'sc docker stop' 和 'sc docker start'，以重新啟動 Docker 服務

### 離線安裝

基本 OS 映像也可以在沒有網際網路連線的情況下安裝。 若要這樣做，請透過網際網路連線將映像下載到電腦上、複製到目標系統，然後使用 `Install-ContainerOSImages` 命令將其匯入。

在下載基本 OS 映像前，請執行下列命令，讓系統備有容器映像提供者。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

從 PowerShell OneGet 封裝管理員傳回映像清單：

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

若要下載映像，請使用 `Save-ContainerImage` 命令。

```powershell
PS C:\> Save-ContainerImage -Name NanoServer -Destination c:\container-image\NanoServer.wim
```

現在可以將下載好的容器映像複製到其他容器主機，並使用 `Install-ContainerOSImage` 命令進行安裝。

```powershell
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### 標記映像

依名稱參考容器映像時，Docker 引擎會搜尋最新版本的映像。 如果無法判斷最新版本，將會擲回下列錯誤。

```powershell
PS C:\> docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

安裝 Windows Server Core 或 Nano Server 基本 OS 映像後，將必須以「最新」版本加以標記。 若要這樣做，請使用 `docker tag` 命令。

如需有關 `docker tag` 的詳細資訊，請參閱[在 docker.com 上標記、推播及提取您的映象](https://docs.docker.com/mac/step_six/)。

```powershell
PS C:\> docker tag <image id> windowsservercore:latest
```

在加上記籤後，`docker images` 的輸出會顯示相同映像的兩種版本，一個具有映像版本的標記，另一個則有「最新」標記。 現在可依名稱參考映像。

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14289.1000     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14289.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### 將 OS 映像解除安裝

您可以使用 `Uninstall-ContainerOSImage` 命令，將基本 OS 映像解除安裝。 下列範例會將 NanoServer 基本 OS 映像解除安裝。

```powershell
Get-ContainerImage -Name NanoServer | Uninstall-ContainerOSImage
```

## 容器映像 PowerShell

### 列出映像

執行 `Get-ContainerImage` 以傳回容器主機上的映像清單。 容器映像類型可透過 `IsOSImage` 屬性來區別。

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### 建立新的映像

您可以從任何現有容器建立新的容器映像。 若要這樣做，請使用 `New-ContainerImage` 命令。

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### 移除映像

如果有任何容器 (即便處於停止狀態) 具有對映像的相依性，容器映像即無法移除。

使用 PowerShell 移除單一映像。

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### 映像相依性

新映像建立後，將會依存於做為其建立來源的映像。 使用 `Get-ContainerImage` 命令可查看此相依性。 如果未列出父映像，即表示此映像是基本 OS 映像。

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

### 移動映像儲存機制

使用 `New-ContainerImage` 命令建立新的容器映像時，此映像會儲存在預設位置 'C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store'。 使用 `Move-ContainerImageRepository` 命令可移動此儲存機制。 例如，以下會在 'c:\container-images' 位置建立新的容器映像儲存機制。

```powershell
Move-ContainerImageRepository -Path c:\container-images
```
> 搭配 `Move-ContainerImageRepository` 命令使用的路徑，在命令執行時不得存在。

## 容器映像 Docker

### 列出映像

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### 建立新的映像

您可以從任何現有容器建立新的容器映像。 若要這樣做，請使用 `docker commit` 命令。 下列範例會建立名為 ‘windowsservercoreiis’ 的新容器映像。

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### 移除映像

如果有任何容器 (即便處於停止狀態) 具有對映像的相依性，容器映像即無法移除。

使用 docker 移除映像時，可依名稱或識別碼來參考映像。

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### 映像相依性

若要使用 Docker 檢視映像相依性，可以使用 `docker history` 命令。

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Docker Hub 登錄包含可下載至容器主機上的預先建置映像。 這些映像在下載後，將可做為 Windows 容器應用程式的基底。

若要可從 Docker Hub 取得的映像清單，請使用 `docker search` 命令。 注意 – 從 Docker Hub 提取這些依存容器映像前，必須先安裝 Windows Serve Core 或 Nano Server 基本 OS 映像。

> 以「nano-」開頭的映像，對 Nano Server 基本 OS 映像具有相依性。

```powershell
C:\> docker search *

NAME                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/aspnet        ASP.NET 5 framework installed in a Windows...   1         [OK]       [OK]
microsoft/django        Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35      .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/golang        Go Programming Language installed in a Win...   1                    [OK]
microsoft/httpd         Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis           Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/mongodb       MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/mysql         MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/nginx         Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/node          Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/php           PHP running on Internet Information Servic...   1                    [OK]
microsoft/python        Python installed in a Windows Server Core ...   1                    [OK]
microsoft/rails         Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/redis         Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/ruby          Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sqlite        SQLite installed in a Windows Server Core ...   1                    [OK]
microsoft/nano-golang   Go Programming Language installed in a Nan...   1                    [OK]
microsoft/nano-httpd    Apache httpd installed in a Nano Server ba...   1                    [OK]
microsoft/nano-iis      Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/nano-mysql    MySQL installed in a Nano Server based con...   1                    [OK]
microsoft/nano-nginx    Nginx installed in a Nano Server based con...   1                    [OK]
microsoft/nano-node     Node installed in a Nano Server based cont...   1                    [OK]
microsoft/nano-python   Python installed in a Nano Server based co...   1                    [OK]
microsoft/nano-rails    Ruby on Rails installed in a Nano Server b...   1                    [OK]
microsoft/nano-redis    Redis installed in a Nano Server based con...   1                    [OK]
microsoft/nano-ruby     Ruby installed in a Nano Server based cont...   1                    [OK]
```

若要從 Docker Hub 下載映像，請使用 `docker pull`。

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

現在，執行 `docker images` 時可以看見此映像。

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```






<!--HONumber=Mar16_HO3-->


