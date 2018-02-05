---
title: "Windows 容器需求"
description: "Windows 容器需求。"
keywords: "中繼資料, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 88d094202c49cf725e9d608a0810e7d9f8a1e271
ms.sourcegitcommit: 7fc79235cbee052e07366b8a6aa7e035a5e3434f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/13/2018
---
# <a name="windows-container-requirements"></a>Windows 容器需求

本指南列出 Windows 容器主機的需求。

## <a name="os-requirements"></a>作業系統需求

- 只有 Windows Server 組建 1709、Windows Server 2016 (Core 與 Desktop 版) 及 Windows 10 專業版與企業版 (Anniversary Edition) 才提供 Windows 容器功能。
- 必須安裝 Hyper-V 角色，才能執行 Hyper-V 容器
- Windows Server 容器主機必須將 Windows 安裝至 c:\。 如果只會部署 Hyper-V 容器，則沒有這項限制。

## <a name="virtualized-container-hosts"></a>虛擬化的容器主機

如果 Windows 容器主機將會在 Hyper-V 虛擬機器上執行，而且也會主控 Hyper-V 容器，就必須啟用巢狀虛擬化。 巢狀的虛擬化的需求如下：

- 至少有 4 GB RAM 可供虛擬化的 HYPER-V 主機使用。
- Windows Server 組建 1709、Windows Server 2016 或主機系統上的 Windows 10，以及虛擬機器上的 Windows Server (完整版、核心版)。
- Intel VT-x 的處理器 (這項功能目前只適用於 Intel 處理器)。
- 容器主機 VM 也將會需要至少 2 部虛擬處理器。

## <a name="supported-base-images"></a>支援的基本映像

Windows 容器隨附兩個容器基本映像：Windows Server Core 與 Nano Server。 並非所有的設定都支援這兩種作業系統映像。 此表詳加說明所支援的設定。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>主機作業系統</center></th>
<th><center>Windows Server 容器</center></th>
<th><center>Hyper-V 容器</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 (Standard 或 Datacenter)</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 專業版 / 企業版</center></td>
<td><center>無法使用</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">從 Windows Server 版本 1709 開始，Nano Server 無法再當做容器主機使用。</span>


### <a name="memory-requirments"></a>記憶體需求
可供容器使用的記憶體限制可透過[資源控制項](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)或多載容器主機來進行設定。  啟動容器及執行基本命令 (ipconfig、dir 等等) 所需的記憶體數量下限列於底下。  __請注意，這些值並未考量容器之間的資源共用或是在容器內執行之應用程式的需求。  例如，有 512MB 可用記憶體的主機可以在 Hyper-V 隔離下執行多個 Server Core 容器，因為這些容器會共用資源。__

#### <a name="windows-server-2016"></a>Windows Server 2016
| 基本映像  | Windows Server 容器 | Hyper-V 隔離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB 分頁檔 |
| Server Core | 50 MB                     | 325 MB + 1 GB 分頁檔 |

#### <a name="windows-server-version-1709"></a>Windows Server 版本 1709
| 基本映像  | Windows Server 容器 | Hyper-V 隔離    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB 分頁檔 |
| Server Core | 45 MB                     | 360 MB + 1 GB 分頁檔 |


### <a name="nano-server-vs-windows-server-core"></a>Nano Server 與 Windows Server Core

該選擇 Windows Server Core 還是 Nano Server 呢？ 雖然您可以隨意使用任何一種來建置，不過如果您認為應用程式需要與 .NET Framework 完全相容，那就應該使用 [Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/)。 反之，如果您的應用程式是針對雲端打造而且使用 .NET Core，那就應該使用 [Nano Server](https://hub.docker.com/r/microsoft/nanoserver/)。 這是因為 Nano Server 的設計宗旨是要盡可能減少磁碟使用量，因此移除了許多非必要的程式庫。 在您考慮是否要以 Nano Server 做為建置基礎時，最好將下列各點列入考量：

- 已移除服務堆疊
- 不包含 .NET Core (不過您可以使用 [.NET Core Nano Server 映像](https://hub.docker.com/r/microsoft/dotnet/))
- 已移除 PowerShell
- 已移除 WMI

這些是最大的差異，但並非完整的清單。 還有其他未包含的元件並未註明。 請記住，您隨時都可以視需要在 Nano Server 之上新增層級。 如需範例，請參閱 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile)。

