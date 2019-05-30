# <a name="hyper-v-backup-approaches"></a>HYPER-V 備份方法
HYPER-V 可讓您備份虛擬機器，從主機作業系統，而不需要執行虛擬機器內的自訂備份軟體。  有數種方法可供開發人員可以利用依其需求而定。
## <a name="hyper-v-vss-writer"></a>HYPER-V VSS 寫入器
HYPER-V 會在所有版本在受支援 HYPER-V 的 Windows Server 上實作 VSS 寫入器。  這個 VSS 寫入器可讓開發人員可以利用現有的 VSS 基礎架構，以備份虛擬機器。  不過，它是針對小型的縮放比例備份的作業，其中的伺服器上的所有虛擬機器都備份同時設計。

若要了解這個架構更好 – 請參閱此簡報：https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>HYPER-V WMI 基礎備份
從 Windows Server 2016 中，HYPER-V，開始支援透過 HYPER-V WMI API 的備份。  這種方式仍在虛擬機器內使用 VSS 利用做為備份目的，但不再使用 VSS 主機作業系統中。  相反地，參考點和具彈性的變更追蹤 (RCT) 的組合來讓開發人員存取的相關資訊以有效率的方式備份虛擬機器。  這種方式是比使用 VSS 主機中更具彈性，但是它只適用於 Windows Server 2016 和更新版本。

若要了解這個架構更好 – 請參閱此簡報：https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

還有有關如何使用這些 Api 可用以下的範例：https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>從 WMI 基礎的備份讀取備份方法
建立虛擬機器備份使用 HYPER-V WMI 時，有三種方法來從備份讀取的實際的資料。  每一個都有唯一的優點和缺點。
### <a name="wmi-export"></a>WMI 匯出
開發人員可以匯出備份資料透過 HYPER-V WMI 介面 （如上述範例中使用）。  HYPER-V 會編譯到虛擬硬碟所做的變更，並將檔案複製到要求的位置。  這個方法使用簡單、 適用於所有案例和是遠端。  不過，虛擬硬碟產生通常會建立大量的資料，以透過網路傳輸。
### <a name="win32-apis"></a>Win32 Api
開發人員可以使用 SetVirtualDiskInformation，GetVirtualDiskInformation 和虛擬硬碟 Win32 API 上 QueryChangesVirtualDisk Api 所記載在這裡設定：https://docs.microsoft.com/windows/desktop/api/_vhd/請注意，若要使用這些 Api，HYPER-V WMI 仍然必須用來建立參考相關聯的虛擬機器上的點。  這些 Win32 Api 然後允許有效率地存取備份虛擬機器的資料。  Win32 Api 有數個限制：
*   它們在本機只能存取
*   不要支援讀取的資料共用虛擬硬碟檔案
*   它們會傳回資料位址，都會在相對於內部結構的虛擬硬碟

### <a name="remote-shared-virtual-disk-protocol"></a>遠端共用的虛擬磁碟通訊協定
最後，如果開發人員需要來有效率地存取備份資料資訊從共用虛擬硬碟檔案 – 他們將需要使用遠端共用虛擬磁碟通訊協定。  此通訊協定會說明[以下](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)。
