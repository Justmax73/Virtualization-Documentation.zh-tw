# <a name="hyper-v-backup-approaches"></a>HYPER-V 備份方法
HYPER-V 可讓您備份的虛擬機器從主機作業系統，而不需要執行自訂的備份軟體內的虛擬機器。  有數種方式可供開發人員可以運用根據使用者的需求而定。
## <a name="hyper-v-vss-writer"></a>HYPER-V VSS 編寫器
HYPER-V 實作 VSS 編寫器上所有版本的 Windows Server HYPER-V 支援的位置。  此 VSS 編寫器允許開發人員運用現有的 VSS 基礎結構來備份的虛擬機器。  不過，它的設計目的所在的伺服器上的所有虛擬機器都備份同時小規模備份作業。

若要了解此架構更妥善地 – 參照此簡報：https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>HYPER-V WMI 基礎備份
從 Windows Server 2016，HYPER-V 入門支援透過 HYPER-V WMI API 的備份。  此方法仍然在虛擬機器內使用 VSS 供備份使用，但無法再使用主機作業系統中的 [VSS。  而參照點以及可恢復變更追蹤 (RCT) 的組合用來允許存取的相關資訊的開發人員備份虛擬機器以有效率的方式。  不過，它只是可在 Windows Server 2016 及更新版本，這個方法會比在主機使用 VSS 更佳。

若要了解此架構更妥善地 – 參照此簡報：https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

如何使用這些 Api 提供以下上也有範例：https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>用於讀取 WMI 根據備份的備份方法
建立虛擬機器備份使用 HYPER-V WMI，有三種方法從備份中讀取的實際資料。  每個具有唯一的優點和缺點。
### <a name="wmi-export"></a>WMI 匯出
開發人員可以匯出備份資料透過 HYPER-V WMI 介面 （如上述範例中使用）。  HYPER-V 將編譯成虛擬硬碟變更並將檔案複製到要求的位置。  這個方法是易於使用、 適用於所有案例並且可在遠端處理。  不過，虛擬硬碟產生通常會建立大量的資料透過網路傳輸。
### <a name="win32-apis"></a>Win32 api （英文)
開發人員可以使用 SetVirtualDiskInformation、 GetVirtualDiskInformation 和 QueryChangesVirtualDisk Api 虛擬硬碟 Win32 API 在此處將所記載的設定：https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/注意到使用這些 Api、 HYPER-V WMI 仍然需要用來建立參考 （英文）相關聯的虛擬機器上的點。  這些 Win32 Api 然後允許完成備份的虛擬機器的資料存取更具效率。  Win32 Api 不要有數個限制：
*   他們只可在本機上
*   不要支援讀取資料從共用的虛擬硬碟檔案
*   傳回相對於虛擬硬碟的內部結構的資料位址

### <a name="remote-shared-virtual-disk-protocol"></a>遠端共用的虛擬磁碟通訊協定
最後，如果開發人員需要有效率地存取備份資料資訊從共用的虛擬硬碟檔案 – 他們必須使用遠端共用虛擬磁碟通訊協定。  此通訊協定所記載以下：https://msdn.microsoft.com/en-us/library/dn393384.aspx
