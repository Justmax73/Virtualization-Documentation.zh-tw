---
title: "Hypervisor 規格"
description: "Hypervisor 規格"
keywords: windows 10, hyper-v
author: theodthompson
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: a2a6727289b5c1ecea6ce863d78d4edbb3426b82
ms.sourcegitcommit: 04563fde5017e8d9e8b8ab2bbce4bf2bdf29b419
ms.translationtype: HT
ms.contentlocale: zh-TW
---
# <a name="hypervisor-specifications"></a>Hypervisor 規格

## <a name="hypervisor-top-level-functional-specification"></a>Hypervisor 功能規格概要

Hyper-V Hypervisor 功能規格概要 (TLFS) 描述其他作業系統元件看得見的 Hypervisor 外部行為。 此規格適用於客體作業系統開發人員。
  
> 此規格受 Microsoft 開放規格承諾的規範。  閱讀下列內容可進一步了解 [Microsoft 開放規格承諾](https://msdn.microsoft.com/en-us/openspecifications)的詳細資料。  

#### <a name="download"></a>下載
版本 | 文件
--- | ---
Windows Server 2016 (修訂版 B) | [Hypervisor Top Level Functional Specification v5.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0b.pdf)
Windows Server 2012 R2 (修訂版 B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>實作 Microsoft Hypervisor 介面的需求

Windows 作業系統需要一組有限的 Hypervisor 介面，以在客體虛擬機器 (也稱為 "HV#1" 介面) 中執行。 此外，與 Microsoft 相容的 Hypervisor 可以實作數個選擇性功能。 這些選項會變更虛擬機器中 Windows 的行為。 ＜實作 Microsoft Hypervisor 介面的需求＞說明由 Microsoft 相容的 Hypervisor 實作的必要和選擇性功能。

#### <a name="download"></a>下載

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)