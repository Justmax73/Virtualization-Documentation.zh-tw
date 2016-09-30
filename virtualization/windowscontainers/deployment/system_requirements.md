---
title: "Windows 容器需求"
description: "Windows 容器需求。"
keywords: "中繼資料, 容器"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: a17a0e4eca2596c8cebbef93ea91c854f09d0b76

---

# Windows 容器需求

本指南列出 Windows 容器主機的需求。

## 作業系統需求

- 只有 Windows Server 2016 (Core 與 Desktop 版)、Nano Server 及 Windows 10 Professional 與 Enterprise (Anniversary Edition) 才提供 Windows 容器功能。
- 必須安裝 Hyper-V 角色才可執行 Hyper-V 容器。
- Windows Server 容器主機必須已將 Windows 安裝於 c:\.。如果只會部署 Hyper-V，這項限制即不適用。

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
<td><center>Windows Server 2016 與 Desktop</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
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



<!--HONumber=Sep16_HO4-->


