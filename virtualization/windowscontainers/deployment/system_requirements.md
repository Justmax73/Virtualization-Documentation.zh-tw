---
title: "Windows 容器需求"
description: "Windows 容器需求。"
keywords: metadata, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: cc216f56acd5e547d05a48beea57450ba5fcb28b
ms.openlocfilehash: 12ae565f012dc87a2cab883c0486322db42b1dcc

---

# Windows 容器需求

**這是初版內容，後續可能會變更。** 

本指南列出 Windows 容器主機的需求。

## 作業系統需求

- Windows 容器角色只在 Windows Server 2016 TP5 (完整版及核心版)、Nano Server 和 Windows 10 (測試人員組建 14352 及更高版本) 上供使用。
- 必須安裝 Hyper-V 角色才可執行 Hyper-V 容器。
- Windows Server 容器主機必須將 Windows 安裝到 c:\\。 如果僅會部署 Hyper-V 容器，則不適用這項限制。

## 虛擬化的容器主機

如果 Windows 容器主機將會在 Hyper-V 虛擬機器上執行，而且也會主控 Hyper-V 容器，就必須啟用巢狀虛擬化。 巢狀的虛擬化的需求如下：

- 至少有 4 GB RAM 可供虛擬化的 HYPER-V 主機使用。
- Windows Server 2016 Technical Preview 5 或主機系統上的 Windows 10 組建 10565 ，以及 Windows Server Technical Preview 5 (完整版、核心版) 或是虛擬機器上的 Nano Server。
- Intel VT-x 的處理器 (這項功能目前只適用於 Intel 處理器)。
- 容器主機 VM 也將會需要至少 2 部虛擬處理器。

## 支援的作業系統映像

Windows Server Technical Preview 5 提供兩種容器作業系統映像 (Windows Server Core 和 Nano Server)。 並非所有的設定都支援這兩種作業系統映像。 此表詳加說明所支援的設定。

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
<td><center>Windows Server 2016 完整 UI</center></td>
<td><center>Server Core 映像</center></td>
<td><center>Nano Server 映像</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core 映像</center></td>
<td><center> Nano Server 映像</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Nano Server 映像</center></td>
<td><center>Nano Server 映像</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 測試人員版本</center></td>
<td><center>無法使用</center></td>
<td><center>Nano Server 映像</center></td>
</tr>
</tbody>
</table>



<!--HONumber=Jun16_HO4-->


