# <a name="using-insider-container-images"></a>使用測試人員容器映像

本練習將逐步引導您在 Windows Insider Preview 計畫中最新的 Windows Server 測試人員組建上部署及使用 Windows 容器功能。 在本練習中，您將會安裝容器角色並部署基本 OS 映像預覽版本。 如果您需要熟悉一下容器，可以在[關於容器](../about/index.md)中找到這項資訊。

本快速入門是針對 Windows Server Insider Preview 計畫的 Windows Server 容器。 請先自行熟悉計畫內容，然後再繼續進行本快速入門。

**必要條件：**

- 加入 [Windows 測試人員計畫](https://insider.windows.com/GettingStarted)並檢閱使用規定。
- 一部電腦系統 (實體或虛擬)，而且執行 Windows 測試人員計畫中最新的 Windows Server 組建及/或 Windows 測試人員計畫中最新的 Windows 10 組建。

>您必須使用 Windows Server Insider Preview 計畫中的 Windows Server 組建或 Windows Insider Preview 計畫中的 Windows 10 組建，才能使用以下所述的基本映像。 如果您沒有使用上述其中一種組建，使用這些基本映像就會導致您無法啟動容器。

## <a name="install-docker"></a>安裝 Docker
需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 為了獲得使用容器最佳化 Nano Server 映像的最佳體驗，您也需要支援多階段組建的 Docker 版本。

我們將使用 OneGet 提供者 PowerShell 模組安裝 Docker。 此提供者會啟用您電腦上的容器功能並安裝 Docker，這會需要重新開機。 請注意，目前有許多管道針對不同情況的用途提供不同的 Docker 版本。 在本練習中，我們將使用來自穩定 (Stable) 管道的最新 Docker 社群版本。 如果您想要測試最新的 Docker 開發中版本，還有邊緣 (Edge) 管道可用。

開啟提高權限的 PowerShell 工作階段，並執行下列命令。

>注意：在測試人員組建中安裝 Docker 所需要的提供者通常有別於目前所使用的提供者。 請注意下列差異。

安裝 OneGet PowerShell 模組。
```powershell
Install-Module -Name DockerMsftProviderInsider -Repository PSGallery -Force
```
使用 OneGet 安裝最新版的 Docker。
```powershell
Install-Package -Name docker -ProviderName DockerMsftProviderInsider -RequiredVersion 17.06.0-ce
```
安裝完成時，請重新啟動電腦。
```
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>安裝基本容器映像

使用 Windows 容器之前，必須先安裝基本映像。 加入 Windows 測試人員計畫之後，您也可以測試最新的基本映像組建。 在測試人員基本映像中，目前有 4 個以 Windows Server 為基礎的基本映像可用。 請參閱下表以查看每個映像的用途：

| 基本 OS 映像                       | 用途                      |
|-------------------------------------|----------------------------|
| microsoft/windowsservercore         | 生產與開發 |
| microsoft/nanoserver                | 生產與開發 |
| microsoft/windowsservercore-insider | 僅限開發           |
| microsoft/nanoserver-insider        | 僅限開發           |

若要提取 Nano Server 測試人員基本映像，請執行下列命令：

```
docker pull microsoft/nanoserver-insider
```

若要提取 Windows Server Core 測試人員基本映像，請執行下列命令：

```
docker pull microsoft/windowsservercore-insider
```

請參閱「Windows 容器 OS 映像授權條款」(位於這裡：[授權條款](../EULA.md )) 以及「Windows 測試人員計畫使用規定」(位於這裡：[使用規定](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver))。

## <a name="next-steps"></a>後續步驟

[使用或不使用 .NET Core 2.0 或 PowerShell Core 6 建置並執行應用程式](./Nano-RS3-.NET-Core-and-PS.md)
