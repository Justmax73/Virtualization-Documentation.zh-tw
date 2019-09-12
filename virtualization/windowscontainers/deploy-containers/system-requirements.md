---
title: Windows 容器需求
description: Windows 容器需求。
keywords: 中繼資料, 容器
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: df5d8e17d0d512f7f53fcacf6c2c2a2652f3e7c0
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107862"
---
# <a name="windows-container-requirements"></a>Windows 容器需求

本指南列出 Windows 容器主機的需求。

## <a name="os-requirements"></a>作業系統需求

- Windows 容器功能僅適用于 Windows Server 2016 （核心與桌面體驗）、Windows 10 專業版與企業版（周年紀念日）及更新版本。
- 在執行 Hyper-v 隔離之前，必須先安裝 Hyper-v 角色
- Windows Server 容器主機必須將 Windows 安裝至 c:\。 如果只要部署 Hyper-v 隔離的容器，則不適用此限制。

## <a name="virtualized-container-hosts"></a>虛擬化的容器主機

如果 Windows 容器主機將從 Hyper-v 虛擬機器執行，而且也會裝載 Hyper-v 隔離，則必須啟用嵌套的虛擬化。 巢狀的虛擬化的需求如下：

- 至少有 4 GB RAM 可供虛擬化的 HYPER-V 主機使用。
- Windows Server 2019、Windows Server 版本1803、Windows Server 版本1709、Windows Server 2016 或主機系統上的 Windows 10，以及虛擬機器中的 Windows Server （完整版、核心）。
- Intel VT-x 的處理器 (這項功能目前只適用於 Intel 處理器)。
- 容器主機 VM 也至少需要兩個虛擬處理器。

### <a name="memory-requirements"></a>記憶體需求

可供容器使用的記憶體限制可透過[資源控制項](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)或多載容器主機來進行設定。  以下列出啟動容器及執行基本命令（ipconfig、dir 等）所需的最低記憶體量。

>[!NOTE]
>這些值不會考慮在容器中執行的應用程式之間的資源分享，也不會考慮其需求。  例如，有 512MB 可用記憶體的主機可以在 Hyper-V 隔離下執行多個 Server Core 容器，因為這些容器會共用資源。

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
