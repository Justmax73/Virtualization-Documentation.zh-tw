---
title: Windows 10 上的 Hyper-V 疑難排解
description: Windows 10 上的 Hyper-V 疑難排解
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
ms.openlocfilehash: bdb9feeb2452f2784a3b814e85dc72f3b967a9d3
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998865"
---
# <a name="troubleshoot-hyper-v-on-windows-10"></a>Windows 10 上的 Hyper-V 疑難排解

## <a name="i-updated-to-windows-10-and-now-i-cant-connect-to-my-downlevel-windows-81-or-server-2012-r2-host"></a>我已更新為 Windows 10，現在我無法連線到我的舊版 (Windows 8.1 或 Server 2012 R2) 主機
在 Windows 10 中，Hyper-V 管理員因為遠端管理而移到 WinRM。  這意思是說，現在遠端主機上必須啟用遠端管理，才能使用 Hyper-V 管理員來管理它。

如需詳細資訊，請參閱[管理遠端 Hyper-V 主機](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Remotely-manage-Hyper-V-hosts)

## <a name="i-changed-the-checkpoint-type-but-it-is-still-taking-the-wrong-type-of-checkpoint"></a>我變更檢查點的類型，但仍建立的檢查點類型錯誤
如果您從 VMConnect 建立檢查點，並在 Hyper-V 管理員中變更檢查點類型，最後建立的檢查點類型會是開啟 VMConnect 時指定的檢查點類型。

關閉再重新啟動 VMConnect，才會建立正確類型的檢查點。

## <a name="when-i-try-to-create-a-virtual-hard-disk-on-a-flash-drive-an-error-message-is-displayed"></a>當我嘗試在快閃磁碟機上建立虛擬硬碟時，顯示錯誤訊息
Hyper-V 不支援 FAT/FAT32 格式的磁碟機，因為這些檔案系統不提供存取控制清單 (ACL)，而且不支援大於 4 GB 的檔案。 ExFAT 格式的磁碟只提供有限的 ACL 功能，因此基於安全性考量也不受支援。
在 PowerShell 中顯示的錯誤訊息是「系統無法建立 '\[path to VHD\]': 無法完成要求的作業，因為檔案系統限制 (0x80070299)」。

請改用 NTFS 格式的磁碟機。 

## <a name="i-get-this-message-when-i-try-to-install-hyper-v-cannot-be-installed-the-processor-does-not-support-second-level-address-translation-slat"></a>當我嘗試安裝時收到這則訊息：「無法安裝 Hyper-V: 處理器不支援第二層位址轉譯 (SLAT)。」
Hyper-V 需要 SLAT 才能執行虛擬機器。 如果您的電腦不支援 SLAT，則不能成為虛擬機器的主機。

如果您只是想要安裝管理工具，請取消選取 **\[Hyper-V 平台\]** (在 **\[程式和功能\]** > **\[開啟或關閉 Windows 功能\]** 中)。
