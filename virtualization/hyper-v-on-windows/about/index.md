---
title: "Windows 10 上的 Hyper-V 簡介"
description: "Hyper-V、虛擬化和相關技術的簡介。"
keywords: windows 10, hyper-v
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: 918fe27f7aee74e3c1de3b7381e7ccea76f49c73
ms.sourcegitcommit: d5f30aa1bdfb34dd9e1909d73b5bd9f4153d6b46
ms.translationtype: HT
ms.contentlocale: zh-TW
---
# <a name="introduction-to-hyper-v-on-windows-10"></a>Windows 10 上的 Hyper-V 簡介

> Hyper-V 取代了 Microsoft Virtual PC。 

無論您是軟體開發人員、IT 專業人員還是科技愛好者，都很有可能需要執行多個作業系統。 與其將實體硬體都奉獻給每部機器，不如讓 Hyper-V 在 Windows 上以虛擬機器形式執行作業系統或電腦系統。  

![](media/HyperVNesting.png)

Hyper-V 專門提供硬體虛擬化。  這表示每個虛擬機器都是在虛擬硬體上執行。  Hyper-V 可讓您建立虛擬硬碟、虛擬交換器，以及許多其他虛擬裝置，這些全都可以加入虛擬機器。

## <a name="reasons-to-use-virtualization"></a>使用虛擬化的原因

虛擬化可讓您：  
* 執行需要舊版 Windows 或非 Windows 作業系統的軟體。 

* 試用其他作業系統。 透過 Hyper-V，要建立或移除不同的作業系統，都非常容易。

* 使用多部虛擬機器在多個作業系統上測試軟體。 有了 Hyper-V，只要在一部桌上型或膝上型電腦就能全部加以執行。 這些虛擬機器可在匯出後匯入任何其他 Hyper-V 系統中，包括 Azure。

* 對任何 Hyper-V 部署中的虛擬機器進行疑難排解。 您可以從實際執行環境中匯出虛擬機器、在執行 Hyper-V 的桌上型電腦上加以開啟、對虛擬機器進行疑難排解，然後再將其匯回實際執行環境中。 

* 使用虛擬網路功能，可讓您針對測試/開發/示範用途建立一個多重電腦環境，同時確定不會影響到生產網路。

## <a name="system-requirements"></a>系統需求
Hyper-V 適用於 64 位元版 Windows 8 及更高版本的 Windows 專業版、企業版和教育版。  但不適用於 Windows 家用版。  

>  請開啟 **\[設定\]** > **\[更新與安全性\]** > **\[啟用\]**，將 Windows 10 家用版升級為 Windows 10 專業版。 您可以在此瀏覽市集並購買升級。

大部分電腦都會執行 Hyper-V，但是虛擬機器會需要大量資源，因為它是執行完整的作業系統。  您通常可以在配備 4GB RAM 的電腦上，執行一或多個虛擬機器，但是您需要更多資源，才能執行額外的虛擬機器，或是安裝及執行耗用大量資源的軟體，例如遊戲、視訊編輯或工程設計軟體。 

您的電腦需要第二層位址轉譯 (SLAT)，這是 Intel 和 AMD 當前推出的 64 位元處理器所具備的一項功能。  您也會需要 64 位元版本的 Windows。

如需 Hyper-V 系統需求以及如何驗證電腦上有執行 Hyper-V 的詳細資訊，請參閱 [Hyper-V 需求參考資料](..\reference\hyper-v-requirements.md)。

## <a name="operating-systems-you-can-run-in-a-virtual-machine"></a>可在虛擬機器中執行的作業系統
「客體」一詞是指虛擬機器，「主機」則是指執行虛擬機器的電腦。 Windows 上的 Hyper-V 可支援許多不同的客體作業系統，包括多種不同版本的 Linux、FreeBSD 和 Windows。 

提醒您，您在 VM 中使用的任何作業系統，都必須是經過有效授權的。 

若要了解 Windows 上的 Hyper-V 中可支援哪些作業系統做為客體，請參閱[支援的 Windows 客體作業系統](supported-guest-os.md)和[支援的 Linux 客體作業系統](https://technet.microsoft.com/library/dn531030.aspx)。 


## <a name="differences-between-hyper-v-on-windows-and-hyper-v-on-windows-server"></a>Windows 上的 Hyper-V 與 Windows Server 上的 Hyper-V 有何差異
有些功能在 Windows 上的 Hyper-V 中和在 Windows Server 上執行的 Hyper-V 之中，會有不同的運作方式。 

Windows 上的 Hyper-V 有不同的記憶體管理模型。 在伺服器上管理 Hyper-V 記憶體時，會假設只有虛擬機器在伺服器上執行。 在 Windows 上的 Hyper-V 中，管理記憶體時會假設大部分的用戶端機器除了執行虛擬機器以外，還會在主機上執行軟體。 例如，開發人員可能會在相同的電腦上執行 Visual Studio 和數個虛擬機器。

Windows Server 上的 Hyper-V 所包含的某些功能，並未包含在 Windows 上的 Hyper-V 中。 這些地方包括：

* 使用 RemoteFX 將 GPU 虛擬化 
* 不同主機之間的虛擬機器即時移轉
* Hyper-V 複本
* 虛擬光纖通道
* SR-IOV 網路功能
* 共用 .VHDX

## <a name="limitations"></a>限制
虛擬化的使用有其限制。 依賴特定硬體的功能或應用程式，將無法在虛擬機器中正常運作。 例如，需要以 GPU 處理的遊戲或應用程式可能無法正常運作。 此外，應用程式若依賴低於 10 毫秒的計時器 (如即時混音應用程式) 或高精確度的時間，在虛擬機器中執行時可能會有問題。

此外，如果您啟動 Hyper-V，對延遲性敏感、高精確性的應用程式在主機中執行時可能也會發生問題。  這是因為啟動虛擬化後，主機 OS 也會在 Hyper-V 虛擬化層頂端執行，就像客體作業系統一樣。 不過，和客體不同，主機 OS 的特殊之處在於其可以直接存取所有的硬體，這表示具有特殊硬體需求的應用程式仍然可以在主機 OS 中執行而不發生問題。

## <a name="next-step"></a>後續步驟
[在 Windows 10 上安裝 Hyper-V](..\quick-start\enable-hyper-v.md) 
