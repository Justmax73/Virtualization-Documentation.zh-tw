---
author: neilpeterson
---

# 將 Windows 容器主機部署至新的 Hyper-V 虛擬機器

此文件將逐步說明如何使用 PowerShell 指令碼部署新的 Hyper-V 虛擬機器，然後設定為 Windows 容器主機。

若要逐步執行指令碼將 Windows 容器主機部署至現有的虛擬或實體系統，請參閱[就地 Windows 容器主機部署](./inplace_setup.md)。

**安裝容器 OS 映像之前請先閱讀：**需遵守 Microsoft Windows Server 發行前版本軟體的授權條款 (「授權條款」)，才能使用 Microsoft Windows 容器 OS 映像補充軟體 (「補充軟體」)。 下載並使用補充軟體後，即表示您同意「授權條款」，如果您不接受授權條款，則無法使用。 Windows 伺服器發行前軟體和補充軟體皆由 Microsoft Corporation 授權。

必須符合下列條件，才能完成此快速入門中的 **Windows Server** 和 **Hyper-V 容器**練習。

* 執行 Windows 10 組建 10586 或更新版本/Windows Server Technical Preview 4 或更新版本的系統。
* 啟用 Hyper-V 角色 ([請參閱指示](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install#UsingPowerShell)))。
* 有 20 GB 的可用儲存體可供容器主機映像、OS 基本映像和安裝指令碼使用。
* Hyper-V 主機的系統管理員權限。

> 執行 Hyper-V 容器的虛擬化容器主機需要巢狀虛擬化。 實體主機和虛擬主機皆必須執行支援巢狀虛擬化的 OS。 如需詳細資訊，請參閱 [Windows Server 2016 Technical Preview](https://technet.microsoft.com/library/dn765471.aspx#BKMK_nested) 上的「Hyper-V 新功能」。

## 在新的虛擬機器上安裝新的容器主機

Windows 容器包含數個元件，例如 Windows 容器主機和容器 OS 基本映像。 我們為您整合出一個可下載並設定這些項目的指令碼。 請遵循下列步驟來部署新的 Hyper-V 虛擬機器，並將此系統設定為 Windows 容器主機。

以系統管理員身分啟動 PowerShell 工作階段。 要執行此動作，請在 [PowerShell] 圖示上按一下滑鼠右鍵，然後選取 [以系統管理員身分執行]，或從任何 PowerShell 工作階段中執行下列命令。

``` powershell
PS C:\> start-process powershell -Verb runAs
```

在下載並執行指令碼之前，請確定已建立外部 Hyper-V 虛擬交換器。 若未建立，此指令碼將會失敗。

執行下列命令，會傳回外部虛擬交換器的清單。 若未傳回任何內容，請建立新的外部虛擬交換器，然後繼續執行本指南的下一個步驟。

```powershell
PS C:\> Get-VMSwitch | where {$_.SwitchType -eq “External”}
```

使用下列命令下載設定指令碼。 您也可以手動從這個位置下載指令碼 - [設定指令碼](https://aka.ms/tp4/New-ContainerHost)。

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/New-ContainerHost -OutFile c:\New-ContainerHost.ps1
```

執行下列命令以建立及設定容器主機，其中，`&lt;containerhost&gt;` 是虛擬機器名稱。

``` powershell
PS C:\> powershell.exe -NoProfile c:\New-ContainerHost.ps1 -VMName testcont -WindowsImage ServerDatacenterCore -Hyperv
```

指令碼開始時，系統會提示您輸入密碼。 這會是指派給系統管理員帳戶的密碼。

接下來，系統會要求您閱讀並接受授權條款。

```
Before installing and using the Windows Server Technical Preview 4 with Containers virtual machine you must:
    1. Review the license terms by navigating to this link: http://aka.ms/tp4/containerseula
    2. Print and retain a copy of the license terms for your records.
By downloading and using the Windows Server Technical Preview 4 with Containers virtual machine you agree to such
license terms. Please confirm you have accepted and agree to the license terms.
[N] No  [Y] Yes  [?] Help (default is "N"):
```

接著，指令碼會開始下載及設定 Windows 容器元件。 執行此程序可能需要一段時間，因為下載內容較大。 完成之後，您的虛擬機器即已設定並就緒，可讓您透過 PowerShell 和 Docker 來建立及管理 Windows 容器和 Windows 容器映像。

設定指令碼完成後，請使用設定程序執行期間所指定的密碼登入虛擬機器，並確定虛擬機器具有有效的 IP 位址。 這些項目皆完成後，您的系統即應可供 Windows 容器使用。

## 後續步驟：開始使用容器

現在您已有執行 Windows 容器功能的 Windows Server 2016 系統，接下來請移至下列指南，開始使用 Windows Server 容器和 Hyper-V 容器。

您可以在 HYPER-V 管理主機中使用 `Enter-PSSession` 命令，以連線至容器主機。

```powershell
PS C:\> Enter-PSSession -VMName <VM Name>
```

[快速入門：Windows 容器和 PowerShell](./manage_powershell.md)  
[快速入門：Windows 容器和 Docker](./manage_docker.md)






<!--HONumber=Mar16_HO3-->


