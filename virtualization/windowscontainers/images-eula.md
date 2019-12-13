# <a name="microsoft-software-supplemental-license-for-windows-container-base-image"></a>WINDOWS 容器基底映射的 MICROSOFT 軟體補充授權

此補充授權適用于 Windows 容器基底映射（「容器映射」）。 如果您遵守此補充授權條款，您可以使用容器映射，如下所述。

## <a name="container-os-image"></a>容器 OS 映像
容器映射只能與的有效授權複本搭配使用：

Windows Server Standard 或 Windows Server Datacenter 軟體（統稱為「伺服器主機軟體」）或 Microsoft Windows 作業系統（版本10）軟體（以下稱「用戶端主機軟體」），或 Windows 10 IoT 企業版和 Windows 10 IoT 核心版（統稱為「IoT」主機軟體」）。
伺服器主機軟體、用戶端主機軟體和 IoT 主機軟體統稱為「主機軟體」，而主機軟體的授權則是「主機授權」。

如果您沒有主機授權的對應版本，您可能不會使用容器映射。 某些限制和其他條款可能適用，如這裡所述。 若授權條款與主機授權衝突，則此補充授權應遵循容器映射的規範。 藉由接受此補充授權或使用容器映射，即表示您同意所有這些條款。 如果您不接受並遵守這些條款，您可能不會使用容器映射。

## <a name="definitions"></a>定義
**Windows Server 容器**（不含 hyper-v 隔離）是 Microsoft windows server 軟體的一項功能。

**具有 Hyper-v 隔離的 Windows Server 容器。** Microsoft Windows Server 授權條款的第2節（k）已完全刪除，並以修改過的詞彙取代，如下面「更新」所示。

已更新：具有 Hyper-v 隔離（先前稱為 Hyper-v 容器）的 Windows Server 容器是 Windows Server 中的容器技術，利用虛擬作業系統環境來裝載一或多個 Windows Server 容器。 用來裝載一或多個 Windows Server 容器的每個 Hyper-v 隔離實例會被視為一個虛擬作業系統環境。

## <a name="license-terms"></a>授權條款
**主機授權。** 主機授權條款適用于您使用容器映射和任何以容器映射建立的 Windows 容器，這是不同的，而且與虛擬機器不同。

**使用權利。** 容器映射可用來建立隔離的虛擬化 Windows 作業系統環境，其中至少包含一個應用程式，可新增主要和重要的功能。 您只能使用容器映射來建立、建立和執行主機軟體上的 Windows 容器。 主機軟體的更新可能不會更新容器映射，因此您可以根據更新的容器映射重新建立任何 Windows 容器。

**限制.** 您不得從容器映射移除此補充授權檔檔案。 您可能不會對您在容器內執行的應用程式啟用遠端存取，以避免適用的授權費用。 貴使用者不得對容器映射進行還原工程、反編譯或拆解，或嘗試執行這項操作，但僅限於協力廠商授權條款所需的範圍，這些是管理軟體可能隨附之特定開放原始碼元件的使用。 可能適用主機授權中的其他限制。

## <a name="additional-terms"></a>其他條款
**用戶端主機軟體。** 在用戶端主機軟體上執行容器映射時，您可以只將任何數目的容器映射具現化為 Windows 容器，以供測試或開發之用。 您不能在用戶端主機軟體的生產環境中使用這些 Windows 容器。

**IoT 主機軟體。** 在 IoT 主機軟體上執行容器映射時，您可以只將任何數目的容器映射具現化為 Windows 容器，以供測試或開發之用。 如果您已同意 Microsoft 商業用 Windows 10 核心執行時間映射或 Windows 10 IoT 企業版裝置授權（「Windows IoT 商業合約」），則只能在生產環境中使用容器映射。 Windows IoT 商業協定中的其他條款和限制適用于在生產環境中使用容器映射。

**協力廠商軟體。** 容器映射可能包括以此補充授權或其專屬條款授權給貴使用者的協力廠商應用程式。 協力廠商應用程式的授權條款、注意事項和通知（如果有的話）可在 http://aka.ms/thirdpartynotices 或隨附的通知檔案中線上存取。 即使這類應用程式是由其他合約所控管，但免責聲明、限制和主機授權中的損毀責任也適用于適用法律允許的範圍。

**開放原始碼元件。** 容器映射可能包含以開放原始碼授權為依據的協力廠商受著作權軟體，並具有原始程式碼可用性義務。 這些授權的複本會包含在 ThirdPartyNotices 檔案或其他隨附的通知檔案中。 如有相關的開放原始碼授權，您可以透過傳送 money 訂單或檢查 $5.00 至：原始程式碼合規性小組、Microsoft Corporation、1 Microsoft 的方式、Redmond、WA 98052 （美國），取得 Microsoft 提供的完整對應原始程式碼。 請在您付款的備忘行中，包含「Windows 容器基底映射的 Microsoft 軟體補充授權」的名稱、開放原始碼元件名稱和版本號碼。 您也可以在 http://aka.ms/getsource 找到來源的複本。
