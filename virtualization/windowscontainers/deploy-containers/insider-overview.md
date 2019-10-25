---
title: 使用容器搭配 Windows 測試人員計畫
description: 瞭解如何開始使用 windows 容器與 Windows 測試人員計畫
keywords: docker、容器、測試人員、windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 6080b2c5053720490d374f6fb0daa870d5ddd4e8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257792"
---
# <a name="use-containers-with-the-windows-insider-program"></a>使用容器搭配 Windows 測試人員計畫

本練習將逐步引導您在 Windows Insider Preview 計畫中最新的 Windows Server 測試人員組建上部署及使用 Windows 容器功能。 在本練習中，您將會安裝容器角色並部署基本 OS 映像預覽版本。 如果您需要熟悉一下容器，可以在[關於容器](../about/index.md)中找到這項資訊。

## <a name="join-the-windows-insider-program"></a>加入 Windows 測試人員計畫

若要執行 Windows 測試人員版本，您必須從 Windows 測試人員計畫，以及/或 Windows 測試人員計畫中的最新 windows 10 組建執行主機。 加入 Windows 測試人員[計畫](https://insider.windows.com/GettingStarted)並查看使用條款。

> [!IMPORTANT]
> 您必須使用 windows Server 測試人員預覽版程式的 Windows Server 組建，或從 Windows 測試人員預覽版程式建立 Windows 10 版本，才能使用下面所述的基本影像。 如果您沒有使用上述其中一種組建，使用這些基本映像就會導致您無法啟動容器。

## <a name="install-docker"></a>安裝 Docker

如果您沒有安裝 Docker，請遵循[入門](../quick-start/set-up-environment.md)指南來安裝 docker。

## <a name="pull-an-insider-container-image"></a>拉 insider 容器影像

透過成為 Windows 測試人員計畫的一部分，您可以使用我們的基本映射的最新組建。

若要提取 Nano Server 測試人員基本映像，請執行下列命令：

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

若要提取 Windows Server Core 測試人員基本映像，請執行下列命令：

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

[Windows] 和 "IoTCore" 基底影像也有可供拉入的測試人員版本。 您可以在[容器基底影像](../manage-containers/container-base-images.md)檔中進一步瞭解可用的「測試人員-基本」影像。

> [!IMPORTANT]
> 請閱讀 Windows 容器 OS 影像[EULA](../images-eula.md )和 windows 測試人員計畫[使用條款](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)。
