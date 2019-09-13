---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入門
keywords: docker、樹枝、LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 5f0922a1ee2588b6e5a06091fe34e07ceadf89cb
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129365"
---
# <a name="get-started-configure-your-environment-for-containers"></a>快速入門：設定容器的環境

此快速入門示範如何：

> [!div class="checklist"]
> * 為容器設定您的環境
> * 執行您的第一個容器影像
> * Containerize 簡單的 .NET core 應用程式

## <a name="prerequisites"></a>必要條件

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

請確定您符合下列需求：

- 一種執行 Windows Server 2016 或更新版本的電腦系統（物理或虛擬）。

> [!NOTE]
> 如果您使用的是 Windows Server 2019 測試人員預覽版，請更新至[Window server 2019 評估](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 )。

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 專業版與企業版](#tab/Windows-10-Client)

請確定您符合下列需求：

- 一種執行 Windows 10 專業版或企業版的物理電腦系統，並含周年紀念更新（版本1607）或更新版本。
- 應該啟用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

> [!NOTE]
>  從 Windows 10 年10月更新2018開始，我們不再允許使用者在 Windows 10 企業版或專業版（適用于開發人員/測試目的）上，以進程隔離模式執行 Windows 容器。 如需深入瞭解，請參閱[常見問題](../about/faq.md)。 
> 
> Windows Server 容器預設會在 Windows 10 上使用 Hyper-v 隔離，以便為開發人員提供將在生產中使用的相同內核版本和配置。 深入瞭解我們檔的 [[概念](../manage-containers/hyperv-container.md)] 區域中的 hyper-v 隔離。

---
<!-- stop tab view -->

## <a name="install-docker"></a>安裝 Docker

Docker 是使用 Windows 容器的權威性工具鏈。 Docker 提供可讓使用者管理指定主機上的容器、組建容器、移除容器等的 CLI。 深入瞭解我們檔的 [[概念](../manage-containers/configure-docker-daemon.md)] 區域中的 Docker。

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

在 Windows Server 上，Docker 是透過 Microsoft 發佈的[OneGet 提供者 PowerShell 模組](https://github.com/oneget/oneget)來安裝，稱為[DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)。 這個提供者：

- 在您的電腦上啟用容器功能
- 在您的電腦上安裝 Docker 引擎和用戶端。

若要安裝 Docker，請開啟提升的 PowerShell 會話，然後從[PowerShell 庫](https://www.powershellgallery.com/packages/DockerMsftProvider)安裝 Docker-Microsoft PackageManagement 提供者。

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

接著，使用 PackageManagement PowerShell 模組來安裝最新版本的 Docker。

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

當 PowerShell 詢問是否要信任封裝來源 'DockerDefault' 時，輸入 `A` 以繼續安裝。 安裝完成後，您必須重新開機電腦。

```powershell
Restart-Computer -Force
```

> [!TIP]
> 如果您想要稍後更新 Docker：
>  - 若要查看已安裝的版本，請使用 `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 若要尋找最新版本，請使用 `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - 準備就緒後，請使用 `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force` 進行升級，後面接著 `Start-Service Docker`

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 專業版與企業版](#tab/Windows-10-Client)

在 Windows 10 專業版與企業版上，Docker 是透過傳統安裝程式安裝。 下載[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)並執行安裝程式。 您必須登入。 如果您還沒有帳戶，請建立一個帳戶。 在[Docker 檔](https://docs.docker.com/docker-for-windows/install)中提供更詳細的安裝指示。

安裝之後，Docker 桌面預設為執行 Linux 容器。 使用 Docker 託盤功能表，或在 PowerShell 提示中執行下列命令，以切換到 Windows 容器：

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>後續步驟

現在您的環境已正確設定，請遵循連結以瞭解如何提取並執行容器。

> [!div class="nextstepaction"]
> [執行您的第一個容器](./run-your-first-container.md)
