---
title: "簡介 Windows 10 上的 Hyper-V"
description: "Windows 10 上的 Hyper-V 簡介。"
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
translationtype: Human Translation
ms.sourcegitcommit: 9f1073ecfa4cf836295de81f2bcf9622274ca034
ms.openlocfilehash: ea19b576219755e09a4064d0fa6bfde5367967e1

---

# 簡介 Windows 10 上的 Hyper-V

無論您是軟體開發人員、IT 專業人員還是科技愛好者，都很有可能需要執行多個作業系統，有時還是在許多不同的電腦上。 並非所有人都可以取得一套完整的實驗室來存放所有的電腦。 因此，您可以將電腦作為 Hyper-V *虛擬機器* (VM) 執行，透過虛擬化技術節省空間和時間，而不需專用的實體硬體來執行這項作業。

> Microsoft 虛擬電腦將於 2017 年 4 月停止提供服務。 支援的取代項目為 Winodws 10 上的 Hyper-V。 

## 虛擬化的使用
虛擬化可讓任何人輕鬆維護由眾多作業系統、軟體設定和硬體設定組成的多個測試環境。  Hyper-V 不僅提供 Windows 上的虛擬化功能，也提供一項簡單的機制可快速地在這些環境之間切換，且無需額外的硬體花費。    

Hyper-V 有多種使用方式。 例如：

- 在單一桌上型電腦或筆記型電腦上，可建立由多個虛擬機器組成的測試環境。 測試完成後，這些虛擬機器可在匯出後匯入任何其他 Hyper-V 系統中。

- 開發人員可在其電腦上使用 Hyper-V，對多個作業系統測試軟體。 例如，如果您有一個應用程式必須在 Windows 8、Windows 7 和 Linux 作業系統上進行測試，您可以在開發系統上建立多個虛擬機器，並使其分別包含前述的各個作業系統。

- 您可以使用 Windows 10 上的 Hyper-V 對任何 Hyper-V 部署中的虛擬機器進行疑難排解。 您可以從實際執行環境中匯出虛擬機器、在執行 Hyper-V 的桌上型電腦上加以開啟，然後再將其匯回實際執行環境中。 

- 使用虛擬網路功能，可讓您針對測試/開發/示範用途建立一個多重電腦環境，同時確定不會影響到生產網路。

- 愛好者可用 Hyper-V 來試用其他作業系統。 透過 Hyper-V，要整合或拆解不同的作業系統，都非常容易。

- 您可以在筆記型電腦上使用 Hyper-V，來示範舊版的 Windows 或非 Windows 作業系統。 


## 系統需求
Hyper-V 需要具有第二層位址轉譯 (SLAT) 的 64 位元系統。 SLAT 是 Intel 和 AMD 當前推出的 64 位元處理器所具備的一項功能。 您還須具備 64 位元版的 Windows 8 或更新版本，和至少 4GB 的 RAM。 Hyper-V 不支援在 VM 中同時建立 32 位元和 64 位元作業系統。

Hyper-V 的動態記憶體可讓 VM 所需的記憶體能夠動態配置和取消配置 (由您指定最小值和最大值)，以及在 VM 間共用未使用的記憶體。 在具備 4GB RAM 的機器上可執行 3 或 4 個 VM，但要執行 5 個或更多 VM，則需要更多 RAM。 反之，您也可以建立具有 32 個處理器和 512GB RAM 的大型 VM，視您的實體硬體而定。

## 可在虛擬機器中執行的作業系統
「客體」一詞是指虛擬機器，「主機」則是指執行虛擬機器的電腦。 Windows 上的 Hyper-V 可支援許多不同的客體作業系統，包括多種不同版本的 Linux、FreeBSD 和 Windows。 如需了解哪些作業系統可在 Windows 上作為 Hyper-V 中的客體，請參閱[支援的 Windows 客體作業系統](supported_guest_os.md)和[ Hyper-V 的 Linux 和 FreeBSD 虛擬機器](https://technet.microsoft.com/library/dn531030.aspx)。 

## Windows 上的 Hyper-V 與 Windows Server 上的 Hyper-V 有何差異
有些功能在 Windows 上的 Hyper-V 中和在 Windows Server 上執行的 Hyper-V 之中，會有不同的運作方式。 這些限制或考量包括：

- Windows 上的 Hyper-V 有不同的記憶體管理模型。 在伺服器上管理 Hyper-V 記憶體時，會假設只有虛擬機器在伺服器上執行。 在 Windows 上的 Hyper-V 中，管理記憶體時會假設大部分的用戶端機器除了執行虛擬機器以外，還會執行軟體。 例如，開發人員可能會在相同的電腦上執行 Visual Studio 和數個虛擬機器。

- SR-IOV 在 64 位元客體上可正常運作，但 32 位元則否，且不受支援。

### 無法在 Windows Hyper-V 中使用的 Windows Server 功能
Windows Server 上的 Hyper-V 所包含的某些功能，並未包含在 Windows 上的 Hyper-V 中。 這些限制或考量包括：

- 使用 RemoteFX 將 GPU 虛擬化 

- 不同主機之間的虛擬機器即時移轉

- Hyper-V 複本

- 虛擬光纖通道

- SR-IOV 網路功能

- 共用 .VHDX

> **警告**：在 Hyper-V 上執行的虛擬機器不會自動處理從有線網路到無線網路的移轉。 您必須手動變更虛擬機器網路介面卡。

## 限制
虛擬化的使用有其限制。 依存於特定硬體的功能或應用程式，將無法在 VM 中正常運作。 例如，需要以 GPU 處理的遊戲或應用程式 (未提供軟體後援) 可能無法正常運作。 此外，應用程式若依賴低於 10 毫秒的計時器，例如對延遲性敏感、高精確性的應用程式 (如即時混音應用程式)，在 VM 中執行時可能會有問題。

此外，如果您啟動虛擬化，對延遲性敏感、高精確性的應用程式在主機 OS 中執行時可能也會有問題。 (這是因為啟動虛擬化後，主機 OS 也會在 Hyper-V 虛擬化層頂端執行，就像客體作業系統一樣。 不過，和客體不同，主機 OS 的特殊之處在於其可以直接存取所有的硬體，這表示具有特殊硬體需求的應用程式仍然可以在主機 OS 中執行而不發生問題。)

提醒您，您在 VM 中使用的任何作業系統，都必須是經過有效授權的。

## 下一個步驟
[逐步解說：Windows 10 上的 Hyper-V](..\quick_start\walkthrough.md) 

了解 Windows 10 上的 Hyper-V 有何[新功能](whats_new.md)。




<!--HONumber=Jun16_HO4-->


