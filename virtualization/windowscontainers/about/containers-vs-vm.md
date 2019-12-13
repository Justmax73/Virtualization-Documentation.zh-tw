---
title: 容器與虛擬機器的比較
description: 本主題討論容器和虛擬機器之間的一些主要相似性和差異，以及您可能想要使用的部分。 容器和虛擬機器都有其用途，事實上，許多容器部署都會使用虛擬機器做為主機作業系統，而不是直接在硬體上執行，特別是在雲端中執行容器時。
keywords: docker，容器，vm，虛擬機器
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910818"
---
# <a name="containers-vs-virtual-machines"></a>容器與虛擬機器的比較

本主題討論容器和虛擬機器（Vm）之間的一些主要相似點和差異，以及您可能想要使用的部分。 容器和 Vm 各有其用途，事實上，許多容器部署都會使用 Vm 做為主機作業系統，而不是直接在硬體上執行，特別是在雲端中執行容器時。

如需容器的總覽，請參閱[Windows 和容器](index.md)。

## <a name="container-architecture"></a>容器架構

容器是一個隔離且輕量的定址接收器，用於在主機作業系統上執行應用程式。 容器是以主機作業系統核心為基礎（可視為作業系統的內嵌管道），而且只包含應用程式，以及在使用者模式下執行的一些輕量作業系統 Api 和服務，如下圖所示。

![顯示容器在核心上執行方式的架構圖](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>虛擬機器架構

與容器相較之下，Vm 會執行完整的作業系統（包括它自己的核心），如下圖所示。

![顯示 Vm 如何在主機作業系統旁執行完整作業系統的架構圖](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>容器與虛擬機器的比較

下表顯示這些互補技術的一些相似性和差異。

|                 | 虛擬機器  | 容器  |
| --------------  | ---------------- | ---------- |
| 隔離性       | 提供與主機作業系統和其他 Vm 的完整隔離。 當強式安全性界限很重要時（例如，將應用程式從相同伺服器或叢集上的競爭公司託管），這會很有用。 | 通常會從主機和其他容器提供輕量隔離，但不會提供做為 VM 的強式安全性界限。 （您可以使用[hyper-v 隔離模式](../manage-containers/hyperv-container.md)來隔離輕量 VM 中的每個容器），以提高安全性。 |
| 作業系統 | 會執行包含核心的完整作業系統，因此需要更多系統資源（CPU、記憶體和儲存體）。 | 執行作業系統的使用者模式部分，而且可以量身打造，只包含應用程式所需的服務，並使用較少的系統資源。 |
| 來賓相容性 | 僅執行虛擬機器內的任何作業系統 | 會在與[主機相同的作業系統版本](../deploy-containers/version-compatibility.md)上執行（hyper-v 隔離可讓您在輕量 VM 環境中執行舊版的相同 OS）
| 部署     | 使用 Windows Admin Center 或 Hyper-v 管理員部署個別 Vm;使用 PowerShell 或 System Center Virtual Machine Manager 部署多個 Vm。 | 透過命令列使用 Docker 部署個別容器;使用協調器（例如 Azure Kubernetes Service）來部署多個容器。 |
| 作業系統更新和升級 | 下載並安裝每個 VM 上的作業系統更新。 安裝新的作業系統版本需要升級，或通常只是建立全新的 VM。 這可能非常耗時，特別是如果您有很多 Vm 。 | 更新或升級容器內的作業系統檔案是相同的： <br><ol><li>編輯容器映射的組建檔案（也稱為 Dockerfile），以指向最新版本的 Windows 基底映射。 </li><li>使用這個新的基底映射重建您的容器映射。</li><li>將容器映射推送至容器登錄。</li> <li>使用 orchestrator 重新部署。<br>Orchestrator 提供強大的自動化功能來大規模執行此作業。 如需詳細資訊，請參閱[教學課程：更新 Azure Kubernetes Service 中的應用程式](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update)。</li></ol> |
| 持續性儲存體 | 將虛擬硬碟（VHD）用於單一 VM 的本機儲存體，或針對多部伺服器共用的存放裝置使用 SMB 檔案共用 | 將 Azure 磁片用於單一節點的本機儲存體，或針對由多個節點或伺服器共用的儲存體，Azure 檔案儲存體（SMB 共用）。 |
| 負載平衡 | 虛擬機器負載平衡會將執行中的 Vm 移至容錯移轉叢集中的其他伺服器。 | 容器本身不會移動;相反地，協調器可以自動啟動或停止叢集節點上的容器，以管理負載和可用性的變更。 |
| 容錯 | Vm 可以故障切換至叢集中的另一部伺服器，並在新的伺服器上重新開機 VM 的作業系統。  | 如果叢集節點失敗，則任何在其上執行的容器都會由另一個叢集節點上的 orchestrator 快速重新建立。 |
| 網路     | 使用虛擬網路介面卡。 | 使用虛擬網路介面卡的隔離視圖，提供較少的虛擬化-主機的防火牆會與容器共用，同時使用較少的資源。 如需詳細資訊，請參閱[Windows 容器網路](../container-networking/architecture.md)功能。 |