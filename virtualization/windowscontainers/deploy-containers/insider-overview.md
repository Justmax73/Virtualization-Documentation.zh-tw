---
title: 使用容器搭配 Windows 測試人員計畫
description: 瞭解如何開始使用 windows 容器與 Windows 測試人員計畫
keywords: docker、容器、測試人員、windows
author: cwilhit
ms.openlocfilehash: 137209a66c3d0b907003498fe78a04a57a140130
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254386"
---
# <a name="use-containers-with-the-windows-insider-program"></a>使用容器搭配 Windows 測試人員計畫

本練習將逐步引導您在 Windows Insider Preview 計畫中最新的 Windows Server 測試人員組建上部署及使用 Windows 容器功能。 在本練習中，您將會安裝容器角色並部署基本 OS 映像預覽版本。 如果您需要熟悉一下容器，可以在[關於容器](../about/index.md)中找到這項資訊。

> [!NOTE]
> 此內容是 Windows Server 測試人員預覽版程式上的 Windows Server 容器所特有的。 如果您正在尋找使用 Windows 容器的非測試人員指示，請參閱[快速入門](../quick-start/set-up-environment.md)指南。

## <a name="join-the-windows-insider-program"></a>加入 Windows 測試人員計畫

若要執行 Windows 測試人員版本，您必須從 Windows 測試人員計畫，以及/或 Windows 測試人員計畫中的最新 windows 10 組建執行主機。 加入 Windows 測試人員[計畫](https://insider.windows.com/GettingStarted)並查看使用條款。

> [!IMPORTANT]
> 您必須使用 windows Server 測試人員預覽版程式的 Windows Server 組建，或從 Windows 測試人員預覽版程式建立 Windows 10 版本，才能使用下面所述的基本影像。 如果您沒有使用上述其中一種組建，使用這些基本映像就會導致您無法啟動容器。

## <a name="install-docker"></a>安裝 Docker

<!-- start tab view -->
# [<a name="windows-server-insider"></a>Windows Server 測試人員](#tab/Windows-Server-Insider)

我們將使用 OneGet 提供者 PowerShell 模組來安裝 Docker EE。 此提供者會啟用您電腦上的容器功能並安裝 Docker EE，這會需要重新開機。 開啟提高權限的 PowerShell 工作階段，並執行下列命令。

> [!NOTE]
> 在 Windows Server 測試人員組建中安裝 Docker EE 時，需要的 OneGet 提供者與非測試人員組建所用的不同。 如果 Docker EE 和 DockerMsftProvider OneGet 提供者已經安裝，必須將它們移除才能繼續。

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

# [<a name="windows-10-insider"></a>Windows 10 測試人員](#tab/Windows-10-Insider)

在 Windows 10 測試人員，Docker 邊緣是透過與 Docker 桌上型電腦穩定的相同安裝程式安裝。 下載[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)並執行安裝程式。 您必須登入。 如果您還沒有帳戶，請建立一個帳戶。 在[Docker 檔](https://docs.docker.com/docker-for-windows/install)中提供更詳細的安裝指示。

安裝之後，開啟 Docker 的設定並切換至 [邊緣] 頻道。

![](./media/docker-edge-instruction.png)

---
<!-- stop tab view -->

## <a name="pull-an-insider-container-image"></a>拉 insider 容器影像

使用 Windows 容器之前，必須先安裝基本映像。 透過成為 Windows 測試人員計畫的一部分，您可以使用我們的基本映射的最新組建。 您可以在[容器基底影像](../manage-containers/container-base-images.md)檔中進一步瞭解可用的基底影像。

若要提取 Nano Server 測試人員基本映像，請執行下列命令：

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

若要提取 Windows Server Core 測試人員基本映像，請執行下列命令：

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> 請閱讀 Windows 容器 OS 影像[EULA](../images-eula.md )和 windows 測試人員計畫[使用條款](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)。
