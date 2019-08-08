---
title: Hyper-V 整合服務
description: Hyper-V 整合服務的參考
keywords: windows 10, hyper-v, 整合服務, 整合元件
author: scooley
ms.date: 05/25/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 18930864-476a-40db-aa21-b03dfb4fda98
ms.openlocfilehash: 6568b68a77fc5506b58249caea44ec78e3e44de2
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998935"
---
# <a name="hyper-v-integration-services"></a>Hyper-V 整合服務

整合服務通常稱為「整合元件」，是可讓虛擬機器與 Hyper-V 主機通訊的服務。 這當中有許多便利的服務，亦有些服務是虛擬機器功能是否能正常運作的關鍵。

本文提供 Windows 適用之每一項整合服務的參考資料。  如需特定整合服務或其歷程記錄的任何相關資訊，也可以將這篇文章做為起點。

**使用者指南：**  
* [管理整合服務](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Manage-Hyper-V-integration-services)


## <a name="quick-reference"></a>快速參考

| Name | Windows 服務名稱 | Linux 精靈名稱 |  說明 | 停用時對 VM 的影響 |
|:---------|:---------|:---------|:---------|:---------|
| [Hyper-V 活動訊號服務](#hyper-v-heartbeat-service) |  vmicheartbeat | hv_utils | 回報虛擬機器目前正確執行。 | 不定 |
| [Hyper-V 客體關機服務](#hyper-v-guest-shutdown-service) | vmicshutdown | hv_utils |  可讓主機觸發虛擬機器關機。 | **高** |
| [Hyper-V 時間同步化服務](#hyper-v-time-synchronization-service) | vmictimesync | hv_utils | 同步虛擬機器的時鐘與主機電腦的時鐘。 | **高** |
| [Hyper-V 資料交換服務 (KVP)](#hyper-v-data-exchange-service-kvp) | vmickvpexchange | hv_kvp_daemon | 提供在虛擬機器和主機之間交換基本中繼資料的方式。 | 中型 |
| [Hyper-V 磁碟區陰影複製要求者](#hyper-v-volume-shadow-copy-requestor) | vmicvss | hv_vss_daemon | 可讓磁碟區陰影複製服務不需關閉虛擬機器，就對其進行備份。 | 不定 |
| [Hyper-V 客體服務介面](#hyper-v-powershell-direct-service) | vmicguestinterface | hv_fcopy_daemon | 提供介面以供 Hyper-V 主機在虛擬機器之間複製檔案。 | 低 |
| [Hyper-V PowerShell Direct 服務](#hyper-v-powershell-direct-service) | vmicvmsession | 無法使用 | 提供一種方法，以使用 PowerShell 管理虛擬機器而不需網路連線。 | 低 |  


## <a name="hyper-v-heartbeat-service"></a>Hyper-V 活動訊號服務

**Windows 服務名稱：** vmicheartbeat  
**Linux 精靈名稱：** hv_utils  
**描述：** 可讓 Hyper-V 主機知道虛擬機器已安裝作業系統並正確開機。  
**已新增至：** Windows Server 2012、Windows 8  
**影響︰** 停用時，虛擬機器即無法回報其內部的作業系統是否正常運作。  這可能會影響某些類型的監視與主機端診斷功能。  

活動訊號服務可回答「虛擬機器是否已開機？」這類基本問題。  

當 Hyper-V 回報虛擬機器狀態為「正在執行中」時 (請參閱下面的範例)，表示 Hyper-V 已針對虛擬機器設定資源；而非表示作業系統已安裝或正常運作。  這時候，活動訊號就非常實用。  活動訊號服務可讓 Hyper-V 知道虛擬機器內的作業系統是否已開機。  

### <a name="check-heartbeat-with-powershell"></a>使用 PowerShell 檢查活動訊號

以系統管理員身分執行 [GET-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps)，以查看虛擬機器的活動訊號︰
``` PowerShell
Get-VM -VMName $VMName | select Name, State, Status
```

您的輸出看起來應該像這樣︰
```
Name    State    Status
----    -----    ------
DemoVM  Running  Operating normally
```

`Status` 欄位是由活動訊號服務決定。



## <a name="hyper-v-guest-shutdown-service"></a>Hyper-V 客體關機服務

**Windows 服務名稱：** vmicshutdown  
**Linux 精靈名稱：** hv_utils  
**描述︰** 可讓 Hyper-V 主機要求虛擬機器關機。  主機一律可以強制關閉虛擬機器，但這就像直接按電源開關來關機一樣。  
**已新增至：** Windows Server 2012、Windows 8  
**影響︰****強烈影響：** 停用時，主機即無法觸發虛擬機器內的溫和關機程序。  所有關閉都將會關閉, 這可能會導致資料遺失或資料損毀。  


## <a name="hyper-v-time-synchronization-service"></a>Hyper-V 時間同步化服務

**Windows 服務名稱：** vmictimesync  
**Linux 精靈名稱：** hv_utils  
**描述︰** 同步實體電腦的系統時鐘與虛擬機器的系統時鐘。  
**已新增至：** Windows Server 2012、Windows 8  
**影響︰****強烈影響：** 停用時，虛擬機器的時鐘會不規律地漂移。  


## <a name="hyper-v-data-exchange-service-kvp"></a>Hyper-V 資料交換服務 (KVP)

**Windows 服務名稱：** vmickvpexchange  
**Linux 精靈名稱：** hv_kvp_daemon  
**描述：** 提供在虛擬機器和主機之間交換基本中繼資料的機制。  
**已新增至：** Windows Server 2012、Windows 8  
**影響︰** 停用時，執行 Windows 8 或 Windows Server 2012 (或更早版本) 的虛擬機器將不會收到 Hyper-V 整合服務的更新。  停用資料交換時，可能也會影響某些類型的監視與主機端診斷功能。  

資料交換服務有時稱為 KVP，其會透過 Windows 登錄來使用機碼值組 (KVP)，以共用虛擬機器與 Hyper-V 主機之間的少量電腦資訊。  您也可以使用相同的機制，共用虛擬機器和主機之間的自訂資料。

機碼值組是由「機碼」和「值」構成。 機碼和值都是字串，且不支援任何其他資料類型。 當您建立或變更機碼值組時，客體和主機中都會有所顯示。 機碼值組的資訊會傳輸給整個 Hyper-V VMbus，且客體和 Hyper-V 主機之間不需要任何一種網路連線。 

資料交換服務是保留虛擬機器資訊的實用工具 – 若是互動式資料共用或資料轉送，請使用 [PowerShell Direct](#hyper-v-powershell-direct-service)。 


**使用者指南：**  
* [Using key-value pairs to share information between the host and guest on Hyper-V](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn798287(v=ws.11)) (使用機碼值組在 Hyper-V 的主機與客體之間共用資訊)。  


## <a name="hyper-v-volume-shadow-copy-requestor"></a>Hyper-V 磁碟區陰影複製要求者

**Windows 服務名稱：** vmicvss  
**Linux 精靈名稱：** hv_vss_daemon  
**描述︰** 可讓磁碟區陰影複製服務備份虛擬機器上的應用程式和資料。  
**已新增至：** Windows Server 2012、Windows 8  
**影響︰** 停用時，即無法在執行虛擬機器時加以備份 (使用 VSS)。  

磁碟區陰影複製服務 ([VSS](https://docs.microsoft.com/windows/desktop/VSS/overview-of-processing-a-backup-under-vss)) 需搭配「磁碟區陰影複製要求者」整合服務才能發揮效益。  磁碟區陰影複製服務 (VSS) 可擷取並複製映像，以備份執行中的系統 (尤其是伺服器)，而不會過度降低所提供之服務的效能和穩定性。  這是由於此整合服務可透過協調虛擬機器和主機備份程序的工作負載，來實現上述目標。

如需磁碟區陰影複製的詳細資訊，請參閱[這裡](https://docs.microsoft.com/previous-versions/windows/desktop/virtual/backing-up-and-restoring-virtual-machines)。


## <a name="hyper-v-guest-service-interface"></a>Hyper-V 客體服務介面

**Windows 服務名稱：** vmicguestinterface  
**Linux 精靈名稱：** hv_fcopy_daemon  
**描述：** 提供介面以供 Hyper-V 主機在虛擬機器之間雙向複製檔案。  
**已新增至：** Windows Server 2012 R2、Windows 8.1  
**影響︰** 停用時，主機即無法使用 `Copy-VMFile` 在客體之間複製檔案。  如需詳細資訊，請參閱 [Copy-VMFile Cmdlet](https://docs.microsoft.com/powershell/module/hyper-v/copy-vmfile?view=win10-ps)。  

**注意：**  
預設為停用。  請參閱[使用 Copy-Item 的 PowerShell Direct](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)。 


## <a name="hyper-v-powershell-direct-service"></a>Hyper-V PowerShell Direct 服務

**Windows 服務名稱：** vmicvmsession  
**Linux 精靈名稱：** 無  
**描述︰** 提供一個機制，以透過 VM 工作階段使用 PowerShell 來管理虛擬機器，而不需虛擬網路。    
**已新增至：** Windows Server TP3、Windows 10  
**影響︰** 停用此服務會導致主機無法使用 PowerShell Direct 連接到虛擬機器。  

**注意：**  
此服務名稱原本為「Hyper-V VM 工作階段服務」。  
PowerShell Direct 目前仍在開發中，因此只能在 Windows 10 和 Windows Server Technical Preview 3 或更新版本的主機/客體中使用。

不論 Hyper-V 主機或虛擬機器上的網路設定或遠端管理設定為何，PowerShell Direct 皆可讓您透過 Hyper-V 主機，進行虛擬機器內部的 PowerShell 管理。 這讓 Hyper-V 系統管理員更容易用指令碼自動化管理和設定工作。

[深入了解 PowerShell Direct](../user-guide/powershell-direct.md)。  

**使用者指南：**  
* [在虛擬機器中執行指令碼](../user-guide/powershell-direct.md#run-a-script-or-command-with-invoke-command)
* [在虛擬機器之間複製檔案](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)
