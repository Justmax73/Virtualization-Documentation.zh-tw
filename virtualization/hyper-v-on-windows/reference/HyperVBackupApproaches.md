# <a name="hyper-v-backup-approaches"></a>Hyper-v 備份方法
Hyper-v 可讓您從主機作業系統備份虛擬機器，而不需要在虛擬機器內執行自訂備份軟體。  有數種方法可供開發人員使用，視其需求而定。
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS 寫入器
Hyper-v 會在支援 Hyper-v 的所有 Windows Server 版本上，執行 VSS 寫入器。  此 VSS 寫入器可讓開發人員利用現有的 VSS 基礎結構來備份虛擬機器。  不過，它是針對小規模備份作業所設計，其中伺服器上的所有虛擬機器都會同時備份。

進一步瞭解此架構–請參閱此簡報： https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>以 hyper-v WMI 為基礎的備份
從 Windows Server 2016 開始，Hyper-v 已開始支援透過 Hyper-v WMI API 進行備份。  這種方法仍會在虛擬機器內利用 VSS 來進行備份，但不再使用主機作業系統中的 VSS。  相反地，參考點和復原變更追蹤（.RCT）的組合是用來讓開發人員以有效率的方式存取備份虛擬機器的相關資訊。  這種方法比使用主機中的 VSS 更具擴充性，但僅適用于 Windows Server 2016 和更新版本。

進一步瞭解此架構–請參閱此簡報： https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

您也可以在這裡找到如何使用這些 Api 的範例： https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>從 WMI 型備份讀取備份的方法
使用 Hyper-v WMI 建立虛擬機器備份時，有三種方法可從備份讀取實際資料。  每個都有獨特的優點和缺點。
### <a name="wmi-export"></a>WMI 匯出
開發人員可以透過 Hyper-v WMI 介面（如上述範例所使用）匯出備份資料。  Hyper-v 會將變更編譯成虛擬硬碟，並將檔案複製到要求的位置。  這個方法很容易使用，適用于所有案例，而且可遠端處理。  不過，所產生的虛擬硬碟通常會建立大量資料，以透過網路傳輸。
### <a name="win32-apis"></a>Win32 Api
開發人員可以將 SetVirtualDiskInformation、GetVirtualDiskInformation 和 QueryChangesVirtualDisk Api 用於虛擬硬碟 WIN32 API 集，如下所述： https://docs.microsoft.com/windows/desktop/api/_vhd/ 請注意，若要使用這些 Api，Hyper-v WMI 仍然需要用來在相關聯的虛擬機器上建立參考點。  然後，這些 Win32 Api 會允許有效率地存取已備份虛擬機器的資料。  Win32 Api 有幾項限制：
* 只能在本機存取
* 不支援從共用虛擬硬碟檔案讀取資料
* 它們會傳回相對於虛擬硬碟之內部結構的資料位址

### <a name="remote-shared-virtual-disk-protocol"></a>遠端共用虛擬磁片通訊協定
最後，如果開發人員需要有效率地從共用虛擬硬碟檔案存取備份資料資訊，則需要使用遠端共用虛擬磁片通訊協定。  此通訊協定記載在[這裡](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)。
