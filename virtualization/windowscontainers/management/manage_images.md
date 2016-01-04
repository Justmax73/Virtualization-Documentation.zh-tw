# 容器映像

**這是初版內容，後續可能會變更。**

容器映像可用來部署容器。 這些映像可包含作業系統、應用程式和所有的應用程式相依性。 例如，您可以開發已預先設定 Nano Server、IIS 和執行於 IIS 之應用程式的容器映像。 接著，此容器映像可儲存在容器登錄中供後續使用、部署在任何 Windows 容器主機上 (內部部署、雲端，甚至是容器服務)，也可以做為新容器映像的基底。

容器映像分成兩種類型：

- 基本 OS 映像 – 此類型由 Microsoft 提供，其中包含 Core OS 元件。
- 容器映像 – 從基本 OS 映像建立的容器映像。

## PowerShell

### 列出映像

執行 `get-containerImage`，以傳回容器主機上的映像清單。 容器映像類型可透過 `IsOSImage` 屬性來區別。

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### 安裝基本 OS 映像

您可以使用 ContainerProvider PowerShell 模組尋找並安裝容器 OS 映像。 此模組必須先安裝，才可使用。 下列命令可用來安裝此模組。

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

若要下載並安裝 Nano Server 基本 OS 映像，請執行下列命令。

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

同樣地，此命令也將下載並安裝 Windows Server Core 基本 OS 映像。

> **問題** Save-ContainerImage 和 Install-ContainerImage Cmdlet 無法在 PowerShell 遠端工作階段中使用 WindowsServerCore 容器映像。 **因應措施：**使用遠端桌面登入機器，並直接使用 Save-ContainerImage Cmdlet。

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
如需容器映像管理的詳細資訊，請參閱 [Windows 容器映像](../management/manage_images.md)。

### 建立新映像

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

新映像建立後，將會依存於做為其建立來源的映像。 可使用 `get-containerimage` 命令來檢視此相依性。 如果未列出父映像，即表示此映像是基本 OS 映像。

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

## Docker

### 列出映像

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### 建立新映像

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

### Docker Hub

Docker Hub 登錄包含可下載至容器主機上的預先建置映像。 這些映像在下載後，將可做為 Windows 容器應用程式的基底。

若要可從 Docker Hub 取得的映像清單，請使用 `docker search` 命令。 注意：從 Docker Hub 提取依存於 Windows Server Core 的映像之前，必須先安裝 Windows Serve Core 基本 OS 映像。

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
```

若要從 Docker Hub 下載映像，請使用 `docker pull`。

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

現在，執行 `docker images` 時顯示可以看見此映像。

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

### 映像相依性

若要使用 Docker 檢視映像相依性，可以使用 `docker history` 命令。

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```



