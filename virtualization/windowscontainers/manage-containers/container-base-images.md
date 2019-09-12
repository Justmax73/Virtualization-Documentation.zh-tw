---
title: Windows 容器基本映像歷程記錄
description: 已發行的 Windows 容器映像的清單，附 SHA256 層雜湊
keywords: docker, 容器, 雜湊
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: b2f2d6418fdda2ad0aa0b81c05efad6b99f74375
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107902"
---
# <a name="container-base-images"></a>容器基底影像

## <a name="supported-base-images"></a>支援的基本影像

Windows 容器提供四個容器基底影像： Windows Server Core、Nano Server、Windows 和 IoT 核心。 並非所有的設定都支援這兩種作業系統映像。 此表詳加說明所支援的設定。

|主機作業系統|Windows 容器|Hyper-V 隔離|
|---------------------|-----------------|-----------------|
|Windows Server 2016 或 Windows Server 2019 （標準或資料中心）|伺服器核心版、Nano Server、Windows|伺服器核心版、Nano Server、Windows|
|Nano Server|Nano Server|伺服器核心版、Nano Server、Windows|
|Windows 10 專業版或 Windows 10 企業版|無法使用|伺服器核心版、Nano Server、Windows|
|IoT 核心版|IoT 核心版|無法使用|

> [!WARNING]  
> 從 Windows Server 版本1709開始，Nano Server 已不再以容器主機的形式提供。

## <a name="base-image-differences"></a>基底影像差異

如何選擇要建立的正確基礎影像？ 雖然您可以隨意建立任何您想要的專案，但以下是每個影像的一般指導方針：

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore)：如果您的應用程式需要完整的 .net 架構，這是要使用的最佳影像。
- [Nano server](https://hub.docker.com/_/microsoft-windows-nanoserver)：針對只需要 .net Core 的應用程式，Nano server 將提供更精簡的影像。
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)：您可能會發現您的應用程式取決於伺服器核心或 Nano server 影像（例如 GDI 文件庫）中遺失的元件或 .dll。 此影像會攜帶完整的 Windows 相依性集合。
- [IoT 核心](https://hub.docker.com/_/microsoft-windows-iotcore)版：此影像是專為[IoT 應用程式](https://developer.microsoft.com/windows/iot)所建立的。 在以 IoT 核心主機為目標時，您應該使用這個容器影像。

對於大部分的使用者，Windows Server Core 或 Nano Server 將是最適合使用的影像。 在您考慮在 Nano Server 上建立時，請記住下列事項：

- 已移除服務堆疊
- 不包含 .NET Core (不過您可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/))
- 已移除 PowerShell
- 已移除 WMI
- 從 Windows Server 版本 1709 開始，應用程式會在使用者內容下執行，所以需要系統管理員權限的命令將會失敗。 您可以透過使用者標誌（例如 docker 執行-user ContainerAdministrator）指定容器系統管理員帳戶，但我們將來想要從 NanoServer 中完整移除系統管理員帳戶。

這些是最大的差異，但並非完整的清單。 還有其他未包含的元件並未註明。 請記住，您隨時都可以視需要在 Nano Server 之上新增層級。 如需範例，請參閱 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)。
