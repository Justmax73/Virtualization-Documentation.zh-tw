---
title: "支援的 Windows 客體"
description: "支援的 Windows 客體。"
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
translationtype: Human Translation
ms.sourcegitcommit: 645b15f32e731b6d4044e8f66d8ab2374870904c
ms.openlocfilehash: 19ecce49df066c5816741f375c4610b79d7ad802

---

# 支援的 Windows 客體 

本文列出在 Windows 上的 Hyper-V 所支援的作業系統組合。  它也會介紹整合服務及支援中的其他因素。

## 支援是什麼意思？ 
支援代表 Microsoft 已測試過這些主機/客體組合。  這些組合的問題可能引起「產品支援服務」的注意。
 
Microsoft 透過下列方式提供客體作業系統的支援：
* Microsoft 作業系統和整合服務中發現的問題可獲得 Microsoft 支援服務的支援。
* 經過作業系統廠商認證可在 Hyper-V 執行的其他作業系統上發現的問題，由廠商提供支援。
* 若為在其他作業系統中發現的問題，Microsoft 會將問題提交給 [TSANet](http://www.tsanet.org/)，其為多個廠商支援的社群。

為了獲得支援，必須以透過 Windows Update 取得的所有重大更新，更新 Hyper-V 主機和客體。

## 支援的客體作業系統

為了收到支援，必須以透過 Windows Update 取得的所有最新重大更新，更新 Windows 客體作業系統和主機作業系統。

| 客體作業系統 |  虛擬處理器數量的上限 | 附註 | 
|:-----|:-----|:-----|
| Windows 10 | 32 | |
| Windows 8.1 | 32 | |
| Windows 8 | 32 |  |
| Windows 7 含 Service Pack 1 (SP 1) | 4 | 旗艦版、企業版和專業版版本 (32 位元與 64 位元)。 |
| Windows 7 | 4 | 旗艦版、企業版和專業版版本 (32 位元與 64 位元)。 |
| Windows Vista (含 Service Pack 2，SP2) | 2 | Business、Enterprise 以及 Ultimate，包含 N 與 KN 版本。 | 
| - | | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 含 Service Pack 1 (SP 1) | 64 | Datercenter、Enterprise、Standard 以及 Web 版本。 |
| Windows Server 2008 含 Service Pack 2 (SP 2) | 4 | Datacenter、Enterprise、Standard 以及 Web 版本 (32 位元與 64 位元)。 |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Essentials 版本 - 2，Standard 版本 - 4 | |
  
 > 在 Windows 8.1 和 Windows Server 2012 R2 Hyper-V 主機上，Windows 10 可做為客體作業系統執行。

## 支援的 Linux 和 Free BSD

| 客體作業系統 |  |
|:-----|:------|
| [CentOS 和 Red Hat Enterprise Linux ](https://technet.microsoft.com/library/dn531026.aspx) | |
| [Hyper-V 上的 Debian 虛擬機器](https://technet.microsoft.com/library/dn614985.aspx) | |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx) | |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)  | |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx) | |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx) | |

如需舊版 Hyper-V 的支援資訊等詳細資訊，請參閱 [Linux and FreeBSD Virtual Machines on Hyper-V](https://technet.microsoft.com/library/dn531030.aspx) (Hyper-V 上的 Linux 和 FreeBSD 虛擬機器)。



<!--HONumber=Jun16_HO4-->


