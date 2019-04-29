---
title: 在 Windows 容器中列印多工緩衝處理器
description: 說明在 Windows 容器中列印多工緩衝處理器服務目前的工作行為
keywords: docker，容器、 印表機，多工緩衝處理器
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576659"
---
# <a name="print-spooler-in-windows-containers"></a>在 Windows 容器中列印多工緩衝處理器

列印服務的相依性的應用程式可以是容器化成功與 Windows 容器。 有特殊需求，必須符合才能成功啟用印表機服務的功能。 本指南說明如何以正確設定您的部署。

> [!IMPORTANT]
> 表單; 而取得的存取權列印服務成功在容器中運作、 有限的功能某些列印相關的動作可能無法運作。 例如，有安裝印表機驅動程式至在主機上的相依性的應用程式無法被容器化因為**容器內的驅動程式安裝來自不受支援**。 請如果您發現您想要支援在容器中不支援列印功能，開啟下列的意見反應。

## <a name="setup"></a>設定

* 主應用程式應該是 Windows Server 2019 或 Windows 10 專業版/企業版 2018 年 10 月更新或更新版本。
* [Mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows)影像應該是特定對象的基本映像。 其他 Windows 容器基本映像 （例如 Nano Server 和 Windows Server Core） 不會執行列印伺服器角色。

### <a name="hyper-v-isolation"></a>Hyper-V 隔離

我們建議使用 HYPER-V 隔離執行您的容器。 此模式中執行時，您可以有多個容器，因為您想要列印的服務存取執行。 您不需要修改主機上的多工緩衝處理器服務。

您可以使用下列 PowerShell 查詢的功能來驗證：

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

### <a name="process-isolation"></a>處理序隔離

處理序隔離容器共用的核心本質，因為目前的行為會限制使用者能夠透過在主機和容器所有子系執行只有**一個執行個體**的印表機多工緩衝處理器服務。 如果主機有印表機多工緩衝處理器執行，您必須先正試之前啟動，客體中的印表機服務在主機上停止服務。

> [!TIP]
> 如果您啟動容器，並在容器與主機多工緩衝處理器服務查詢的同時，同時將報告其狀態為 'running'。 但不是 deceived-容器將無法查詢可用的印表機清單。 主機的多工緩衝處理器服務必須不會執行。 

若要檢查主應用程式是否正在執行印表機服務，請在下方的 PowerShell 中使用查詢：

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

若要停止在主機上的多工緩衝處理器服務，請在下面的 PowerShell 中使用下列命令：

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

啟動容器，並確認印表機的存取。

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