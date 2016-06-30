---
title: "Windows 容器中的應用程式相容性"
description: "Windows 容器中的應用程式相容性。"
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3e524458-bd03-400e-913f-210335add8dc
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: 2830dc81317311dd54dfcca45251b668f3d2cc29

---

# Windows 容器中的應用程式相容性

**這是初版內容，後續可能會變更。** 

本文是產品預覽。  雖然執行於 Windows 上的應用程式終究也可在容器中執行，但我們不妨趁此機會檢視一下目前應用程式的相容情形。

本文只是為了分享我們的體驗。

此清單中是否遺漏了什麼？  經由[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)可讓我們了解您環境中成功與失敗的情況。

## Windows Server 容器

我們嘗試在 Windows Server 容器中執行了下列應用程式。  這些結果不保證應用程式正常運作。

| **Name** | **版本** | **Windows Server Core 基本映像** | **Nano Server 基本映像** | **註解** |
|:-----|:-----|:-----|:-----|:-----|
| .NET | 3.5 | [是] | Unknown |  | 
| .NET | 4.6 | [是] | Unknown |  | 
| .NET CLR | 5 beta 6 | [是] | [是]| 兩者，x64 和 x86 | 
| Active Python | 3.4.1 | [是] | [是] | |
| Apache Cassandra || [是] | 不明 | 
| Apache CouchDB | 1.6.1 | 否 | 否 | |
| Apache Hadoop | | [是] | 否 | |
| Apache HTTPD | 2.4 | [是] | [是] | 如果載入 dedup 篩選器，VC++ 執行階段就無法安裝。 使用下列項目卸載 dedup `fltmc unload dedup` |
| Apache Tomcat | 8.0.24 x64 | [是] | Unknown | |
| ASP.NET | 4.6 | [是] | 不明 | |
| ASP.NET | 5 beta 6 | [是] | [是] | 兩者，x64 和 x86 |
| Django | |[是]|[是]| |
| Go | 1.4.2 | [是] | [是] | |
| Internet Information Service | 10.0 | [是] | 是 | HTTPS/TLS 無法運作。  如果載入 dedup 篩選器，VC++ 執行階段就無法安裝。 使用下列項目卸載 dedup `fltmc unload dedup` |
| Java | 1.8.0_51 | [是] | [是] | 使用伺服器版本。 用戶端版本未正確安裝 |
| MongoDB | 3.0.4 | [是] | 不明 | |
| MySQL | 5.6.26 | [是] | [是] | |
| NGinx | 1.9.3 | [是] | [是] | |
| Node.js | 0.12.6 | 部分 | 部分 | NPM 無法下載封裝。 |
| Perl | | [是] | 不明 | |
| PHP | 5.6.11 | [是] | [是] | 如果載入 dedup 篩選器，VC++ 執行階段就無法安裝。 使用下列項目卸載 dedup `fltmc unload dedup` |
| PostgreSQL | 9.4.4 | [是] | Unknown | 如果載入 dedup 篩選器，VC++ 執行階段就無法安裝。 使用下列項目卸載 dedup `fltmc unload dedup` |
| Python | 3.4.3 | [是] | [是] | |
| R | 3.2.1 | 否 | 否 | |
| RabbitMQ | 3.5.x | [是] | Unknown | |
| Redis | 2.8.2101 | [是] | [是] | |
| Ruby | 2.2.2 | [是] | [是] | 兩者，x64 和 x86 | 
| Ruby on Rails | 4.2.3 | [是] | [是] | |
| SQLite | 3.8.11.1 | [是] | 否 | |
| SQL Server Express | 2014 | 是 | 未知 | 這個[由社群參與編輯的 Dockerfile](https://github.com/brogersyh/Dockerfiles-for-windows/tree/master/sqlexpress) 會安裝 SQL Express 2014，您可以加以建置以便快速開始。 |
| Sysinternals Tools | * | 是 | [是] | 僅試用不需要 GUI 的部分。 PsExec 在目前的設計下無法運作 | 

## Hyper-V 容器

我們嘗試在 Hyper-V 容器中執行了下列應用程式。  這些結果不保證應用程式正常運作。

| **Name** | **版本** | **Nano Server 基本映像** | **註解** |
|:-----|:-----|:-----|:-----|
| Apache Hadoop | | 否 | |
| Apache HTTPD | 2.4 | [是] | 如果載入 dedup 篩選器，VC++ 執行階段就無法安裝。 使用下列項目卸載 dedup `fltmc unload dedup` |
| ASP.NET | 5 beta 6 | [是] | 兩者，x64 和 x86 |
| Django |  | [是] | 如果使用 DockerFile 建立映像，並將 python 二進位檔複製於其中，Python 即無法運作。 啟動容器，然後複製 python 二進位檔。 |
| Go | 1.4.2 | [是] | |
| Internet Information Service | 10.0 | 是 | HTTPS/TLS 無法運作。  IIS 無法直接使用 dism 進行安裝。  使用 dism 命令執行 IIS 的自動安裝。 |
| Java | 1.8.0_51 | [是] | 使用伺服器版本。 用戶端版本未正確安裝 |
| MySQL | 5.6.26 | [是] | |
| NGinx | 1.9.3 | [是] | |
| Node.js | 0.12.6 | 部分 | NPM 無法下載封裝。 |
| Python | 3.4.3 | [是] |  如果使用 DockerFile 建立映像，並將 python 二進位檔複製於其中，Python 即無法運作。 啟動容器，然後複製 python 二進位檔。 |
| Redis | 2.8.2101 | [是] | |
| Ruby | 2.2.2 | [是] | 兩者，x64 和 x86 | 
| Ruby on Rails | 4.2.3 | [是] | |
| Sysinternals Tools | | [是] | 僅試用不需要 GUI 的部分。 PsExec 在目前的設計下無法運作。 |

## 請與我們分享您的體驗
此清單中是否遺漏了什麼？  經由[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)可讓我們了解您環境中成功與失敗的情況。



<!--HONumber=Jun16_HO4-->


