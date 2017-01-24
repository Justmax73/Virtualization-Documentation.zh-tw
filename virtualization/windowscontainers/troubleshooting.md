---
title: "Windows 容器疑難排解"
description: "適用於 Windows 容器和 Docker 的疑難排解秘訣、自動化指令碼，以及記錄檔資訊"
keywords: "docker, 容器, 疑難排解, 記錄檔"
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
translationtype: Human Translation
ms.sourcegitcommit: c65353d0b6dff233819dcc8f4f92eb186bf3b8fc
ms.openlocfilehash: 9f28c35c6eaddd8bcf3883863b63251378f845a7

---

# 疑難排解

您有設定電腦或執行容器方面的問題嗎？ 我們建立了一份 PowerShell 指令碼，可讓您用於檢查常見的問題。 請嘗試先使用這份指令碼進行檢查，然後將結果提供給我們。

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
指令碼的[讀我檔案](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md)中列有一份清單，包括了此指令碼所執行的各項測試及其常用解決方法。

若執行這份指令碼無法找出問題的根源，請將您的指令碼輸出張貼到[容器論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)。 論壇社群中的各方高手 (包括 Windows 測試人員及開發人員) 是為您提供協助的最佳人選。


## 尋找記錄檔
有多項服務可以用於管理 Windows 容器。 接下來的幾節將會指出各項服務記錄檔的位置。

### Docker Engine
Docker 引擎會將事件記錄至 Windows 應用程式事件記錄檔，而不是記錄至檔案。 您可以使用 Windows PowerShell，輕鬆讀取、排序和篩選這些記錄檔

比方說，這會顯示 Docker 引擎前 5 分鐘的記錄檔，並從最舊的開始排序。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

您也可以輕鬆透過管道將記錄檔傳送至 CSV 檔案，以供其他工具或試算表讀取。

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### 啟用偵錯記錄
您也可以對 Docker Engine 啟用偵錯層級記錄。 當標準記錄檔的詳細資料不足時，這可能會很有幫助。

首先，請開啟提升權限的命令提示字元，然後執行 `sc.exe qc docker`，以取得 Docker 服務目前的命令列。
範例：
```none
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

接著在執行後接新字串的 `sc.exe config docker binpath= `。 例如： 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


立即重新啟動 Docker 服務
```none
sc.exe stop docker
sc.exe start docker
```

這會記錄更多資訊到應用程式事件記錄檔中。因此，當您完成疑難排解之後，建議最好將 `-D` 選項移除。 使用上述相同的步驟但移除 `-D`，然後重新啟動服務，以停用偵錯記錄。


### Host Container Service
Docker Engine 需要 Windows 專用的 Host Container Service。 此服務有自己的記錄檔︰ 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

這些記錄檔會顯示在事件檢視器中，也可以使用 PowerShell 進行查詢。

例如：
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```




<!--HONumber=Jan17_HO4-->


