
# <a name="using-insider-container-images"></a>使用測試人員容器映像

本練習將逐步引導您在 Windows Insider Preview 計畫中最新的 Windows Server 測試人員組建上部署及使用 Windows 容器功能。 在本練習中，您將會安裝容器角色並部署基本 OS 映像預覽版本。 如果您需要熟悉一下容器，可以在[關於容器](../about/index.md)中找到這項資訊。

本快速入門是針對 Windows Server Insider Preview 計畫的 Windows Server 容器。 請先自行熟悉計畫內容，然後再繼續進行本快速入門。

## <a name="prerequisites"></a>必要條件：

- 加入 [Windows 測試人員計畫](https://insider.windows.com/GettingStarted)並檢閱使用規定。
- 一部電腦系統 (實體或虛擬)，而且執行 Windows 測試人員計畫中最新的 Windows Server 組建及/或 Windows 測試人員計畫中最新的 Windows 10 組建。

> [!IMPORTANT]
> 您必須使用 windows Server 測試人員預覽版程式的 Windows Server 組建, 或從 Windows 測試人員預覽版程式建立 Windows 10 版本, 才能使用下面所述的基本影像。 如果您沒有使用上述其中一種組建，使用這些基本映像就會導致您無法啟動容器。

## <a name="install-docker-enterprise-edition-ee"></a>安裝 Docker Enterprise Edition (EE)

需要有 Docker EE 才能使用 Windows 容器。 Docker EE 是由 Docker 引擎及 Docker 用戶端所組成。

我們將使用 OneGet 提供者 PowerShell 模組來安裝 Docker EE。 此提供者會啟用您電腦上的容器功能並安裝 Docker EE，這會需要重新開機。 開啟提高權限的 PowerShell 工作階段，並執行下列命令。

> [!NOTE]
> 在 Windows Server 測試人員組建中安裝 Docker EE 時, 需要的 OneGet 提供者與非測試人員組建所用的不同。 如果 Docker EE 和 DockerMsftProvider OneGet 提供者已經安裝，必須將它們移除才能繼續。

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

安裝 OneGet PowerShell 模組以用於 Windows 測試人員組建。

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

使用 OneGet 安裝最新版的 Docker EE 預覽版。

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

安裝完成時，請重新啟動電腦。

```powershell
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>安裝基本容器映像

使用 Windows 容器之前，必須先安裝基本映像。 加入 Windows 測試人員計畫之後，您也可以測試最新的基本映像組建。 在測試人員基本映像中，目前有 4 個以 Windows Server 為基礎的基本映像可用。 請參閱下表以查看每個映像的用途：

| 基本 OS 映像                       | 用途                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | 生產與開發 |
| mcr.microsoft.com/windows/nanoserver              | 生產與開發 |
| mcr.microsoft.com/windows/servercore/insider | 僅限開發           |
| mcr.microsoft.com/windows/nanoserver/insider        | 僅限開發           |

若要提取 Nano Server 測試人員基本映像，請執行下列命令：

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

若要提取 Windows Server Core 測試人員基本映像，請執行下列命令：

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> 請閱讀 Windows 容器 OS 影像[EULA](../EULA.md )和 windows 測試人員計畫[使用條款](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [建立並執行範例應用程式](./Nano-RS3-.NET-Core-and-PS.md)
