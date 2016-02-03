# 使用 PowerShell Direct 管理 Windows

您可以使用 PowerShell Direct，從 Windows 10 或 Windows Server 技術預覽 Hyper-V 主機遠端管理 Windows 10 或 Windows Server 技術預覽虛擬機器。 使用 PowerShell Direct 可在虛擬機器內用 PowerShell 管理，不論在 Hyper-V 主機上或虛擬機器上的網路設定或遠端管理設定為何。 這讓 Hyper-V 系統管理員更容易用指令碼自動化虛擬機器的管理和設定。

有兩種方式可以執行 PowerShell Direct：
* 做為互動式工作階段 - [移至本節](vmsession.md#create-and-exit-an-interactive-powershell-session)，使用 PSSession Cmdlet 建立及結束 PowerShell Direct 工作階段
* 執行一組命令或指令碼 -- [移至本節](vmsession.md#run-a-script-or-command-with-invoke-command)，使用 Invoke-Command Cmdlet 執行指令碼或命令


## 需求

**作業系統需求：**
* 主機作業系統必須執行 Windows 10、Windows Server 技術預覽或更高版本。
* 虛擬機器必須執行 Windows 10、Windows Server 技術預覽或更高版本。

如果您管理的是較舊的虛擬機器，請使用虛擬機器連線 (VMConnect)，或是[設定虛擬機器的虛擬網路](http://technet.microsoft.com/library/cc816585.aspx)。

若要在虛擬機器上建立 PowerShell Direct 工作階段，
* 虛擬機器必須在本機主機上執行和啟動。
* 您必須以 Hyper-V 系統管理員身分登入主機電腦。
* 您必須提供虛擬機器的有效使用者認證。

## 建立並結束互動式 PowerShell 工作階段

1. 在 Hyper-V 主機上，以系統管理員身分開啟 Windows PowerShell。

3. 使用虛擬機器名稱或 GUID，執行下列命令之一來建立工作階段：
``` PowerShell
Enter-PSSession -VMName <VMName>
Enter-PSSession -VMGUID <VMGUID>
```

4. 執行您需要執行的命令。 這些命令會在您用來建立工作階段的虛擬機器上執行。
5. 當您完成時，執行下列命令以關閉工作階段：
``` PowerShell
Exit-PSSession 
```

> 附註：如果您的工作階段不會連線，請確定您使用的是您要連線之虛擬機器的認證，不是 Hyper-V 主機的認證。

若要深入了解這些 Cmdlet，請參閱 [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) 和 [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx)。

## 執行指令碼或 Invoke-Command 命令

您可以使用 [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx) Cmdlet，在虛擬機器上執行一組預先決定的命令。 以下是如何使用 Invoke-Command Cmdlet 的範例，其中 PSTest 是虛擬機器名稱，要執行的指令碼 (foo.ps1) 位在 C:/ 磁碟機的指令碼資料夾：

 ``` PowerShell
 Invoke-Command -VMName PSTest -FilePath C:\script\foo.ps1 
 ```

若要執行單一命令，請使用 **-ScriptBlock** 參數：

 ``` PowerShell
 Invoke-Command -VMName PSTest -ScriptBlock { cmdlet } 
 ```

## 疑難排解

PowerShell Direct 有一小組常見錯誤訊息。 以下是最常見的訊息、部分原因，以及用來診斷問題的工具。

### 錯誤：遠端工作階段可能已經結束

**錯誤訊息：**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**可能的原因：**
* VM 未執行
* 客體作業系統不支援 PowerShell Direct (請參閱[需求](#Requirements)))
* 客體中還沒有 PowerShell
  * 作業系統尚未完成開機
  * 作業系統無法正確開機
  * 某些開機期間事件需要使用者輸入
* 無法驗證客體認證
  * 提供的認證不正確
  * 在客體 (作業系統未啟動之前) 中沒有使用者帳戶
  * 如果以系統管理員身分進行連接：系統管理員尚未設定為使用中的使用者。 按一下[這裡](https://technet.microsoft.com/en-us/library/hh825104.aspx)以深入了解。

您可以使用 [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) Cmdlet 來確認您使用的認證是否具有 Hyper-V 系統管理員角色，並查看哪些 VM 在本機主機上執行及啟動。

### 錯誤：無法解析參數集

**錯誤訊息：**
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**可能的原因：**  
連接到虛擬機器時不支援 `-RunAsAdministrator`。

PowerShell Direct 連接到虛擬機器與 Windows 容器時，會有不同的行為。 連接到 Windows 容器時，`-RunAsAdministrator` 旗標允許系統管理員不需明確的認證即可連線。 由於虛擬機器不會提供主機隱含的系統管理員存取權，所以您必須明確地輸入認證。

利用 `-credential` 參數或在出現提示時手動輸入認證，即可將系統管理員認證傳遞到虛擬機器。


## 範例

檢視 [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) 上的範例。

請參閱 [PowerShell Direct 程式碼片段](../develop/powershell_snippets.md)，其中有許多有關如何在您的環境中使用 PowerShell Direct 的範例，以及使用 PowerShell 編寫 Hyper-V 指令碼的秘訣與竅門。




<!--HONumber=Jan16_HO2-->
