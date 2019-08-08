---
title: Windows 容器中的列印多工緩衝處理器
description: 說明 Windows 容器中的列印多工緩衝處理器服務目前的工作行為
keywords: docker、容器、印表機、多工緩衝處理程式
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999095"
---
# <a name="print-spooler-in-windows-containers"></a>Windows 容器中的列印多工緩衝處理器

在列印服務上有相依相依性的應用程式, 可以使用 Windows 容器成功建立容器。 必須符合一些特殊需求, 才能成功啟用印表機服務功能。 本指南說明如何正確地設定您的部署。

> [!IMPORTANT]
> 在容器中成功存取列印服務時, 功能在表單中受到限制;某些與列印相關的動作可能無法運作。 例如, 由於不**支援從容器中安裝驅動程式**, 因此無法以容器方式將印表機驅動程式安裝到主機的 app。 如果您發現您想要在容器中支援的列印功能不受支援, 請開啟下方的意見反應。

## <a name="setup"></a>設定

* 主機應該是 Windows Server 2019 或 Windows 10 專業版/企業版2018更新或更新版本。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)影像應該是目標的基底影像。 其他 Windows 容器基底影像 (例如 Nano Server 和 Windows Server Core) 不會攜帶列印伺服器角色。

### <a name="hyper-v-isolation"></a>Hyper-V 隔離

我們建議您使用 Hyper-v 隔離來執行容器。 在此模式下執行時, 您可以有多個您想要執行列印服務存取的容器。 您不需要修改主機上的多工緩衝處理程式服務。

您可以使用下列 PowerShell 查詢驗證功能:

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

由於進程隔離容器的共用內核本質, 目前的行為限制使用者只能在整個主機及其所有容器子網站上執行印表機多工緩衝處理程式服務的**一個實例**。 如果主機上的印表機列印多工緩衝處理器正在執行, 您必須先停止主機上的服務, 然後 attemping 才能在來賓中啟動印表機服務。

> [!TIP]
> 如果您同時在容器與主機中同時啟動某個多工緩衝處理程式服務的容器和查詢, 這兩者都會將其狀態報表為「正在執行」。 但不要 deceived--容器將無法查詢可用印表機的清單。 主機的多工緩衝處理程式服務不能執行。 

若要檢查主機是否正在執行印表機服務, 請使用下列在 PowerShell 中的查詢:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

若要在主機上停止列印多工緩衝處理器服務, 請在下面的 PowerShell 中使用下列命令:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

啟動容器並驗證對印表機的存取權。

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