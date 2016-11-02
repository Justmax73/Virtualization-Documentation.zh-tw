---
title: "簡介 Windows 10 上的 Hyper-V"
description: "Windows 10 上的 Hyper-V 簡介。"
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 702894a9f1d89d1f6a5846048f9dd794d9d1a59c

---

# 簡介 Windows 10 上的 Hyper-V

無論您是軟體開發人員、IT 專業人員還是科技愛好者，都很有可能需要執行多個作業系統。  與其將實體硬體都奉獻給每部機器，不如讓 Hyper-V 在您的 Windows 電腦以虛擬機器 (VM) 形式執行多個作業系統。

> Microsoft 虛擬電腦將於 2017 年 4 月停止提供服務。 Windows 10 企業版與 Windows 10 專業版上的 Hyper-V 會成為支援的替代方案。  

## 使用虛擬化的原因
虛擬化可讓任何人在同一部實體機器上執行多個作業系統、軟體設定和硬體設定。  Hyper-V 同時提供虛擬化及管理虛擬機器的工具。

Hyper-V 有多種使用方式。 例如：

* 執行需要舊版 Windows 或非 Windows 作業系統的軟體。 

* 試用其他作業系統。 透過 Hyper-V，要建立或移除不同的作業系統，都非常容易。

* 使用多部虛擬機器在多個作業系統上測試軟體。 有了 Hyper-V，只要在一部桌上型或膝上型電腦就能全部加以執行。 這些虛擬機器可在匯出後匯入任何其他 Hyper-V 系統中，包括 Azure。

* 對任何 Hyper-V 部署中的虛擬機器進行疑難排解。 您可以從實際執行環境中匯出虛擬機器、在執行 Hyper-V 的桌上型電腦上加以開啟、對虛擬機器進行疑難排解，然後再將其匯回實際執行環境中。 

* 使用虛擬網路功能，可讓您針對測試/開發/示範用途建立一個多重電腦環境，同時確定不會影響到生產網路。

## 系統需求
Hyper-V 僅適用於 Windows 8 及更高版本的 Windows 專業版、企業版及教育版。

其需要具有第二層位址轉譯 (SLAT) 的 64 位元系統。 SLAT 是 Intel 和 AMD 當前推出的 64 位元處理器所具備的一項功能。  您也會需要 64 位元版本的 Windows。  
也就是說，Hyper-V 在虛擬機器內不同時支援 32 位元和 64 位元的作業系統。

您可以在具有 4GB RAM 的主機上執行 3 或 4 部基本虛擬機器，但更多虛擬機器會需要更多資源。 反之，您也可以建立具有 32 個處理器和 512GB RAM 的大型虛擬機器，視您的實體硬體而定。

如需 Hyper-V 系統需求及如何驗證您電腦上之 Hyper-V 作業的詳細資訊，請參閱[逐步解說：Windows 10 Hyoer-V 系統需求](..\quick_start\walkthrough_install.md)。


## 可在虛擬機器中執行的作業系統
「客體」一詞是指虛擬機器，「主機」則是指執行虛擬機器的電腦。 Windows 上的 Hyper-V 可支援許多不同的客體作業系統，包括多種不同版本的 Linux、FreeBSD 和 Windows。 

提醒您，您在 VM 中使用的任何作業系統，都必須是經過有效授權的。 

如需了解哪些作業系統可在 Windows 上作為 Hyper-V 中的客體，請參閱[支援的 Windows 客體作業系統](supported_guest_os.md)和[ Hyper-V 的 Linux 和 FreeBSD 虛擬機器](https://technet.microsoft.com/library/dn531030.aspx)。 


## Windows 上的 Hyper-V 與 Windows Server 上的 Hyper-V 有何差異
有些功能在 Windows 上的 Hyper-V 中和在 Windows Server 上執行的 Hyper-V 之中，會有不同的運作方式。 

Windows 上的 Hyper-V 有不同的記憶體管理模型。 在伺服器上管理 Hyper-V 記憶體時，會假設只有虛擬機器在伺服器上執行。 在 Windows 上的 Hyper-V 中，管理記憶體時會假設大部分的用戶端機器除了執行虛擬機器以外，還會在主機上執行軟體。 例如，開發人員可能會在相同的電腦上執行 Visual Studio 和數個虛擬機器。

### 僅適用於 Windows Server 的 Hyper-V 功能
Windows Server 上的 Hyper-V 所包含的某些功能，並未包含在 Windows 上的 Hyper-V 中。 這些地方包括：

* 使用 RemoteFX 將 GPU 虛擬化 
* 不同主機之間的虛擬機器即時移轉
* Hyper-V 複本
* 虛擬光纖通道
* SR-IOV 網路功能
* 共用 .VHDX

## 限制
虛擬化的使用有其限制。 依賴特定硬體的功能或應用程式，將無法在虛擬機器中正常運作。 例如，需要以 GPU 處理的遊戲或應用程式可能無法正常運作。 此外，應用程式若依賴低於 10 毫秒的計時器 (如即時混音應用程式) 或高精確度的時間，在虛擬機器中執行時可能會有問題。

此外，如果您啟動 Hyper-V，對延遲性敏感、高精確性的應用程式在主機中執行時可能也會發生問題。  這是因為啟動虛擬化後，主機 OS 也會在 Hyper-V 虛擬化層頂端執行，就像客體作業系統一樣。 不過，和客體不同，主機 OS 的特殊之處在於其可以直接存取所有的硬體，這表示具有特殊硬體需求的應用程式仍然可以在主機 OS 中執行而不發生問題。

## 下一個步驟
[逐步解說：在 Windows 10 上安裝 Hyper-V](..\quick_start\walkthrough_install.md) 

了解 Windows 10 上的 Hyper-V 有何[新功能](whats_new.md)。




<!--HONumber=Oct16_HO4-->


