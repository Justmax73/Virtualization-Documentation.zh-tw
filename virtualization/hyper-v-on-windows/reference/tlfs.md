---
title: Hypervisor 規格
description: Hypervisor 規格
keywords: windows 10, hyper-v
author: allenma
ms.date: 06/26/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: afbbcf120961081191aaf9051866427c9ce1478e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911178"
---
# <a name="hypervisor-specifications"></a>Hypervisor 規格

## <a name="hypervisor-top-level-functional-specification"></a>Hypervisor 功能規格概要

Hyper-V Hypervisor 功能規格概要 (TLFS) 描述其他作業系統元件看得見的 Hypervisor 外部行為。 此規格適用於客體作業系統開發人員。
  
> 此規格受 Microsoft 開放規格承諾的規範。  閱讀下列內容可進一步了解 [Microsoft 開放規格承諾](https://docs.microsoft.com/openspecs/dev_center/ms-devcentlp/51a0d3ff-9f77-464c-b83f-2de08ed28134)的詳細資料。  

#### <a name="download"></a>下載
發行 | 文件
--- | ---
Windows Server 2016 (修訂版 C) | [虛擬程式管理層頂級功能規格 v 5.0 c .pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2 (修訂版 B) | [Hypervisor 功能規格概要 v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor 功能規格概要 v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor 功能規格概要 v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>實作 Microsoft Hypervisor 介面的需求

TLFS 完整描述 Microsoft 專屬 Hypervisor 架構的各個層面，其宣告來賓虛擬機器為「HV#1」介面。  不過，並非 TLFS 中所述的所有介面都需由希望宣告符合 Microsoft HV#1 Hypervisor 規格的第三方 Hypervisor 實作。 文件「實作 Microsoft Hypervisor 介面的需求」說明必須由聲明符合 Microsoft HV#1 介面之任何 Hypervisor 實作的最低 Hypervisor 介面設定。

#### <a name="download"></a>下載

[實作 Microsoft Hypervisor 介面的需求.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
