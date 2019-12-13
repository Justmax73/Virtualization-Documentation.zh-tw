---
title: Windows 容器需求
description: Windows 容器需求。
keywords: 中繼資料, 容器
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 74f501e5efab3a93e60c9d4797464cea283cdc0b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910498"
---
# <a name="windows-container-requirements"></a>Windows 容器需求

本指南列出 Windows 容器主機的需求。

## <a name="operating-system-requirements"></a>作業系統需求

- Windows 容器功能適用于 Windows Server （半年通道）、Windows Server 2019、Windows Server 2016 和 Windows 10 Professional 和 Enterprise Edition （1607版和更新版本）。
- 執行 Hyper-v 隔離之前，必須先安裝 Hyper-v 角色
- Windows Server 容器主機必須將 Windows 安裝至 c:\。 如果只會部署 Hyper-v 隔離的容器，則不適用此限制。

## <a name="virtualized-container-hosts"></a>虛擬化的容器主機

如果 Windows 容器主機將會從 Hyper-v 虛擬機器執行，而且也會裝載 Hyper-v 隔離，則必須啟用嵌套虛擬化。 巢狀的虛擬化的需求如下：

- 至少有 4 GB RAM 可供虛擬化的 HYPER-V 主機使用。
- 主機系統上的 windows Server （半年通道）、Windows Server 2019、Windows Server 2016 或 Windows 10;和 Windows Server （完整或伺服器核心）在虛擬機器中。
- Intel VT-x 的處理器 (這項功能目前只適用於 Intel 處理器)。
- 容器主機 VM 也會需要至少兩個虛擬處理器。

### <a name="memory-requirements"></a>記憶體需求

可供容器使用的記憶體限制可透過[資源控制項](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls)或多載容器主機來進行設定。  以下列出啟動容器和執行基本命令所需的最小記憶體數量（ipconfig、dir 等等）。

>[!NOTE]
>這些值不會將容器或需求之間的資源分享，從容器中執行的應用程式納入考慮。  例如，有 512MB 可用記憶體的主機可以在 Hyper-V 隔離下執行多個 Server Core 容器，因為這些容器會共用資源。

#### <a name="windows-server-2016"></a>Windows Server 2016

| Base image  | Windows Server 容器 | Hyper-V 隔離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB 分頁檔 |
| Server Core | 50 MB                     | 325 MB + 1 GB 分頁檔 |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server (半年通道)

| Base image  | Windows Server 容器 | Hyper-V 隔離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB 分頁檔 |
| Server Core | 45 MB                     | 360 MB + 1 GB 分頁檔 |

## <a name="see-also"></a>請參閱

[內部部署案例中 Windows 容器和 Docker 的支援原則](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)