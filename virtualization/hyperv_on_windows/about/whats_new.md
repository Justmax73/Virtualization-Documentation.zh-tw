---
redirect_url: https://msdn.microsoft.com/virtualization/hyperv_on_windows/windows_welcome
---

## 網路介面卡和記憶體的熱新增和移除

您現在可以在虛擬機器執行時新增或移除網路介面卡，而不需要停機。 這適用於執行 Windows 和 Linux 作業系統的第 2 代虛擬機器。

即使您沒有啟用動態記憶體，也可以在虛擬機器執行時調整指派給虛擬機器的記憶體量。 這適用於第 1 代和第 2 代虛擬機器。

## 生產檢查點

生產檢查點可讓您輕鬆地建立虛擬機器的「時間點」映像，可以在稍後以所有生產工作負載完全支援的方式還原。 這是利用在客體內使用備份技術建立檢查點，而不是使用儲存狀態技術來達成。 針對生產檢查點，Windows 虛擬機器內使用磁碟區快照服務 (VSS)。 Linux 虛擬機器會排清其檔案系統緩衝區，以建立一致的檔案系統檢查點。 如果您想要使用儲存狀態技術建立檢查點，仍然可以為您的虛擬機器選擇使用標準檢查點。


> <g id="1" ctype="x-strong">重要事項：</g>新虛擬機器的預設行為是建立生產檢查點並回到標準檢查點。


## Hyper-V 管理員的改進

- <g id="1" ctype="x-strong">備用認證支援</g> – 現在，當您連線到另一部 Windows 10 Technical Preview 遠端主機，可以在 Hyper-V 管理員中使用另一組不同的認證。 您也可以儲存這些認證，以便於之後登入。

- <g id="1" ctype="x-strong">舊版管理</g> – 您現在可以使用 Hyper-V 管理員來管理多個版本的 Hyper-V。 使用 Windows 10 Technical Preview 中的 Hyper-V 管理員，您可以管理 Windows Server 2012、Windows 8、Windows Server 2012 R2 和 Windows 8.1 上執行 Hyper-V 的電腦。

- <g id="1" ctype="x-strong">更新的管理通訊協定</g> – Hyper-V 管理員已更新為使用 WS-MAN 通訊協定與遠端 Hyper-V 主機通訊，此協定允許 CredSSP、Kerberos 或 NTLM 驗證。 當您使用 CredSSP 連線到遠端 Hyper-V 主機時，它可讓您執行即時移轉，而不需要先在 Active Directory 中啟用限制委派。 WS-MAN 基礎結構也可以簡化啟用主機遠端管理所需的設定。 WS-MAN 透過連接埠 80 連線，預設會開啟此連接埠。


## 連線待命相容性

現在，如果在使用｢永遠開啟/永遠連線｣ (AOAC) 電源模式的電腦上啟用 Hyper-V，將可使用「連線待命」電源狀態。

在 Windows 8 和 8.1 中，Hyper-V 會造成使用｢永遠開啟永遠連線｣ (AOAC) 電源模式 (也稱為 InstantON) 的電腦永遠不會進入睡眠狀態。 請參閱此 [知識庫文章](
https://support.microsoft.com/en-us/kb/2973536) 以取得詳細說明。


## Linux 安全開機

現在，在第 2 代虛擬機器上執行更多個 Linux 作業系統，可以啟用安全開機選項進行開機。 為了在執行 Technical Preview 的主機上安全開機，現已啟用 Ubuntu 14.04 和更新版本以及 SUSE Linux Enterprise Server 12。 您第一次啟動虛擬機器之前，您必須指定虛擬機器應該使用 Microsoft UEFI 憑證授權單位。 在提升權限的 Windows PowerShell 提示中輸入：

    Set-VMFirmware [-VMName] <VMName> [-SecureBootTemplate] <MicrosoftUEFICertificateAuthority>

如需在 Hyper-V 上執行 Linux 虛擬機器的詳細資訊，請參閱 <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Hyper-V 上的 Linux 和 FreeBSD 虛擬機器</g><g id="2CapsExtId3" ctype="x-title"></g></g>。


## 虛擬機器設定版本

當您將虛擬機器從執行 Windows 8.1 的主機移動或匯入至執行 Hyper-V 的 Windows 10 主機時，虛擬機器的設定檔案不會自動升級。 這讓虛擬機器可以移回執行 Windows 8.1 的主機。 您將無法使用新的 Hyper-V 功能搭配虛擬機器，直到手動更新虛擬機器設定版本為止。

虛擬機器設定版本代表虛擬機器設定的 Hyper-V 版本、儲存的狀態，以及與之相容的快照檔案。 設定版本 5 的虛擬機器與 Windows 8.1 相容，可以在 Windows 8.1 和 Windows 10 上執行。 設定版本 6 的虛擬機器與 Windows 10 相容，無法在 Windows 8.1 上執行。

### 檢查設定版本

在已提高權限的命令提示字元中執行下列命令：

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### 升級設定版本

從提升權限的 Windows PowerShell 提示字元，執行下列其中一個命令：

``` 
Update-VmConfigurationVersion <VMName>
```

或是

``` 
Update-VmConfigurationVersion <VMObject>
```

> <g id="1" ctype="x-strong">重要事項：</g>

- 升級虛擬機器設定版本之後，無法將虛擬機器移至執行 Windows 8.1 的主機。
- 您無法將虛擬機器設定版本從版本 6 降級至版本 5。
- 您必須關閉虛擬機器，才能升級虛擬機器設定。
- 升級之後，虛擬機器會使用新的設定檔案格式。 如需詳細資訊，請參閱<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">設定檔案格式</g><g id="2CapsExtId3" ctype="x-title"></g></g>。


## <g id="1" ctype="x-html"></g><g id="2" ctype="x-html"></g>設定檔案格式

現在，虛擬機器有新的設定檔案格式，這種格式是為了提高讀取和寫入虛擬機器設定資料的效率而設計。 它也設計為會降低儲存體失敗時的資料損毀可能性。 新的設定檔案在虛擬機器設定資料使用 .VMCX 副檔名，在執行階段狀態資料使用 .VMRS 副檔名。

> <g id="1" ctype="x-strong">重要事項：</g>.VMCX 檔案是一種二進位格式。 不支援直接編輯 .VMCX 或 .VMRS 檔案。





<!--HONumber=May16_HO1-->


