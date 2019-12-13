---
title: Windows 10 上的 windows 和 Linux 容器
description: 設定適用于容器的 Windows 10 或 Windows Server，然後繼續執行您的第一個容器映射。
keywords: docker、容器、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909558"
---
# <a name="get-started-prep-windows-for-containers"></a>開始使用：準備適用于容器的 Windows

本教學課程說明如何：

- 設定適用于容器的 Windows 10 或 Windows Server
- 執行您的第一個容器映射
- 容器化簡單的 .NET core 應用程式

## <a name="prerequisites"></a>必要條件

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

若要在 Windows Server 上執行容器，您需要執行 Windows Server （半年通道）、Windows Server 2019 或 Windows Server 2016 的實體伺服器或虛擬機器。

若要進行測試，您可以下載一份[Window server 2019 評估版](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )或[Windows server Insider preview](https://insider.windows.com/for-business-getting-started-server/)。

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

若要在 Windows 10 上執行容器，您需要下列各項：

- 一部執行 Windows 10 專業版或企業版含年度更新版（版本1607）或更新版本的實體電腦系統。
- 應啟用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

> [!NOTE]
>  從 Windows 10 10 月更新2018開始，我們不再禁止使用者在 Windows 10 企業版或專業版上以程式隔離模式執行 Windows 容器，以進行開發/測試用途。 若要深入瞭解，請參閱[常見問題](../about/faq.md)。 
> 
> Windows Server 容器預設會在 Windows 10 上使用 Hyper-v 隔離，以便為開發人員提供將在生產環境中使用的相同核心版本和設定。 在檔的 [[概念](../manage-containers/hyperv-container.md)] 區域中深入瞭解 hyper-v 隔離。

---
<!-- stop tab view -->

## <a name="install-docker"></a>安裝 Docker

第一個步驟是安裝 Docker，這是使用 Windows 容器的必要條件。 Docker 提供適用于容器的標準執行時間環境，其中包含通用 API 和命令列介面（CLI）。

如需更多設定詳細資訊，請參閱[Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)。

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

若要在 Windows Server 上安裝 Docker，您可以使用由 Microsoft 所發佈的[OneGet 提供者 PowerShell 模組](https://github.com/oneget/oneget)，稱為[DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)。 此提供者會啟用 Windows 中的 [容器] 功能，並安裝 Docker 引擎和用戶端。 方法如下：

1. 開啟已提升許可權的 PowerShell 會話，並從[PowerShell 資源庫](https://www.powershellgallery.com/packages/DockerMsftProvider)安裝 Docker-Microsoft PackageManagement 提供者。

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   如果系統提示您安裝 NuGet 提供者，請輸入 `Y` 以進行安裝。

2. 使用 PackageManagement PowerShell 模組來安裝最新版本的 Docker。

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   當 PowerShell 詢問是否要信任封裝來源 'DockerDefault' 時，輸入 `A` 以繼續安裝。
3. 安裝完成之後，請重新開機電腦。

   ```powershell
   Restart-Computer -Force
   ```

如果您稍後想要更新 Docker：

- 使用 `Get-Package -Name Docker -ProviderName DockerMsftProvider` 檢查已安裝的版本
- 尋找具有 `Find-Package -Name Docker -ProviderName DockerMsftProvider` 的目前版本
- 當您準備好時，請使用 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`進行升級，然後 `Start-Service Docker`

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

您可以使用下列步驟，在 Windows 10 專業版和企業版上安裝 Docker。 

1. 下載並安裝[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)，並建立免費的 docker 帳戶（如果您還沒有的話）。 如需詳細資訊，請參閱[Docker 檔](https://docs.docker.com/docker-for-windows/install)。

2. 在安裝期間，將預設容器類型設定為 [Windows 容器]。 若要在安裝完成後切換，您可以使用 Windows 系統匣中的 Docker 專案（如下所示），或 PowerShell 提示字元中的下列命令：

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![顯示 [切換至 Windows 容器] 命令的 Docker system 工作列功能表。](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>後續步驟

現在您已正確設定您的環境，請遵循連結以瞭解如何執行容器。

> [!div class="nextstepaction"]
> [執行您的第一個容器](./run-your-first-container.md)
