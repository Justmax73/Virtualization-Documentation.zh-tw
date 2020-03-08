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
ms.openlocfilehash: ebc9be132f05c20eb8daf9b5e6713b9258012305
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853984"
---
# <a name="windows-10-hyper-v-system-requirements"></a>Windows 10 Hyoer-V 系統需求

Hyper-v 可在64位版本的 Windows 10 專業版、企業版和教育版中使用。 Hyper-V 需要第二層位址轉譯 (SLAT) -- Intel 和 AMD 當前推出的 64 位元處理器所具備的一項功能。

您可以在具有 4GB RAM 的主機上執行 3 或 4 部基本虛擬機器，但更多虛擬機器會需要更多資源。 反之，您也可以建立具有 32 個處理器和 512GB RAM 的大型虛擬機器，視您的實體硬體而定。

## <a name="operating-system-requirements"></a>作業系統需求

在這些版本的 Windows 10 上可啟用 Hyper-V 角色：

- Windows 10 Enterprise
- Windows 10 Pro
- Windows 10 Education

Hyper-V 角色**無法**安裝在：

- Windows 10 Home
- Windows 10 Mobile
- Windows 10 Mobile Enterprise

>Windows 10 Home edition 可以升級至 Windows 10 專業版。 若要這麼做，請開啟 **\[設定\]**  >  **\[更新和安全性\]**  >  **\[啟用\]** 。 您可以在此瀏覽市集並購買升級。

## <a name="hardware-requirements"></a>硬體需求

這份文件未提供與 Hyper-V 相容硬體的完整清單，但下列為必要項目：

- 使用第二層位址轉譯 (SLAT) 的 64 位元處理器。
- VM 監視器模式擴充功能的 CPU 支援（Intel CPU 上的 VT-x）。
- 至少 4 GB 記憶體。 因為虛擬機器與 Hyper-V 主機共用記憶體，所以您必須提供足夠的記憶體來處理預期的虛擬工作負載。

在系統 BIOS 中必須啟用下列項目：
- 虛擬化技術 - 可能有不同的項目名稱，視主機板製造商而定。
- 硬體強制的資料執行防止。

## <a name="verify-hardware-compatibility"></a>確認硬體相容性

檢查過上述作業系統和硬體需求之後，請開啟 PowerShell 會話或命令提示字元（cmd.exe）視窗，輸入**systeminfo**，然後檢查 [hyper-v 需求] 區段，以確認 Windows 中的硬體相容性。 如果所有列出的 Hyper-V 需求的值皆為[Yes]，則您的系統可執行 Hyper-V 角色。 如有任何項目傳回 **[否]** ，請檢閱這份文件中列出的需求，並盡可能調整。

![](media/SystemInfo-upd.png)

## <a name="final-check"></a>最後檢查

如果符合所有作業系統、硬體和相容性需求，您會在 [控制台] 中看到**hyper-v** **：開啟或關閉 Windows 功能**，它會有2個選項。

1. Hyper-v 平臺
1. Hyper-V 管理工具

![](media/hyper_v_feature_screenshot.png)

> [!NOTE] 如果您在 [控制台] 中看到**Windows 虛擬機器平臺**，而不是**hyper-v** **：開啟 > 或關閉系統上的 windows 功能**可能與 hyper-v 不相容，然後交叉檢查上述需求。
>
>如果您在現有 Hyper-V 主機上執行 **systeminfo**，\[Hyper-V 需求\] 區段會顯示：
>
>```
>Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V will not be displayed.
>```
