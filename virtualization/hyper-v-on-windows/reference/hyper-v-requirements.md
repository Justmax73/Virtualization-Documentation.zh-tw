---
title: Windows 10 Hyoer-V 系統需求
description: Windows 10 Hyoer-V 系統需求
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: 51720a92cd04cfae25c7d0c010ee7b441073dd7e
ms.sourcegitcommit: 5e5644bff6dba70e384db6c80787b3bbe7adb93c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "4303914"
---
# <a name="windows-10-hyper-v-system-requirements"></a>Windows 10 Hyper-V 系統需求

HYPER-V 是 64 位元版本的 Windows 10 專業版、 企業版和教育版中提供。 Hyper-V 需要第二層位址轉譯 (SLAT) -- Intel 和 AMD 當前推出的 64 位元處理器所具備的一項功能。

您可以在具有 4GB RAM 的主機上執行 3 或 4 部基本虛擬機器，但更多虛擬機器會需要更多資源。 反之，您也可以建立具有 32 個處理器和 512GB RAM 的大型虛擬機器，視您的實體硬體而定。

## <a name="operating-system-requirements"></a>作業系統需求

在這些版本的 Windows 10 上可啟用 Hyper-V 角色：

- Windows 10 企業版
- Windows 10 專業版
- Windows 10 Education

Hyper-V 角色**無法**安裝在：

- Windows 10 Home
- Windows 10 Mobile
- Windows10 行動裝置企業版

>Windows 10 home 可以升級到 Windows 10 專業版。 若要這麼做，請開啟 **\[設定\]** > **\[更新和安全性\]** > **\[啟用\]**。 您可以在此瀏覽市集並購買升級。

## <a name="hardware-requirements"></a>硬體需求

這份文件未提供與 Hyper-V 相容硬體的完整清單，但下列為必要項目：
    
- 使用第二層位址轉譯 (SLAT) 的 64 位元處理器。
- CPU 對 VM 監視模式延伸模組的支援 (Intel CPU 上的 VT-c)。
- 至少 4 GB 記憶體。 因為虛擬機器與 Hyper-V 主機共用記憶體，所以您必須提供足夠的記憶體來處理預期的虛擬工作負載。

在系統 BIOS 中必須啟用下列項目：
- 虛擬化技術 - 可能有不同的項目名稱，視主機板製造商而定。
- 硬體強制的資料執行防止。

## <a name="verify-hardware-compatibility"></a>確認硬體相容性

若要確認相容性，請開啟 PowerShell 或命令提示字元 (cmd.exe)，然後輸入 **systeminfo**。 如果所有列出的 Hyper-V 需求值皆為 **\[是\]**，您的系統即可執行 Hyper-V 角色。 如有任何項目傳回 **\[否\]**，請檢閱這份文件中列出的需求，並盡可能調整。

![](media/SystemInfo-upd.png)

如果您在現有 Hyper-V 主機上執行 **systeminfo**，\[Hyper-V 需求\] 區段會顯示：

```
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V are not be displayed.
```
