# Windows 10 上的 Hyper-V 新功能

本主題說明 Windows 10® 上的 Hyper-V 的新功能和已變更的功能。

## Windows PowerShell Direct

現在，有個簡單可靠的方法可以從主機作業系統執行虛擬機器的 Windows PowerShell 命令。 沒有任何網路或防火牆的需求或特殊設定。 
不論您的遠端管理如何設定都能運作。 若要使用它，您必須在主機和虛擬機器客體作業系統上執行 Windows 10 或 Windows Server Technical Preview。

若要建立 PowerShell Direct 工作階段，使用下列命令之一：

``` PowerShell
Enter-PSSession -VMName VMName
Invoke-Command -VMName VMName -ScriptBlock { commands }
```

今天，Hyper-V 系統管理員依賴兩種工具來連線至其 Hyper-V 主機上的虛擬機器：
- 遠端管理工具，例如 PowerShell 或遠端桌面
- Hyper-V 虛擬機器連線 (VM Connect)

這兩種技術都運作良好，但隨著您的 Hyper-V 部署成長各有利弊。 VMConnect 很可靠，但可能難以自動化。 遠端 PowerShell 很強大，但可能難以設定及維護。

Windows PowerShell Direct 提供強大的指令碼處理和自動化體驗，又具備 VMConnect 的簡單。 由於 Windows PowerShell Direct 在主機與虛擬機器之間執行，不需要網路連線或啟用遠端管理。 您需要有客體的認證才能登入虛擬機器。

### 需求

- 您必須連線到 Windows 10 或 Windows Server Technical Preview 主機，且主機具有執行 Windows 10 或 Windows Server Technical Preview 客體的虛擬機器。
- 您必須使用主機上的 Hyper-V 系統管理員認證登入。
- 您需要虛擬機器的使用者認證。
- 您想要連線的虛擬機器必須已啟動和執行。


## 網路介面卡和記憶體的熱新增和移除

您現在可以在虛擬機器執行時新增或移除網路介面卡，而不需要停機。 這適用於執行 Windows 和 Linux 作業系統的第 2 代虛擬機器。

即使您沒有啟用動態記憶體，也可以在虛擬機器執行時調整指派給虛擬機器的記憶體量。 這適用於第 1 代和第 2 代虛擬機器。

## 生產檢查點

生產檢查點可讓您輕鬆地建立虛擬機器的「時間點」映像，可以在稍後以所有生產工作負載完全支援的方式還原。 這是利用在客體內使用備份技術建立檢查點，而不是使用儲存狀態技術來達成。 針對生產檢查點，Windows 虛擬機器內使用磁碟區快照服務 (VSS)。 Linux 虛擬機器會排清其檔案系統緩衝區，以建立一致的檔案系統檢查點。 如果您想要使用儲存狀態技術建立檢查點，仍然可以為您的虛擬機器選擇使用標準檢查點。


> **重要事項：**新虛擬機器的預設行為是建立生產檢查點並回到標準檢查點。


## Hyper-V 管理員的改進

- **備用認證支援** – 現在，當您連線到另一部 Windows 10 Technical Preview 遠端主機，可以在 Hyper-V 管理員中使用另一組不同的認證。 您也可以儲存這些認證，以便於之後登入。

- **舊版管理** - 您現在可以使用 Hyper-V 管理員來管理多個版本的 Hyper-V。 使用 Windows 10 Technical Preview 中的 Hyper-V 管理員，您可以管理 Windows Server 2012、Windows 8、Windows Server 2012 R2 和 Windows 8.1 上執行 Hyper-V 的電腦。

- **更新的管理通訊協定** - Hyper-V 管理員已更新為使用 WS-MAN 通訊協定與遠端 Hyper-V 主機通訊，此協定允許 CredSSP、Kerberos 或 NTLM 驗證。 當您使用 CredSSP 連線到遠端 Hyper-V 主機時，它可讓您執行即時移轉，而不需要先在 Active Directory 中啟用限制委派。 WS-MAN 基礎結構也可以簡化啟用主機遠端管理所需的設定。 WS-MAN 透過連接埠 80 連線，預設會開啟此連接埠。


## 連線待命狀態的運作

現在，如果在使用｢永遠開啟/永遠連線｣ (AOAC) 電源模式的電腦上啟用 Hyper-V，將可使用「連線待命」電源狀態。

