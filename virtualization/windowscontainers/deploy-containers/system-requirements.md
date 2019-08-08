---
title: Windows 容器需求
description: Windows 容器需求。
keywords: 中繼資料, 容器
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 5fc9b5c9135e87a0d3246952c35c9755e9ad209e
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998465"
---
# <a name="windows-container-requirements"></a>Windows 容器需求

本指南列出 Windows 容器主機的需求。

## <a name="os-requirements"></a>作業系統需求

- Windows 容器功能僅適用于 Windows Server 2016 (核心與桌面體驗)、Windows 10 專業版與企業版 (周年紀念日) 及更新版本。
- 在執行 Hyper-v 隔離之前, 必須先安裝 Hyper-v 角色
- Windows Server 容器主機必須將 Windows 安裝至 c:\。 如果只要部署 Hyper-v 隔離的容器, 則不適用此限制。

## <a name="virtualized-container-hosts"></a>虛擬化的容器主機

如果 Windows 容器主機將從 Hyper-v 虛擬機器執行, 而且也會裝載 Hyper-v 隔離, 則必須啟用嵌套的虛擬化。 巢狀的虛擬化的需求如下：

- 至少有 4 GB RAM 可供虛擬化的 HYPER-V 主機使用。
- Windows Server 2019、Windows Server 版本1803、Windows Server 版本1709、Windows Server 2016 或主機系統上的 Windows 10, 以及虛擬機器中的 Windows Server (完整版、核心)。
- Intel VT-x 的處理器 (這項功能目前只適用於 Intel 處理器)。
- 容器主機 VM 也至少需要兩個虛擬處理器。

## <a name="supported-base-images"></a>支援的基本影像

Windows 容器提供四個容器基底影像: Windows Server Core、Nano Server、Windows 和 IoT 核心。 並非所有的設定都支援這兩種作業系統映像。 此表詳加說明所支援的設定。

|主機作業系統|Windows 容器|Hyper-V 隔離|
|---------------------|-----------------|-----------------|
|Windows Server 2016 或 Windows Server 2019 (標準或資料中心)|伺服器核心版、Nano Server、Windows|伺服器核心版、Nano Server、Windows|
|Nano Server|Nano Server|伺服器核心版、Nano Server、Windows|
|Windows 10 專業版或 Windows 10 企業版|無法使用|伺服器核心版、Nano Server、Windows|
|IoT 核心版|IoT 核心版|無法使用|

> [!WARNING]  
> 從 Windows Server 版本1709開始, Nano Server 已不再以容器主機的形式提供。

### <a name="memory-requirements"></a>記憶體需求

可供容器使用的記憶體限制可透過[資源控制項](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)或多載容器主機來進行設定。  以下列出啟動容器及執行基本命令 (ipconfig、dir 等) 所需的最低記憶體量。

>[!NOTE]
>這些值不會考慮在容器中執行的應用程式之間的資源分享, 也不會考慮其需求。  例如，有 512MB 可用記憶體的主機可以在 Hyper-V 隔離下執行多個 Server Core 容器，因為這些容器會共用資源。

#### <a name="windows-server-2016"></a>Windows Server 2016

| 基底影像  | Windows Server 容器 | Hyper-V 隔離    |
| ----------- | ------------------------ | -------------------- |
| Nano 伺服器 | 40 MB                     | 130 MB + 1 GB 分頁檔 |
| Server Core | 50 MB                     | 325 MB + 1 GB 分頁檔 |

#### <a name="windows-server-version-1709"></a>Windows Server 版本 1709

| 基底影像  | Windows Server 容器 | Hyper-V 隔離    |
| ----------- | ------------------------ | -------------------- |
| Nano 伺服器 | 30 MB                     | 110 MB + 1 GB 分頁檔 |
| Server Core | 45 MB                     | 360 MB + 1 GB 分頁檔 |

### <a name="base-image-differences"></a>基底影像差異

如何選擇要建立的正確基礎影像？ 雖然您可以隨意建立任何您想要的專案, 但以下是每個影像的一般指導方針:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): 如果您的應用程式需要完整的 .net 架構, 這是要使用的最佳影像。
- [Nano server](https://hub.docker.com/_/microsoft-windows-nanoserver): 針對只需要 .net Core 的應用程式, Nano server 將提供更精簡的影像。
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): 您可能會發現您的應用程式取決於伺服器核心或 Nano server 影像 (例如 GDI 文件庫) 中遺失的元件或 .dll。 此影像會攜帶完整的 Windows 相依性集合。
- [IoT 核心](https://hub.docker.com/_/microsoft-windows-iotcore)版: 此影像是專為[IoT 應用程式](https://developer.microsoft.com/windows/iot)所建立的。 在以 IoT 核心主機為目標時, 您應該使用這個容器影像。

對於大部分的使用者, Windows Server Core 或 Nano Server 將是最適合使用的影像。 在您考慮在 Nano Server 上建立時, 請記住下列事項:

- 已移除服務堆疊
- 不包含 .NET Core (不過您可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/))
- 已移除 PowerShell
- 已移除 WMI
- 從 Windows Server 版本 1709 開始，應用程式會在使用者內容下執行，所以需要系統管理員權限的命令將會失敗。 您可以透過使用者標誌 (例如 docker 執行-user ContainerAdministrator) 指定容器系統管理員帳戶, 但我們將來想要從 NanoServer 中完整移除系統管理員帳戶。

這些是最大的差異，但並非完整的清單。 還有其他未包含的元件並未註明。 請記住，您隨時都可以視需要在 Nano Server 之上新增層級。 如需範例，請參閱 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)。