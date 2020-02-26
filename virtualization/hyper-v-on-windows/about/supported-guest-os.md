---
title: 支援的 Windows 客體
description: 支援的 Windows 客體。
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 25c72b910af15fc0b498a5b2abce72d32e6d1efd
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439595"
---
# <a name="supported-windows-guests"></a>支援的 Windows 客體

本文列出在 Windows 上的 Hyper-V 所支援的作業系統組合。  它也會介紹整合服務及支援中的其他因素。

Microsoft 已測試過這些主機/客體組合。  這些組合的問題可能引起「產品支援服務」的注意。

Microsoft 以下列方式提供支援：

* Microsoft 作業系統和整合服務中發現的問題可獲得 Microsoft 支援服務的支援。

* 經過作業系統廠商認證可在 Hyper-V 執行的其他作業系統上發現的問題，由廠商提供支援。

* 若為在其他作業系統中發現的問題，Microsoft 會將問題提交給 [TSANet](http://www.tsanet.org/)，其為多個廠商支援的社群。

若想受到支援，所有作業系統 (客體和主機) 都必須更新至最新狀態。  檢查 Windows Update 是否有重大更新。

## <a name="supported-guest-operating-systems"></a>支援的客體作業系統

| 客體作業系統 |  虛擬處理器數量的上限 | 注意事項 |
|:-----|:-----|:-----|
| Windows 10 | 32 |加強的工作階段模式無法在 Windows 10 家用版上運作 |
| Windows 8.1 | 32 | |
| Windows 8 | 32 ||
| Windows 7 含 Service Pack 1 (SP 1) | 4 | 旗艦版、企業版和專業版版本 (32 位元與 64 位元)。 |
| Windows 7 | 4 | 旗艦版、企業版和專業版版本 (32 位元與 64 位元)。 |
| Windows Vista (含 Service Pack 2，SP2) | 2 | Business、Enterprise 以及 Ultimate，包含 N 與 KN 版本。 |
| - | | |
| [Windows Server 半年通道](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| Windows Server 2019 | 64 | |
| Windows Server 2016 | 64 | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 含 Service Pack 1 (SP 1) | 64 | Datercenter、Enterprise、Standard 以及 Web 版本。 |
| Windows Server 2008 含 Service Pack 2 (SP 2) | 4 | Datacenter、Enterprise、Standard 以及 Web 版本 (32 位元與 64 位元)。 |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Essentials 版本 - 2，Standard 版本 - 4 | |

> 在 Windows 8.1 和 Windows Server 2012 R2 Hyper-V 主機上，Windows 10 可做為客體作業系統執行。

## <a name="supported-linux-and-free-bsd"></a>支援的 Linux 和 Free BSD

| 客體作業系統 |  |
|:-----|:------|
| [CentOS 和 Red Hat Enterprise Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-CentOS-and-Red-Hat-Enterprise-Linux-virtual-machines-on-Hyper-V) | |
| [Hyper-V 上的 Debian 虛擬機器](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Debian-virtual-machines-on-Hyper-V) | |
| [SUSE](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-SUSE-virtual-machines-on-Hyper-V) | |
| [Oracle Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Oracle-Linux-virtual-machines-on-Hyper-V)  | |
| [Ubuntu](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Ubuntu-virtual-machines-on-Hyper-V) | |
| [FreeBSD](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-FreeBSD-virtual-machines-on-Hyper-V) | |

如需詳細資訊，包括舊版 Hyper-V 的支援資訊，請參閱 [Hyper-V 上的 Linux 和 FreeBSD 虛擬機器](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows)。
