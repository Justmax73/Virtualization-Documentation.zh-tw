---
title: "Windows 容器需求"
description: "Windows 容器需求。"
keywords: "中繼資料, 容器"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: d4594bd5efe3e1852f6a47b6474bf9ea56196c14
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/21/2017
---
# Windows 容器需求

本指南列出 Windows 容器主機的需求。

## 作業系統需求

- 只有 Windows Server 2016 (Core 與 Desktop 版)、Nano Server 及 Windows 10 專業版與企業版 (Anniversary Edition) 才提供 Windows 容器功能。
- 必須安裝 Hyper-V 角色，才能執行 Hyper-V 容器
- Windows Server 容器主機必須將 Windows 安裝至 c:\。 如果只會部署 Hyper-V 容器，則沒有這項限制。

## 虛擬化的容器主機

如果 Windows 容器主機將會在 Hyper-V 虛擬機器上執行，而且也會主控 Hyper-V 容器，就必須啟用巢狀虛擬化。 巢狀的虛擬化的需求如下：

- 至少有 4 GB RAM 可供虛擬化的 HYPER-V 主機使用。
- Windows Server 2016 或主機系統上的 Windows 10，以及 Windows Server (完整版、核心版) 或是虛擬機器上的 Nano Server。
- Intel VT-x 的處理器 (這項功能目前只適用於 Intel 處理器)。
- 容器主機 VM 也將會需要至少 2 部虛擬處理器。

## 支援的基本映像

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
<td><center>Nano Server</center></td>
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

## 使容器主機版本與容器映像版本相符
### Windows Server 容器
Windows Server 容器和基礎主機共用單一核心，因此容器的基本映像必須與主機相符。  如果版本不同，容器仍可啟動，但無法保證可使用完整功能。 因此不支援不相符的版本。  Windows 作業系統有 4 個層級的版本設定：主要、次要、組建和修訂，例如 10.0.14393.0。 只有在發行新的 OS 版本時，組建編號才會變更。 修訂編號會隨著套用 Windows 更新時進行更新。 組建編號不同時，Windows Server 容器便無法啟動，例如 10.0.14300.1030 (Technical Preview 5) 和 10.0.14393 (Windows Server 2016 RTM)。 如果組建編號相符但修訂編號不同，仍可啟動，例如 10.0.14393 (Windows Server 2016 RTM) 和 10.0.14393.206 (Windows Server 2016 GA)。 雖然技術上仍可啟動，但該設定可能無法在所有情況下正常運作，因此無法用於生產環境支援。 

若要檢查已安裝的 Windows 主機版本為何，您可以查詢 HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion。  若要檢查基本映像使用的版本為何，您可以檢閱 Docker 中樞的標籤，或是映像描述中提供的映像雜湊表。  [Windows 10 更新歷程記錄](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)頁面上列出每個組建與修訂發行的時間。

在此範例中，14393 是主要組建編號，而 321 為修訂編號。
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### 容器的 Hyper-V 隔離
Windows 容器可以在使用或不使用 Hyper-V 隔離的情況下執行。  Hyper-V 隔離會使用最佳化的 VM，在容器周圍建立安全的界限。  每個 Hyper-V 隔離的容器都具有自身的 Windows 核心執行個體，不像標準 Windows 容器會和其他容器與主機共用核心。  因此，您可以在容器主機和映像中使用不同的 OS 版本 (請參閱下面的相容性對照表)。  

若要使用 Hyper-V 隔離來執行容器，只要將 "--isolation=hyper-v" 標記新增至您的 docker run 命令即可。

### 相容性對照表
2016 GA (10.0.14393.206) 之後的 Windows Server 組建都可以在受支援的設定中，不受修訂編號的限制執行 Windows Server Core 或 Nano Server 的 Windows Server 2016 GA 映像。    

請務必了解，為了使用 Windows 更新所提供的完整功能、可靠性和安全性保證，您應該在所有系統上皆維持最新的版本。  
