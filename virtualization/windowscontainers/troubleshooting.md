---
title: Windows 容器疑難排解
description: 適用於 Windows 容器和 Docker 的疑難排解秘訣、自動化指令碼，以及記錄檔資訊
keywords: docker, 容器, 疑難排解, 記錄檔
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: 16d2794688d60757ef1321d687f6a987ccf0b581
ms.sourcegitcommit: 62fff5436770151a28b6fea2be3a8818564f3867
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2019
ms.locfileid: "10147231"
---
# <a name="troubleshooting"></a>疑難排解

您有設定電腦或執行容器方面的問題嗎？ 我們建立了一份 PowerShell 指令碼，可讓您用於檢查常見的問題。 請嘗試先使用這份指令碼進行檢查，然後將結果提供給我們。

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
指令碼的[讀我檔案](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md)中列有一份清單，包括了此指令碼所執行的各項測試及其常用解決方法。

若執行這份指令碼無法找出問題的根源，請將您的指令碼輸出張貼到[容器論壇](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)。 論壇社群中的各方高手 (包括 Windows 測試人員及開發人員) 是為您提供協助的最佳人選。


## <a name="finding-logs"></a>尋找記錄檔
有多個服務可用來管理 Windows 容器。 接下來的幾節將會指出各項服務記錄檔的位置。

# <a name="docker-engine"></a>Docker 引擎
Docker 引擎會將事件記錄至 Windows 應用程式事件記錄檔，而不是記錄至檔案。 您可以使用 Windows PowerShell，輕鬆讀取、排序和篩選這些記錄檔

比方說，這會顯示 Docker 引擎前 5 分鐘的記錄檔，並從最舊的開始排序。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

您也可以輕鬆透過管道將記錄檔傳送至 CSV 檔案，以供其他工具或試算表讀取。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

## <a name="enabling-debug-logging"></a>啟用偵錯記錄
您也可以對 Docker 引擎啟用偵錯層級記錄。 當標準記錄檔的詳細資料不足時，這可能會很有幫助。

首先，請開啟提升權限的命令提示字元，然後執行 `sc.exe qc docker`，以取得 Docker 服務目前的命令列。
範例：
```
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

取得目前的 `BINARY_PATH_NAME` 之後，請修改如下︰
- 在結尾新增 -D
- 使用 \ 逸出每一個 "
- 使用 " 括住整個命令

接著在執行後接新字串的 `sc.exe config docker binpath=`。 例如： 
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


立即重新啟動 Docker 服務
```
sc.exe stop docker
sc.exe start docker
```

這會記錄更多資訊到應用程式事件記錄檔中。因此，當您完成疑難排解之後，建議最好將 `-D` 選項移除。 使用上述相同的步驟，但移除 `-D`，然後重新啟動服務，以停用偵錯記錄。

上述做法的替代方法是從提高權限的 PowerShell 提示，以偵錯模式執行 Docker 精靈，將輸出直接擷取至檔案中。
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

## <a name="obtaining-stack-dump"></a>取得堆疊轉儲。

一般來說，只有 Microsoft 支援或 docker 開發人員明確要求時，才能使用這種做法。 它可以用來協助診斷螢幕畫面看起來好像已暫停的情況。 

下載 [docker-signal.exe](https://github.com/jhowardmsft/docker-signal)。

使用方式：
```PowerShell
docker-signal --pid=$((Get-Process dockerd).Id)
```

輸出檔案會位於執行中的資料根目錄 docker 中。 預設目錄為 `C:\ProgramData\Docker`。 若要確認實際目錄，可以執行 `docker info -f "{{.DockerRootDir}}"`。

檔案就會是`goroutine-stacks-<timestamp>.log`。

請注意`goroutine-stacks*.log` ，不包含個人資訊。


# <a name="host-compute-service"></a>主機運算服務
Docker 引擎需要 Windows 專用的主機運算服務。 此服務有自己的記錄檔︰ 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

這些記錄檔會顯示在事件檢視器中，也可以使用 PowerShell 進行查詢。

例如：
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```

## <a name="capturing-hcs-analyticdebug-logs"></a>擷取 HCS 分析/偵錯記錄

若要為「Hyper-V 運算」啟用分析/偵錯記錄，並將其儲存至 `hcslog.evtx`。

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

## <a name="capturing-hcs-verbose-tracing"></a>擷取 HCS 詳細追蹤資料

一般而言，這些資料只有在 Microsoft 支援服務要求時才有用。 

下載 [HcsTraceProfile.wprp](https://gist.github.com/jhowardmsft/71b37956df0b4248087c3849b97d8a71)

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

提供 `HcsTrace.etl` 給支援連絡人。
