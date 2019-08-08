---
title: 使用 PowerShell Direct 管理 Windows 虛擬機器
description: 使用 PowerShell Direct 管理 Windows 虛擬機器
keywords: windows 10, hyper-v, powershell, 整合服務, 整合元件, 自動化, powershell direct
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
ms.openlocfilehash: ed96c7ba30c83906cd3245a279ab078229400d8d
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998725"
---
# <a name="virtual-machine-automation-and-management-using-powershell"></a>使用 PowerShell 進行虛擬機器自動化與管理

您可以使用 PowerShell Direct，透過 Hyper-V 主機，在 Windows 10 或 Windows Server 2016 虛擬機器中執行任意 PowerShell (不論網路設定或遠端管理設定為何皆可)。

以下是一些您可以直接執行 PowerShell Direct 的方法:

* [使用 Enter-PSSession Cmdlet 做為互動式會話](#create-and-exit-an-interactive-powershell-session)
* [使用 Invoke 命令 Cmdlet 作為單一使用區段來執行單一命令或腳本](#run-a-script-or-command-with-invoke-command)
* [使用新的 PSSession、複本專案和移除 PSSession Cmdlet 作為 persistant 會話 (組建14280及更新版本)](#copy-files-with-new-pssession-and-copy-item)

## <a name="requirements"></a>需求
**作業系統需求：**
* 主機：執行 Hyper-V 的 Windows 10、Windows Server 2016 或更新版本。
* 客體/虛擬機器：Windows 10、Windows Server 2016 或更新版本。

如果您管理的是較舊的虛擬機器，請使用虛擬機器連線 (VMConnect)，或是[設定虛擬機器的虛擬網路](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc816585(v=ws.10))。 

**設定需求：**    
* 虛擬機器必須在本機主機上執行。
* 虛擬機器必須開啟，且使用至少一個已設定的使用者設定檔執行。
* 您必須以 Hyper-V 系統管理員身分登入主機電腦。
* 您必須提供虛擬機器的有效使用者認證。

-------------

## <a name="create-and-exit-an-interactive-powershell-session"></a>建立並結束互動式 PowerShell 工作階段

在虛擬機器上執行 PowerShell 命令的最簡單方式，是啟動互動式工作階段。

當工作階段啟動時，您輸入的命令會在虛擬機器上執行，就如同您直接在虛擬機器本身的 PowerShell 工作階段輸入一樣。

**啟動互動式工作階段：**

1. 在 Hyper-V 主機上，以系統管理員身分開啟 PowerShell。

2. 使用虛擬機器名稱或 GUID，執行下列命令之一來建立互動式工作階段：  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  在提示時，提供虛擬機器帳戶的認證。

3. 在虛擬機器上執行命令。
  
  您應該會看到 VMName 做為 PowerShell 提示字元的首碼，如下所示︰
  
  ``` 
  [VMName]: PS C:\>
  ```
  
  執行的任何命令，都將在您的虛擬機器上執行。 若要測試，您可以執行 `ipconfig` 或 `hostname`，確定這些命令在虛擬機器中執行。
  
4. 當您完成時，執行下列命令以關閉工作階段：  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> 注意︰如果您的工作階段不會連線，請參閱[疑難排解](#troubleshooting)以了解可能的原因。 

若要深入了解這些 Cmdlet，請參閱 [Enter-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Enter-PSSession?view=powershell-5.1) 和 [Exit-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Exit-PSSession?view=powershell-5.1)。 

-------------

## <a name="run-a-script-or-command-with-invoke-command"></a>執行指令碼或 Invoke-Command 命令

PowerShell Direct 與 Invoke-Command 最適合您需要在虛擬機器上執行一個命令或一個指令碼，但超過該時間點便不需要繼續與虛擬機器互動的情況。

**執行單一命令：**

1. 在 Hyper-V 主機上，以系統管理員身分開啟 PowerShell。

2. 使用虛擬機器名稱或 GUID，執行下列其中一個命令來建立工作階段：  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { command } 
   Invoke-Command -VMId <VMId> -ScriptBlock { command }
   ```
   
   在提示時，提供虛擬機器帳戶的認證。
   
   命令會在虛擬機器上執行，如果有送往主控台的輸出，它會列印到您的主控台。  命令執行之後便會自動關閉連線。
   
   
**執行指令碼：**

1. 在 Hyper-V 主機上，以系統管理員身分開啟 PowerShell。

2. 使用虛擬機器名稱或 GUID，執行下列其中一個命令來建立工作階段：  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   在提示時，提供虛擬機器帳戶的認證。
   
   指令碼會在虛擬機器上執行。  命令執行之後便會自動關閉連線。

若要深入了解這個 Cmdlet，請參閱 [Invoke-Command](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Invoke-Command?view=powershell-5.1)。 

-------------

## <a name="copy-files-with-new-pssession-and-copy-item"></a>使用 New-PSSession 和 Copy-Item 複製檔案

> **注意︰** PowerShell Direct 只在 Windows 組建 14280 和更新版本中支援持續性工作階段

撰寫跨一或多部遠端電腦來協調動作的指令碼時，持續性的 PowerShell 工作階段會非常有用。  一旦建立後，持續性工作階段便會存在於背景中，直到您決定將它們刪除為止。  這表示您可以不斷地使用 `Invoke-Command` 或 `Enter-PSSession` 來參考相同的工作階段，而不必傳遞認證。

工作階段藉由相同的權杖來保存狀態。  因為持續性工作階段會持續存在，工作階段中建立的任何變數，或是傳遞至工作階段的任何變數，都將會在多個呼叫之間保留。 有數種工具可用來處理持續性工作階段。  在本例中，我們將使用 [New-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/New-PSSession?view=powershell-5.1) 和 [Copy-Item](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Management/Copy-Item?view=powershell-5.1) 來將資料從主機移到虛擬機器，以及從虛擬機器移到主機。

**建立工作階段然後複製檔案︰**  

1. 在 Hyper-V 主機上，以系統管理員身分開啟 PowerShell。

2. 執行下列其中一個命令，使用 `New-PSSession` 建立對虛擬機器的持續性 PowerShell 工作階段。
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  在提示時，提供虛擬機器帳戶的認證。
  
  > **警告：**  
   在 14500 之前的組建中有一項錯誤。  如果未使用 `-Credential` 旗標明確指定認證，客體中的服務會當機，並且將需要重新啟動。  如果您遇到這個問題，[這裡](#error-a-remote-session-might-have-ended)有提供因應措施的指示。
  
3. 將檔案複製到虛擬機器。
  
  若要將 `C:\host_path\data.txt` 從主機電腦複製到虛擬機器，請執行︰
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  從虛擬機器複製檔案 (然後複製到主機)。 
   
   若要將 `C:\guest_path\data.txt` 從虛擬機器複製到主機，請執行︰
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. 使用 `Remove-PSSession` 停止持續性工作階段。
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## <a name="troubleshooting"></a>疑難排解

PowerShell Direct 有一小組常見的錯誤訊息。  以下是最常見的訊息、部分原因，以及用來診斷問題的工具。

### <a name="-vmname-or--vmid-parameters-dont-exist"></a>-VMName 或 -VMID 參數不存在
**問題：**  
`Enter-PSSession`、`Invoke-Command` 或 `New-PSSession` 沒有 `-VMName` 或 `-VMId` 參數。

**可能的原因：**  
最可能的問題是您的主機作業系統不支援 PowerShell Direct。

您可以執行下列命令來檢查 Windows 組建︰

``` PowerShell
[System.Environment]::OSVersion.Version
```

如果您正在執行支援的組建，也很可能您的 PowerShell 版本不會執行 PowerShell Direct。  針對 PowerShell Direct 和 JEA，主要版本必須是 5 或更新版本。

您可以執行下列命令來檢查 PowerShell 版本組建︰

``` PowerShell
$PSVersionTable.PSVersion
```


### <a name="error-a-remote-session-might-have-ended"></a>錯誤：遠端工作階段可能已經結束
> **注意：**  
針對主機組建 10240 和 12400 之間的 Enter-PSSession，以下所有錯誤都報告為「遠端工作階段可能已經結束」。

**錯誤訊息：**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**可能的原因：**
* 虛擬機器存在，但未執行。
* 客體 OS 不支援 PowerShell Direct (請參閱[需求](#requirements))
* 客體中還沒有 PowerShell
  * 作業系統尚未完成開機
  * 作業系統無法正確開機
  * 某些開機期間事件需要使用者輸入

您可以使用 [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) Cmdlet，查看在主機上執行的 VM。

**錯誤訊息：**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**可能的原因：**
* 上面所列的其中一個原因 - 它們都同樣適用於 `New-PSSession`  
* 目前組建中的錯誤，其中必須使用 `-Credential` 來明確傳遞認證。  發生這種情況時，客體作業系統中的整個服務會停止回應，並需要重新啟動。  您可以使用 Enter-PSSession，檢查工作階段是否仍然可用。

若要解決認證問題，請使用 VMConnect 登入虛擬機器、開啟 PowerShell，然後使用下列 PowerShell 來重新啟動 vmicvmsession 服務︰

``` PowerShell
Restart-Service -Name vmicvmsession
```

### <a name="error-parameter-set-cannot-be-resolved"></a>錯誤：無法解析參數集
**錯誤訊息：**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**可能的原因：**  
* `-RunAsAdministrator` 連接到虛擬機器時不受支援。
     
  連接到 Windows 容器時，`-RunAsAdministrator` 旗標允許系統管理員不需明確的認證即可連線。  由於虛擬機器不會提供主機隱含的系統管理員存取權，所以您必須明確地輸入認證。

利用 `-Credential` 參數或在出現提示時手動輸入認證，即可將系統管理員認證傳遞到虛擬機器。


### <a name="error-the-credential-is-invalid"></a>錯誤︰認證無效。

**錯誤訊息：**  
```
Enter-PSSession : The credential is invalid.
```

**可能的原因：** 
* 無法驗證客體認證
  * 提供的認證不正確。
  * 在客體 (作業系統未啟動之前) 中沒有使用者帳戶
  * 如果以系統管理員身分進行連接：系統管理員尚未設定為使用中的使用者。  請按一下[這裡](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh825104(v=win.10)>)進一步了解。
  
### <a name="error-the-input-vmname-parameter-does-not-resolve-to-any-virtual-machine"></a>錯誤︰輸入的 VMName 參數未解析為任何虛擬機器。

**錯誤訊息：**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**可能的原因：**  
* 您不是 Hyper-V 系統管理員。  
* 虛擬機器不存在。

您可以使用 [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) Cmdlet 來確認您使用的認證是否具有 Hyper-V 系統管理員角色，並查看哪些 VM 在本機主機上執行及啟動。


-------------

## <a name="samples-and-user-guides"></a>範例和使用者指南

PowerShell Direct 支援 JEA (Just Enough Administration)。  請參閱本使用者指南來試試。

檢視 [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) 上的範例。
