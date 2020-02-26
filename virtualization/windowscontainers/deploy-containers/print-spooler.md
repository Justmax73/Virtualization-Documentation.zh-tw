---
title: 在 Windows 容器中列印多工緩衝處理器
description: 說明 Windows 容器中的列印多工緩衝處理器服務目前的工作行為
keywords: docker，容器，印表機，多工緩衝處理器
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439535"
---
# <a name="print-spooler-in-windows-containers"></a>在 Windows 容器中列印多工緩衝處理器

與列印服務有相依性的應用程式可以使用 Windows 容器成功地容器化。 為了成功啟用印表機服務功能，有一些特殊需求必須符合。 本指南說明如何正確設定您的部署。

> [!IMPORTANT]
> 在容器中順利存取列印服務時，功能的形式有限;某些列印相關的動作可能無法正常執行。 例如，相依于將印表機驅動程式安裝到主機的應用程式無法容器化，因為**不支援從容器內進行驅動程式安裝**。 如果您發現您想要在容器中支援的列印功能不受支援，請在下方開啟意見反應。

## <a name="setup"></a>安裝程式

* 主機應該是 Windows Server 2019 或 Windows 10 Pro/企業版2018更新或更新版本。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)映射應該是目標基底映射。 其他 Windows 容器基底映射（如 Nano Server 和 Windows Server Core）則不會包含列印伺服器角色。

### <a name="hyper-v-isolation"></a>Hyper-V 隔離

建議您使用 Hyper-v 隔離來執行您的容器。 在此模式中執行時，您可以擁有多個您想要以存取列印服務的容器。 您不需要修改主機上的多工緩衝處理器服務。

您可以使用下列 PowerShell 查詢來驗證功能：

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>進程隔離

由於進程隔離容器的共用核心本質，因此目前的行為會限制使用者只能在主機及其所有容器子系上執行**一個印表機多**任務緩衝處理器服務實例。 如果主機上有執行中的印表機多工緩衝處理器，則您必須先停止主機上的服務，然後再嘗試啟動來賓中的印表機服務。

> [!TIP]
> 如果您同時啟動容器，並同時查詢容器和主機中的多工緩衝處理器服務，這兩個都會將其狀態報表為「執行中」。 但不是 deceived--容器將無法查詢可用的印表機清單。 主機的多工緩衝處理器服務不得執行。 

若要檢查主機是否正在執行印表機服務，請使用下列 PowerShell 中的查詢：

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

若要停止主機上的多工緩衝處理器服務，請在下列 PowerShell 中使用下列命令：

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

啟動容器並確認對印表機的存取。

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```