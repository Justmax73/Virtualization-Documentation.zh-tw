---
title: 容器與虛擬電腦
description: 本主題討論容器與虛擬電腦之間的一些主要相似點與差異，以及您可能會想要使用每個內容的時間。 容器和虛擬機器都有它們的用途，事實上，許多容器部署都會將虛擬機器當作主機作業系統使用，而不是直接在硬體上執行，尤其是在雲端執行容器時。
keywords: docker、容器、vm、虛擬機器
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254389"
---
# <a name="containers-vs-virtual-machines"></a>容器與虛擬電腦

本主題討論容器與虛擬電腦（Vm）之間的一些主要相似點與差異，以及您可能會想要使用每個部分的時間。 容器和 Vm 都有其用途，事實上，許多容器部署都將 Vm 當作主機作業系統使用，而不是直接在硬體上執行，尤其是在雲端執行容器時。

如需容器的概覽，請參閱[Windows 和容器](index.md)。

## <a name="container-architecture"></a>容器架構

容器是獨立的輕型思洛儲存體，可在主機作業系統上執行應用程式。 容器是在主機作業系統的內核（可以看作是作業系統的掩蔽管道）上建立，而且只包含應用程式，以及一些在使用者模式下執行的輕型作業系統 Api 與服務，如下圖所示。

![顯示樹枝如何在內核頂端執行的結構化圖表](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>虛擬機器架構

相對於容器，Vm 會執行完整的作業系統，包括它本身的核心，如下圖所示。

![顯示 Vm 在主機作業系統旁執行完整作業系統的體系結構圖表](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>容器與虛擬電腦

下表顯示這些互補技術的一些相似與差異。

|                 | 虛擬機器  | 包裝箱  |
| --------------  | ---------------- | ---------- |
| 單獨       | 提供主機作業系統與其他 Vm 的完整隔離。 當強安全性界限至關重要時（例如，將來自相同伺服器或群集的競爭性公司的 app 託管），這是很實用的方法。 | 通常會提供與主機和其他容器的輕型隔離，但不會提供與 VM 一樣強的安全性界限。 （您可以使用[hyper-v 隔離模式](../manage-containers/hyperv-container.md)來隔離輕型 VM 中的每個容器）來提高安全性。 |
| 作業系統 | 執行包括內核在內的完整作業系統，因此需要更多系統資源（CPU、記憶體和儲存空間）。 | 執行作業系統的使用者模式部分，而且只要使用較少的系統資源，就能量身定制以包含 app 所需的服務。 |
| 來賓相容性 | 在虛擬機器內執行任何作業系統 | [與主機在相同的作業系統版本](../deploy-containers/version-compatibility.md)上執行（hyper-v 隔離可讓您在輕型 VM 環境中執行舊版相同的作業系統）
| 部署     | 使用 Windows 系統管理中心或 Hyper-v 管理員來部署個別 Vm;使用 PowerShell 或 System Center Virtual Machine Manager 來部署多個虛擬機器。 | 透過命令列使用 Docker 來部署個別容器;使用 orchestrator （例如 Azure Kubernetes 服務）部署多個容器。 |
| 作業系統更新與升級 | 在每個 VM 上下載並安裝作業系統更新。 安裝新的作業系統版本需要升級，或經常只建立全新的 VM。 這可能相當耗時，特別是如果您有許多 Vm .。。 | 更新或升級容器內的作業系統檔案是相同的： <br><ol><li>編輯您的容器影像的組建檔案（稱為 Dockerfile），以指向最新版本的 Windows 基礎映射。 </li><li>使用這個新的基本影像重建您的容器影像。</li><li>將容器影像推到您的容器註冊表。</li> <li>使用 orchestrator 重新部署。<br>Orchestrator 提供強大的自動化功能，可讓您以比例進行。 如需詳細資訊，請參閱[教學課程：在 Azure Kubernetes 服務中更新應用程式](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update)。</li></ol> |
| 持久性儲存 | 針對單一 VM 使用虛擬硬碟（VHD），或針對由多個伺服器共用的儲存空間使用 SMB 檔案共用 | 針對單一節點或由多個節點或伺服器共用的儲存空間，使用 Azure 磁片進行本機儲存，或使用 Azure Files （SMB 共用）。 |
| 負載平衡 | 虛擬機器負載平衡會將執行中的 Vm 移至容錯移轉叢集中的其他伺服器。 | 容器本身沒有移動;而是可以在叢集節點上自動啟動或停止容器，以管理載入和可用性的變更。 |
| 容錯 | Vm 可以容錯移轉到群集中的另一個伺服器，並在新伺服器上重新開機 VM 的作業系統。  | 如果叢集節點發生故障，任何在它上執行的容器都會快速由另一個叢集節點上的 orchestrator 重新建立。 |
| 網路     | 使用虛擬網路介面卡。 | 使用隔離的虛擬網路介面卡視圖，提供較少的虛擬化–主機的防火牆會與容器共用，同時使用較少的資源。 如需詳細資訊，請參閱[Windows 容器網路](../container-networking/architecture.md)。 |