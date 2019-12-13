---
title: 使用容器搭配 Windows 測試人員程式
description: 瞭解如何透過 Windows 測試人員程式開始使用 windows 容器
keywords: docker，容器，insider，windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909888"
---
# <a name="use-containers-with-the-windows-insider-program"></a>使用容器搭配 Windows 測試人員程式

本練習將逐步引導您在 Windows Insider Preview 計畫中最新的 Windows Server 測試人員組建上部署及使用 Windows 容器功能。 在本練習中，您將會安裝容器角色並部署基本 OS 映像預覽版本。 如果您需要熟悉一下容器，可以在[關於容器](../about/index.md)中找到這項資訊。

## <a name="join-the-windows-insider-program"></a>加入 Windows 測試人員計畫

若要執行 Windows 容器的 insider 版本，您必須有一部主機，從 [Windows 測試人員程式] 和（或） [Windows 測試人員] 程式中的最新 Windows 10 組建執行 Windows Server 的最新組建。 加入[Windows 測試人員方案](https://insider.windows.com/GettingStarted)，並查看使用規定。

> [!IMPORTANT]
> 您必須從 Windows Server Insider Preview 計畫或 Windows 10 組建中，使用 Windows 測試人員 Preview 計畫的 Windows Server 組建，才能使用如下所述的基底映射。 如果您沒有使用上述其中一種組建，使用這些基本映像就會導致您無法啟動容器。

## <a name="install-docker"></a>安裝 Docker

如果您尚未安裝 Docker，請遵循[開始使用指南](../quick-start/set-up-environment.md)來安裝 docker。

## <a name="pull-an-insider-container-image"></a>提取 insider 容器映射

藉由成為 Windows 測試人員程式的一部分，您可以使用我們最新的組建作為基底映射。

若要提取 Nano Server 測試人員基本映像，請執行下列命令：

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

若要提取 Windows Server Core 測試人員基本映像，請執行下列命令：

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

「Windows」和「IoTCore」基底映射也有可供提取的 insider 版本。 您可以在[容器基底映射](../manage-containers/container-base-images.md)檔中閱讀更多有關可用 insider 基底映射的資訊。

> [!IMPORTANT]
> 請閱讀 Windows 容器 OS 映射的[EULA](../images-eula.md )和 Windows 測試人員計畫[使用](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)規定。
