---
title: 嘗試 Hyper-V 的發行前版本功能
description: 嘗試 Hyper-V 的發行前版本功能
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 426c87cc-fa50-4b8d-934e-0b653d7dea7d
ms.openlocfilehash: 725466f657ae8fc4f14813822e90657e12d26fa6
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439555"
---
# <a name="try-pre-release-features-for-hyper-v"></a>嘗試 Hyper-V 的發行前版本功能

> 這是初版內容，後續可能會變更。  
  由於 Microsoft 不支援發行前版本的虛擬機器，因此這類虛擬機器主要用於開發或測試環境。

搶先存取 Windows Server 2016 Technical Preview 上的 Hyper-V 發行前版本功能，並在開發或測試環境中試用。 您可以率先看到最新的 Hyper-V 功能，並藉由提供回饋意見來協助塑造產品。

但是，您所建立的發行前版本虛擬機器不會有任何組建對組建的相容性或未來的支援。  因此，請不要在生產環境中使用它們。

下列說明這類虛擬機器僅適用於非生產環境的進一步原因：

* 發行前版本的虛擬機器沒有往後相容性。 這類虛擬機器無法升級至新的設定版本。
* 發行前版本的虛擬機器組建之間沒有一致的定義。 如果您更新主機作業系統，現有的發行前版本虛擬機器可能會與主機不相容。 這類虛擬機器可能無法啟動，或一開始看似可以運作，但稍後就發生重大的相容性問題。
* 如果您將發行前版本的虛擬機器匯入不同組建的主機時，會產生無法預測的結果。 您可以將發行前版本的虛擬機器移至另一部主機。 但這種情況下，應該只有在兩部主機都執行相同的組建時才能運作。

## <a name="create-a-pre-release-virtual-machine"></a>建立發行前版本的虛擬機器

您可以在執行 Windows Server 2016 Technical Preview 的 Hyper-V 主機上建立發行前版本的虛擬機器。

1. 在 Windows 桌面上，按一下 \[開始\] 按鈕，然後輸入 **Windows PowerShell** 名稱的任何一部分。
2. 以滑鼠右鍵按一下 [Windows PowerShell]，並選取 [以系統管理員身分執行]。
3. 使用 [New-VM](https://docs.microsoft.com/powershell/module/hyper-v/new-vm?view=win10-ps) Cmdlet 來建立發行前版本虛擬機器的 -Prerelease 旗標。 例如，執行下列命令，其中 VM Name 是您想要建立的虛擬機器名稱。

``` PowerShell
New-VM -Name <VM Name> -Prerelease
```
下列為可以使用 -Prerelease 旗標的其他範例：
 - 若要建立使用現有虛擬硬碟或新硬碟的虛擬機器，請參閱 [Create a virtual machine in Hyper-V on Windows Server 2016 Technical Preview](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/Create-a-virtual-machine-in-Hyper-V#BKMK_PowerShell) (在 Windows Server 2016 Technical Preview 的 Hyper-V 中建立虛擬機器) 的 PowerShell 範例。
 - 若要建立使用作業系統映像開機的新虛擬硬碟，請參閱[在 Windows 10 Hyper-V 中部署 Windows 虛擬機器](https://docs.microsoft.com/virtualization/hyper-v-on-windows/quick-start/create-virtual-machine)的 PowerShell 範例。

 這些文件包含的範例適用於執行 Windows 10 或 Windows Server 2016 Technical Preview 的 Hyper-V 主機。 但目前，您僅可使用 -Prerelease 旗標在執行 Windows Server 2016 Technical Preview 的 Hyper-V 主機上建立發行前版本的虛擬機器。

## <a name="see-also"></a>另請參閱
-  [Virtualization Blog](https://techcommunity.microsoft.com/t5/Virtualization/bg-p/Virtualization) (虛擬化部落格) - 深入了解可用的發行前版本功能，以及如何試用。
- [Supported virtual machine configuration versions](https://docs.microsoft.com/windows-server/virtualization/hyper-v/deploy/Upgrade-virtual-machine-version-in-Hyper-V-on-Windows-or-Windows-Server#BKMK_SupportedConfigVersions) (支援的虛擬機器設定版本) - 深入了解如何檢查虛擬機器設定版本，以及 Microsoft 支援哪些版本。
