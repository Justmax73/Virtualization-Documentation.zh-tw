---
title: "使用 PowerShell Direct 管理 Windows 虛擬機器"
description: "使用 PowerShell Direct 管理 Windows 虛擬機器"
keywords: "windows 10, hyper-v, powershell, 整合服務, 整合元件, 自動化, powershell direct"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
translationtype: Human Translation
ms.sourcegitcommit: 8f08c85921b9d41f10f3b8cff5e4bafe945bd4af
ms.openlocfilehash: 807043a744c4735158720371ec3afb22ebe7fc24

---

# 使用 PowerShell 進行虛擬機器自動化與管理
 
您可以使用 PowerShell Direct，透過 Hyper-V 主機，在 Windows 10 或 Windows Server Technical Preview 虛擬機器中執行任意 PowerShell (不論網路設定或遠端管理設定為何皆可)。

**執行 PowerShell Direct 的方式：**  
* 作為互動式工作階段 - [按一下這裡](vmsession.md#create-and-exit-an-interactive-powershell-session)使用 Enter-PSSession 建立並結束互動式 PowerShell 工作階段。
* 作為單次使用工作階段來執行單一命令或指令碼 - [按一下這裡](vmsession.md#run-a-script-or-command-with-invoke-command)使用 Invoke-Command 執行指令碼或命令。
* 作為持續工作階段 (組建 14280 和更新版本) - [按一下這裡](vmsession.md#copy-files-with-new-pssession-and-copy-item)使用 New-PSSSession 來建立持續工作階段。  
繼續使用 Copy-Item 來複製檔案至虛擬機器或從中複製檔案，然後使用 Remove-PSSession 中斷連線。

## 需求
**作業系統需求：**
* 主機：執行 Hyper-V 的 Windows 10、Windows Server Technical Preview 2 或更新版本。
* 客體/虛擬機器：Windows 10、Windows Server Technical Preview 2 或更新版本。

如果您管理的是較舊的虛擬機器，請使用虛擬機器連線 (VMConnect)，或是[設定虛擬機器的虛擬網路](http://technet.microsoft.com/library/cc816585.aspx)。 

**設定需求：**    
* 虛擬機器必須在本機主機上執行。
* 虛擬機器必須開啟，且使用至少一個已設定的使用者設定檔執行。
* 您必須以 Hyper-V 系統管理員身分登入主機電腦。
* 您必須提供虛擬機器的有效使用者認證。

-------------

## 建立並結束互動式 PowerShell 工作階段

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
  
  您應該會看到 VMName 作為 PowerShell 提示字元的首碼，如下所示︰
  
  ``` 
  [VMName]: PS C:\ >
  ```
  
  執行的任何命令，都將在您的虛擬機器上執行。  若要測試，您可以執行 `ipconfig` 或 `hostname`，確定這些命令在虛擬機器中執行。
  
4. 當您完成時，執行下列命令以關閉工作階段：  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> 注意︰如果您的工作階段不會連線，請參閱[疑難排解](vmsession.md#troubleshooting)以了解可能的原因。 

若要深入了解這些 Cmdlet，請參閱 [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) 和 [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx)。 

-------------

## 執行指令碼或 Invoke-Command 命令

PowerShell Direct 與 Invoke-Command 最適合您需要在虛擬機器上執行一個命令或一個指令碼，但超過該時間點便不需要繼續與虛擬機器互動的情況。

**執行單一命令：**

1. 在 Hyper-V 主機上，以系統管理員身分開啟 PowerShell。

2. 使用虛擬機器名稱或 GUID，執行下列其中一個命令來建立工作階段：  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { cmdlet } 
   Invoke-Command -VMId <VMId> -ScriptBlock { cmdlet }
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

若要深入了解這個 Cmdlet，請參閱 [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx)。 

-------------

## 使用 New-PSSession 和 Copy-Item 複製檔案

> **注意︰**PowerShell Direct 只在 Windows 組建 14280 和更新版本中支援持續性工作階段

撰寫跨一或多部遠端電腦來協調動作的指令碼時，持續性的 PowerShell 工作階段會非常有用。  一旦建立後，持續性工作階段便會存在於背景中，直到您決定將它們刪除為止。  這表示您可以不斷地使用 `Invoke-Command` 或 `Enter-PSSession` 來參考相同的工作階段，而不必傳遞認證。

工作階段藉由相同的權杖來保存狀態。  因為持續性工作階段會持續存在，工作階段中建立的任何變數，或是傳遞至工作階段的任何變數，都將會在多個呼叫之間保留。 有數種工具可用來處理持續性工作階段。  在本例中，我們將使用 [New-PSSession](https://technet.microsoft.com/en-us/library/hh849717.aspx) 和 [Copy-Item](https://technet.microsoft.com/en-us/library/hh849793.aspx) 來將資料從主機移到虛擬機器，以及從虛擬機器移到主機。

**建立工作階段然後複製檔案︰**  

1. 在 Hyper-V 主機上，以系統管理員身分開啟 PowerShell。

2. 執行下列其中一個命令，使用 `New-PSSession` 建立對虛擬機器的持續性 PowerShell 工作階段。
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  在提示時，提供虛擬機器帳戶的認證。
  
  > **警告：**  
   在 14500 之前的組建中有一項錯誤。  如果未使用 `-Credential` 旗標明確指定認證，客體中的服務會當機，並且將需要重新啟動。  如果您遇到這個問題，解決的指示提供在[這裡](vmsession.md#error-a-remote-session-might-have-ended)。
  
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

## 疑難排解

PowerShell Direct 有一小組常見的錯誤訊息。  以下是最常見的訊息、部分原因，以及用來診斷問題的工具。

### -VMName 或 -VMID 參數不存在
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
$PSVersionTable.PSVersionTable
```


### 錯誤：遠端工作階段可能已經結束
> **注意：**  
針對主機組建 10240 和 12400 之間的 Enter-PSSession，以下所有錯誤都報告為「遠端工作階段可能已經結束」。

**錯誤訊息：**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**可能的原因：**
* 虛擬機器存在，但未執行。
* 客體 OS 不支援 PowerShell Direct (請參閱[需求](#Requirements))
* 客體中還沒有 PowerShell
  * 作業系統尚未完成開機
  * 作業系統無法正確開機
  * 某些開機期間事件需要使用者輸入

您可以使用 [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) Cmdlet，查看在主機上執行的 VM。

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

### 錯誤：無法解析參數集
**錯誤訊息：**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**可能的原因：**  
* `-RunAsAdministrator` 連接到虛擬機器時不受支援。
     
  連接到 Windows 容器時，`-RunAsAdministrator` 旗標允許系統管理員不需明確的認證即可連線。  由於虛擬機器不會提供主機隱含的系統管理員存取權，所以您必須明確地輸入認證。

利用 `-Credential` 參數或在出現提示時手動輸入認證，即可將系統管理員認證傳遞到虛擬機器。


### 錯誤︰認證無效。

**錯誤訊息：**  
```
Enter-PSSession : The credential is invalid.
```

**可能的原因：** 
* 無法驗證客體認證
  * 提供的認證不正確。
  * 在客體 (作業系統未啟動之前) 中沒有使用者帳戶
  * 如果以系統管理員身分進行連接：系統管理員尚未設定為使用中的使用者。  請按一下[這裡](https://technet.microsoft.com/en-us/library/hh825104.aspx)進一步了解。
  
### 錯誤︰輸入的 VMName 參數未解析為任何虛擬機器。

**錯誤訊息：**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**可能的原因：**  
* 您不是 Hyper-V 系統管理員。  
* 虛擬機器不存在。

您可以使用 [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) Cmdlet 來確認您使用的認證是否具有 Hyper-V 系統管理員角色，並查看哪些 VM 在本機主機上執行及啟動。


-------------

## 範例和使用者指南

PowerShell Direct 支援 JEA (Just Enough Administration)。  請參閱本使用者指南來試試。

檢視 [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) 上的範例。

請參閱 [PowerShell Direct 程式碼片段](../develop/powershell_snippets.md)，其中有許多有關如何在您的環境中使用 PowerShell Direct 的範例，以及使用 PowerShell 編寫 Hyper-V 指令碼的秘訣與訣竅。



<!--HONumber=Nov16_HO1-->


