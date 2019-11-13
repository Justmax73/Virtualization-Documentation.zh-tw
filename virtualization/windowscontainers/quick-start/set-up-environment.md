---
title: Windows 10 上的 windows 和 Linux 容器
description: 針對 [容器] 設定 Windows 10 或 Windows Server，然後移至執行您的第一個容器影像。
keywords: docker、樹枝、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288116"
---
# <a name="get-started-prep-windows-for-containers"></a>快速入門：準備適用于容器的視窗

本教學課程說明如何：

- 設定 Windows 10 或 Windows Server for 容器
- 執行您的第一個容器影像
- Containerize 簡單的 .NET core 應用程式

## <a name="prerequisites"></a>必要條件

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

若要在 Windows Server 上執行容器，您需要執行 Windows Server （半年通道）、Windows Server 2019 或 Windows Server 2016 的物理伺服器或虛擬機器。

針對測試，您可以下載[Window server 2019 評估版](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )或[Windows Server](https://insider.windows.com/for-business-getting-started-server/)測試人員預覽版的複本。

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

若要在 Windows 10 上執行容器，您必須執行下列動作：

- 一種執行 Windows 10 專業版或企業版的物理電腦系統，並含周年紀念更新（版本1607）或更新版本。
- 應該啟用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

> [!NOTE]
>  從 Windows 10 年10月更新2018開始，我們不再允許使用者在 Windows 10 企業版或專業版（適用于開發人員/測試目的）上，以進程隔離模式執行 Windows 容器。 如需深入瞭解，請參閱[常見問題](../about/faq.md)。 
> 
> Windows Server 容器預設會在 Windows 10 上使用 Hyper-v 隔離，以便為開發人員提供將在生產中使用的相同內核版本和配置。 深入瞭解我們檔的 [[概念](../manage-containers/hyperv-container.md)] 區域中的 hyper-v 隔離。

---
<!-- stop tab view -->

## <a name="install-docker"></a>安裝 Docker

第一個步驟是安裝 Docker，這是使用 Windows 容器所需要的。 Docker 可為容器提供標準的執行時間環境，使用通用的 API 與命令列介面（CLI）。

如需更多配置詳細資料，請參閱[Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)。

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

若要在 Windows Server 上安裝 Docker，您可以使用 Microsoft 發佈的[OneGet 提供者 PowerShell 模組](https://github.com/oneget/oneget)（稱為[DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)）。 這個提供者會啟用 Windows 中的 [容器] 功能，並安裝 Docker 引擎和用戶端。 方法如下：

1. 開啟提升許可權的 PowerShell 會話，然後從[PowerShell 庫](https://www.powershellgallery.com/packages/DockerMsftProvider)安裝 Docker-Microsoft PackageManagement 提供者。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   如果系統提示您安裝 NuGet 提供者，請輸入`Y`並安裝。

2. 使用 PackageManagement PowerShell 模組來安裝最新版本的 Docker。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   當 PowerShell 詢問是否要信任封裝來源 'DockerDefault' 時，輸入 `A` 以繼續安裝。
3. 安裝完成後，請重新開機電腦。

   ```powershell
   Restart-Computer -Force
   ```

如果您想要稍後更新 Docker：

- 若要查看已安裝的版本，請使用 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- 若要尋找最新版本，請使用 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- 準備就緒後，請使用 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` 進行升級，後面接著 `Start-Service Docker`

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

您可以使用下列步驟，在 Windows 10 專業版和企業版上安裝 Docker。 

1. 如果您還沒有可用的 Docker 桌面，請下載並安裝該[桌上型電腦](https://store.docker.com/editions/community/docker-ce-desktop-windows)。 如需更多詳細資料，請參閱[Docker 檔](https://docs.docker.com/docker-for-windows/install)。

2. 在安裝期間，請將預設容器類型設定為 Windows 容器。 若要在安裝完成後切換，您可以使用 Windows 系統託盤中的 Docker 專案（如下所示），或 PowerShell 提示中的下列命令：

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![顯示「切換到 Windows 容器」命令的 Docker 系統工作列功能表。](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>後續步驟

現在您的環境已正確設定，請遵循連結以瞭解如何執行容器。

> [!div class="nextstepaction"]
> [執行您的第一個容器](./run-your-first-container.md)
