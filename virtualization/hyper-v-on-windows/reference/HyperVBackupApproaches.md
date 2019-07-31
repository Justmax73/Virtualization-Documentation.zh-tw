# <a name="hyper-v-backup-approaches"></a>Hyper-v 備份方法
Hyper-v 可讓您從主機作業系統備份虛擬機器, 而不需要在虛擬機器內執行自訂備份軟體。  有幾種方法可讓開發人員根據自己的需求來使用。
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS 書寫器
Hyper-v 會在支援 Hyper-v 的所有 Windows 伺服器版本上執行 VSS 寫入程式。  這個 VSS 寫入程式可讓開發人員利用現有的 VSS 基礎結構來備份虛擬機器。  不過, 它是針對在伺服器上的所有虛擬機器都同時備份的小型調整備份作業而設計。

若要更清楚地瞭解此架構, 請參閱此簡報:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-v 基於 WMI 的備份
從 Windows Server 2016 開始, Hyper-v 已透過 Hyper-v WMI API 開始支援備份。  這個方法仍會在虛擬機器中利用 VSS 進行備份, 但不會在主機作業系統中使用 VSS。  您可以改為使用參考點與彈性變更追蹤 (.RCT) 的組合, 讓開發人員以有效的方式存取有關備份的虛擬機器的資訊。  這個方法比在主機中使用 VSS 更具可伸縮性, 但它只能在 Windows Server 2016 和更新版本上使用。

若要更清楚地瞭解此架構, 請參閱此簡報:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

您也可以在以下範例中使用這些 Api:https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>從以 WMI 為基礎的備份讀取備份的方法
使用 Hyper-v WMI 建立虛擬機器備份時, 有三種方法可從備份讀取實際資料。  每個都有獨特的優點與缺點。
### <a name="wmi-export"></a>WMI 匯出
開發人員可以透過 Hyper-v WMI 介面匯出備份資料 (如上述範例所用)。  Hyper-v 會將變更編譯成虛擬硬碟, 並將檔案複製到要求的位置。  這個方法很容易使用, 適用于所有案例, 且可進行遠端操作。  不過, 產生的虛擬硬碟通常會建立大量的資料, 以便在網路上傳輸。
### <a name="win32-apis"></a>Win32 Api
開發人員可以在虛擬硬碟 WIN32 API 上使用 SetVirtualDiskInformation、GetVirtualDiskInformation 和 QueryChangesVirtualDisk Api, 如下所述: https://docs.microsoft.com/windows/desktop/api/_vhd/請注意, 若要使用這些 api, 仍需要使用 hyper-v WMI 建立參照關聯虛擬機器上的點數。  然後, 這些 Win32 Api 就能高效存取已備份的虛擬機器的資料。  Win32 Api 有幾項限制:
* 它們只能在本機存取
* 不支援從共用虛擬硬碟檔案讀取資料
* 它們會傳回相對於虛擬硬碟內部結構的資料位址

### <a name="remote-shared-virtual-disk-protocol"></a>遠端共用虛擬磁片通訊協定
最後, 如果開發人員需要高效存取共用虛擬硬碟檔案中的備份資料資訊, 就必須使用遠端共用虛擬磁片通訊協定。  此通訊協定已在[這裡](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)記錄。