在 Windows 8 和 8.1 中，Hyper-V 會造成使用｢永遠開啟永遠連線｣ (AOAC) 電源模式 (也稱為 InstantON) 的電腦永遠不會進入睡眠狀態。 請參閱此 [知識庫文章](
https://support.microsoft.com/en-us/kb/2973536) 以取得詳細說明。


## Linux 安全開機

現在，在第 2 代虛擬機器上執行更多個 Linux 作業系統，可以啟用安全開機選項進行開機。 為了在執行 Technical Preview 的主機上安全開機，現已啟用 Ubuntu 14.04 和更新版本以及 SUSE Linux Enterprise Server 12。 您第一次啟動虛擬機器之前，您必須指定虛擬機器應該使用 Microsoft UEFI 憑證授權單位。 在提升權限的 Windows PowerShell 提示中輸入：

    Set-VMFirmware vmname -SecureBootTemplate MicrosoftUEFICertificateAuthority

如需在 Hyper-V 上執行 Linux 虛擬機器的詳細資訊，請參閱 [Hyper-V 上的 Linux 和 FreeBSD 虛擬機器](http://technet.microsoft.com/library/dn531030.aspx)。


## 虛擬機器設定版本

當您將虛擬機器從執行 Windows 8.1 的主機移動或匯入至執行 Hyper-V 的 Windows 10 主機時，虛擬機器的設定檔案不會自動升級。 這讓虛擬機器可以移回執行 Windows 8.1 的主機。 您將不能存取新的虛擬機器功能，直到您手動更新虛擬機器設定版本。

虛擬機器設定版本代表虛擬機器設定的 Hyper-V 版本、儲存的狀態，以及與之相容的快照檔案。 設定版本 5 的虛擬機器與 Windows 8.1 相容，可以在 Windows 8.1 和 Windows 10 上執行。 設定版本 6 的虛擬機器與 Windows 10 相容，無法在 Windows 8.1 上執行。

### 檢查設定版本

在已提高權限的命令提示字元中執行下列命令：

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### 升級設定版本

從提升權限的 Windows PowerShell 令提示字元，執行下列命令之一：

``` PowerShell
Update-VmConfigurationVersion <vmname>
```

或

``` PowerShell
Update-VmConfigurationVersion <vmobject>
```

*重要事項：*
- 升級虛擬機器設定版本之後，無法將虛擬機器移至執行 Windows 8.1 的主機。
- 您無法將虛擬機器設定版本從版本 6 降級至版本 5。
- 您必須關閉虛擬機器，才能升級虛擬機器設定。
- 升級之後，虛擬機器會使用新的設定檔案格式。 如需詳細資訊，請參閱新的虛擬機器設定檔案格式。


## 設定檔案格式

現在，虛擬機器有新的設定檔案格式，這種格式是為了提高讀取和寫入虛擬機器設定資料的效率而設計。 它也設計為會降低儲存體失敗時的資料損毀可能性。 新的設定檔案在虛擬機器設定資料使用 .VMCX 副檔名，在執行階段狀態資料使用 .VMRS 副檔名。


> **重要事項：**.VMCX 檔案是一種二進位格式。 不支援直接編輯 .VMCX 或 .VMRS 檔案。

## 透過 Windows Update 的整合服務

Windows 客體整合服務的更新現在是透過 Windows Update 散發。

整合元件 (又稱為「整合服務」) 是一組綜合的驅動程式，可讓虛擬機器與主機作業系統通訊。 整合元件控制的服務範圍從時間同步處理到客體檔案複製都有。 過去一年我們一直和客戶談到整合元件安裝和更新，發現這些是升級程序期間的巨大痛苦點。


在過去，所有新版本的 Hyper-V 會隨附新的整合元件。 升級 Hyper-V 主機，也需要升級虛擬機器的整合元件。 新的整合元件都包含在 Hyper-V 主機，然後再用 vmguest.iso 將其安裝在虛擬機器中。 這個處理序需要重新啟動虛擬機器，而且無法與其他 Windows 更新同批次處理。 因為 Hyper-V 系統管理員必須提供 vmguest.iso，且虛擬機器系統管理員必須將其安裝，所以整合元件升級需要 Hyper-V 系統管理員具有虛擬機器的系統管理員認證的 -- 但這並非絕對。
　　


在 Windows 10 和接下來的系統中，會透過 Windows Update 將所有的整合元件 (和其他重要更新) 傳遞給虛擬機器。


今天，執行這些系統的虛擬機器有更新：
*  Windows Server 2012
*  Windows Server 2008 R2
*  Windows 8
*  Windows 7

虛擬機器必須連線到 Windows Update 或 WSUS 伺服器。 在未來，整合元件更新將會有類別識別碼，在此版本中，它們會列為 KB。

若要深入了解我們如何判斷適用性，請參閱此[部落格文章](http://blogs.technet.com/b/virtualization/archive/2014/11/24/integration-components-how-we-determine-windows-update-applicability.aspx)。


如需安裝整合服務的詳細逐步解說，請參閱[這個部落格](http://blogs.msdn.com/b/virtual_pc_guy/archive/2014/11/12/updating-integration-components-over-windows-update.aspx)文章。


> **重要事項：**更新整合元件不再需要 ISO 映像檔案 vmguest.iso。 它並不包含在 Windows 10 上的 Hyper-V 中。


## 後續步驟

[逐步解說 Windows 10 上的 Hyper-V](..\quick_start\walkthrough.md)



