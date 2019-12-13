# <a name="hyper-v-architecture"></a>Hyper-V 架構

Hyper-V 是以 Hypervisor 為基礎的虛擬化技術，適用於特定 x64 版 Windows。  Hypervisor 是虛擬化的核心。  這是處理器特定的虛擬化平台，可讓多個獨立的作業系統共用單一硬體平台。

就磁碟分割而言，Hyper-V 可支援隔離。 磁碟分割是隔離的邏輯單元，為 Hypervisor 所支援，是作業系統執行的所在之處。 Microsoft 的 Hypervisor 必須至少有一個父磁碟分割或根磁碟分割執行 Windows。 虛擬化管理堆疊會在父磁碟分割中執行，並可直接存取硬體裝置。 根磁碟分割則會建立用來裝載客體作業系統的子磁碟分割。 根磁碟分割會使用 Hypercall 應用程式開發介面 (API) 來建立子磁碟分割。

磁碟分割無法存取實體處理器，也不能操控處理器插斷。 反之，磁碟分割具有處理器的虛擬檢視，並且在專屬於每個客體磁碟分割的虛擬記憶體位址區域中執行。 Hypervisor 會操控處理器插斷，並將其重新導向至各自的磁碟分割。 Hyper-V 也可以使用輸入/輸出記憶體管理單元 (IOMMU) (其操作不受 CPU 使用的記憶體管理硬體所影響)，為各種客體虛擬位址空間之間的位址轉譯進行硬體加速。 IOMMU 可用來將實體記憶體位址重新對應至子磁碟分割所使用的位址。

子磁碟分割也無法直接存取其他硬體資源，只能以虛擬方式檢視資源，將其做為虛擬裝置 (VDev)。 對虛擬裝置提出的要求，會透過 VMBus 或 Hypervisor 重新導向至父磁碟分割中的裝置，以處理要求。 VMBus 是磁碟分割之間的邏輯通訊通道。 父磁碟分割裝載虛擬化服務提供者 (VSP)，可透過 VMBus 進行通訊，以處理來自子磁碟分割的裝置存取要求。 子磁碟分割裝載虛擬化服務消費者 (VSC)，可透過 VMBus 將裝置要求重新導向至父磁碟分割中的 VSP。 客體作業系統看不到這整個程序。

虛擬裝置也可以利用名為「啟發式 I/O」的 Windows Server 虛擬化功能來進行儲存、網路設定、繪圖和輸入子系統。 「啟發式 I/O」是高階通訊協定 (例如 SCSI) 的特殊感知虛擬化實作，可跳過任何裝置模擬層，直接使用 VMBus。 這使得通訊更有效率，但是需要可感知 Hypervisor 和 VMBus 的啟發式客體。 安裝 Hyper-V 整合服務即可提供 Hyper-V 啟發式 I/O 和 Hypervisor 感知核心。 整合元件 (包括虛擬伺服器用戶端 (VSC) 驅動程式) 也可供其他用戶端作業系統使用。 Hyper-V 需要包含硬體輔助虛擬化 (例如 Intel VT 或 AMD 虛擬化 (AMD-V) 技術) 的處理器。

下圖提供 Hyper-V 環境架構的高階概觀。

![](./media/hv_architecture.png)

## <a name="glossary"></a>詞彙
* **APIC** – 進階可程式化插斷控制器 (Advanced Programmable Interrupt Controller) – 此裝置可允許為其插斷輸出指派優先順序層級。
* **子磁碟分割 (Child Partition)** – 裝載客體作業系統的磁碟分割 - 子磁碟分割對實體記憶體和裝置的所有存取權，都是透過虛擬機器匯流排 (VMBus) 或 Hypervisor 來提供。
* **Hypercall** – 用來與 Hypervisor 通訊的介面 - Hypercall 介面可供存取 Hypervisor 所提供的最佳化。
* **Hypervisor** – 位於硬體與一或多個作業系統之間的軟體層。 其主要工作是提供稱為磁碟分割的隔離執行環境。 Hypervisor 可控制及仲裁對基礎硬體的存取權。
* **IC** – 整合元件 (Integration component) – 可讓子磁碟分割與其他磁碟分割和 Hypervisor 通訊的元件。
* **I/O 堆疊 (I/O stack)** – 輸入/輸出堆疊
* **MSR** – 記憶體服務常式 (Memory Service Routine)
* **根磁碟分割 (Root Partition)** – 有時稱為父磁碟分割。  管理機器層級的功能，例如裝置驅動程式、電源管理，以及熱新增/移除裝置。 根 (或父) 磁碟分割是唯一可以直接存取實體記憶體和裝置的磁碟分割。
* **VID** – 虛擬化基礎結構驅動程式 (Virtualization Infrastructure Driver) – 為磁碟分割提供磁碟分割管理服務、虛擬處理器管理服務，以及記憶體管理服務。
* **VMBus** – 通道型通訊機制，可在具有多個使用中虛擬化磁碟分割的系統上，用來進行磁碟分割間的通訊和裝置列舉。 VMBus 會隨著 Hyper-V 整合服務一起安裝。
* **VMMS** – 虛擬機器管理服務 (Virtual Machine Management Service) – 負責管理子磁碟分割中所有虛擬機器的狀態。
* **VMWP** – 虛擬機器工作者處理序 (Virtual Machine Worker Process) – 虛擬化堆疊的使用者模式元件。 工作者處理序會將父磁碟分割中 Windows Server 2008 執行個體的虛擬機器管理服務，提供給子磁碟分割中的客體作業系統。 虛擬機器管理服務會針對每個執行中的虛擬機器，繁衍個別的工作者處理序。
* **VSC** – 虛擬化服務用戶端 (Virtualization Service Client) – 位於子磁碟分割中的綜合裝置執行個體。 VSC 會使用父磁碟分割中虛擬化服務提供者 (VSP) 所提供的硬體資源。 它們會透過 VMBus 來與父磁碟分割中的對應 VSP 通訊，以滿足子磁碟分割的裝置 I/O 要求。
* **VSP** – 虛擬化服務提供者 (Virtualization Service Provider) – 位於根磁碟分割中，可透過虛擬機器匯流排 (VMBus) 為子磁碟分割提供綜合裝置支援。
* **WinHv** – Windows Hypervisor 介面程式庫 (Windows Hypervisor Interface Library) - WinHv 基本上是分割作業系統的驅動程式與 Hypervisor 之間的橋接器，可讓驅動程式使用標準 Windows 呼叫慣例來呼叫 Hypervisor。
* **WMI** – 虛擬機器管理服務會公開一組 Windows Management Instrumentation (WMI) 型 API 來管理及控制虛擬機器。
