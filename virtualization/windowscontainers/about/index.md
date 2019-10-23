---
title: 關於 Windows 容器
description: 容器是一種封裝及執行 app 的技術，包括 Windows app--在內部部署和雲端中的多個環境中。 本主題討論 Microsoft、Windows 和 Azure 如何協助您在容器中開發和部署應用程式，包括使用 Docker 和 Azure Kubernetes Service。
keywords: Docker, 容器
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: acce214cc8991f20c979b6dbe636590416841cb9
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254288"
---
# <a name="windows-and-containers"></a>視窗和容器

樹枝是一種在內部部署和雲端的不同環境中封裝及執行 Windows 和 Linux 應用程式的技術。 容器提供一個羽量級、隔離的環境，讓應用程式更容易開發、部署及管理。 容器會快速開始和停止，讓它們適合需要快速適應變更需求的 app。 容器的羽量級性質也會使其成為可增加基礎結構密度與利用率的公用程式。

![圖形顯示容器在雲端或內部部署中的執行方式，支援以幾乎任何語言撰寫的單一 app 或微。](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Microsoft 容器生態系統

Microsoft 提供幾個工具和平臺，協助您在容器中開發和部署應用程式：

- <strong>在 windows 10 上執行 windows 10 上的 windows 或 Linux 版容器</strong>，以使用[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)進行開發與測試，從而使用 Windows 內建的容器功能。 您也可以[在 Windows Server 上本機執行容器](../quick-start/set-up-environment.md?tabs=Windows-Server)。
- 在 Visual Studio 與[Visual studio 程式碼](https://code.visualstudio.com/docs/azure/docker)[中使用功能強大的容器支援](https://docs.microsoft.com/visualstudio/containers/overview)來<strong>開發、測試、發佈及部署 Windows 容器</strong>，其中包含 docker、docker 撰寫、Kubernetes、Helm 及其他有用的支援科技.
- 將<strong>您的應用程式發佈</strong>到公用 DockerHub 供其他人使用，或發佈到您組織自己的開發與部署的私人[Azure 容器](https://azure.microsoft.com/services/container-registry/)登錄，直接從 Visual studio 和 visual studio 程式碼中推入並拉入.
- 在 Azure 或其他雲的<strong>縮放比例部署容器</strong>：

  - 從容器登錄（例如 Azure 容器登錄）提取您的應用程式（容器影像），然後使用 orchestrator （例如[Azure Kubernetes Service （AKS）](https://docs.microsoft.com/azure/aks/intro-kubernetes) （在 Windows 版應用程式的預覽）或 Azure 服務以縮放比例進行部署和管理。 [結構](https://docs.microsoft.com/azure/service-fabric/)。
  - Azure Kubernetes Service 會將容器部署到 Azure 虛擬機器，並依規模管理（無論是數十個容器、上百甚至上千個）。 Azure 虛擬機器會執行自訂的 Windows Server 映射（如果您要部署 Windows 應用程式），或是自訂的 Ubuntu Linux 影像（如果您要部署 Linux 應用程式）。
- <strong>在內部部署</strong>中使用[AZURE 堆疊與 AKS 引擎](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)（在使用 Linux 容器進行預覽）或 OpenShift 中的[azure 堆疊](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack)，來部署容器。 您也可以在 Windows Server 上自行設定 Kubernetes （請參閱 Windows 上的[Kubernetes](../kubernetes/getting-started-kubernetes-windows.md)），而且我們正在努力支援[在 RedHat 上執行 Windows 容器 OpenShift 容器平臺](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821)。

## <a name="how-containers-work"></a>容器的運作方式

容器是獨立的輕型思洛儲存體，可在主機作業系統上執行應用程式。 容器是在主機作業系統的內核（可以看作是作業系統的掩蔽管道）上建立，如圖所示。

![顯示樹枝如何在內核頂端執行的結構化圖表](media/container-diagram.svg)

當容器與主機作業系統的內核共用時，容器不會取得它的 unfettered 存取權。 相反地，容器會在某些情況下取得隔離的，在某些情況下，系統會進行虛擬化的查看。 例如，容器可以存取檔案系統和註冊表的虛擬化版本，但任何變更只會影響容器，在停止時就會遭到捨棄。 若要儲存資料，容器可以裝入永久性儲存空間，例如[Azure 磁片](https://azure.microsoft.com/services/storage/disks/)或檔案共用（包括[Azure](https://azure.microsoft.com/services/storage/files/)檔案）。

容器是在內核上方建立，但內核不會提供 app 需要執行的所有 Api 與服務，這些都是由在使用者模式下執行的系統檔案（文件庫）所提供。 因為容器是與主機的使用者模式環境隔離，所以容器需要自己的這些使用者模式系統檔案複本，這些檔案封裝在稱為基本映射的專案中。 基底影像是您建立容器時所使用的基本層級，提供的是內核不提供的作業系統服務。 但我們稍後會深入討論容器影像。

## <a name="containers-vs-virtual-machines"></a>容器與虛擬電腦

相對於容器，虛擬機器（Vm）會執行完整的作業系統，包括它本身的內核，如此圖表所示。

![顯示 Vm 在主機作業系統旁執行完整作業系統的體系結構圖表](media/virtual-machine-diagram.svg)

容器和虛擬機器都有它們的用途，事實上，許多容器部署都會將虛擬機器當作主機作業系統使用，而不是直接在硬體上執行，尤其是在雲端執行容器時。

如需這些互補技術的相似性與差異的詳細資訊，請參閱[容器與虛擬機器](containers-vs-vm.md)。

## <a name="container-images"></a>容器影像

所有容器都是從容器影像建立。 容器影像是一種結構套件，可整理成位於本機電腦或遠端容器登錄的圖層堆疊中。 容器影像由支援您 app 的使用者模式作業系統檔案、您的應用程式、應用程式的任何執行時間或相依性，以及您 app 需要正常執行的任何其他其他設定檔所組成。

Microsoft 提供數個影像（稱為基本影像），您可以用來做為建立您自己的容器影像的起點：

* <strong>Windows</strong> -包含完整的 windows api 與系統服務集（減去伺服器角色）。
* <strong>Windows Server Core</strong> -一個較小的影像，其中包含 Windows Server api 的子集，即完整的 .net 架構。 它也包含大部分的伺服器角色，不過很少，不是傳真伺服器。
* <strong>Nano Server</strong> -最小的 Windows Server 映射，支援 .Net Core api 及部分伺服器角色。
* <strong>Windows 10 IoT 核心</strong>版-適用于執行 ARM 或 x86/x64 處理器之小型網際網路之硬體製造商使用的 windows 版本。

如前文所述，容器影像是由一系列的圖層組成。 每個圖層都包含一組檔案，當疊加在一起時會代表您的容器影像。 由於容器的層次性質，您不一定要以基本映射為目標來建立 Windows 容器。 相反地，您可以將已攜帶您想要的架構的另一個影像作為目標。 例如，.NET 小組發佈一個攜帶 .NET core 執行時間的[.net 核心影像](https://hub.docker.com/_/microsoft-dotnet-core)。 它會讓使用者不需要重複安裝 .NET core 的程式，而是必須重複使用這個容器影像的圖層。 .NET core image 本身是根據 Nano Server 建立的。

如需詳細資訊，請參閱[容器基底影像](../manage-containers/container-base-images.md)。

## <a name="container-users"></a>容器使用者

### <a name="containers-for-developers"></a>開發人員容器

容器可協助開發人員以更快的速度建立及傳送較高品質的 app。 使用容器，開發人員可以建立可在數秒內（與環境相同）部署的容器影像。 容器是一種簡單的機制，可跨團隊共用程式碼，並可引導開發環境而不會影響您的主機 filesystem。

容器是可移植且通用的功能，可執行以任何語言撰寫的應用程式，且與任何執行 Windows 10 版本1607或更新版本的電腦相容，或在 Windows 2016 或更新版本上相容。 開發人員可以在膝上型電腦或桌上型電腦上建立並測試容器，然後將這個相同的容器影像部署到其公司的私人雲端、公有雲端或服務提供者。 容器的自然靈活性支援大型、虛擬化雲端環境中的新式 app 開發模式。

### <a name="containers-for-it-professionals"></a>IT 專業人員的容器

容器可協助系統管理員建立更容易更新及維護的基礎結構，以及更充分地利用硬體資源。 IT 專業人員可以使用容器為開發、QA 及生產團隊提供標準化的環境。 透過使用容器，系統管理員會將作業系統安裝與底層基礎結構的差異取消摘要。

## <a name="container-orchestration"></a>容器 orchestration

Orchestrators 是設定容器式環境時的一個重要基礎結構。 雖然您可以使用 Docker 和 Windows 手動管理幾個容器，但 app 通常會使用五個、十個甚至上百個容器，就是 orchestrators 的功能。

容器 orchestrators 的建立可協助您在規模及生產環境中管理容器。 Orchestrators 提供下列功能：

- 依比例部署
- 工作負載排程
- 健康情況監視
- 當節點失敗時的容錯移轉
- 向上或向下縮放
- 網路
- 服務探索
- 協調 app 升級
- 叢集節點關聯

有許多不同的 orchestrators 可與 Windows 容器搭配使用;以下是 Microsoft 提供的選項：
- [Azure Kubernetes Service （AKS）](https://docs.microsoft.com/azure/aks/intro-kubernetes) -使用受管理的 Azure Kubernetes 服務
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) -使用受管理的服務
- [具有 AKS 引擎的 Azure 堆疊](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)-使用 Azure Kubernetes 服務內部部署
- [Windows 上的 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md) -在 windows 上設定 Kubernetes

## <a name="try-containers-on-windows"></a>在 Windows 上試用容器

若要開始使用 Windows Server 或 Windows 10 上的容器，請參閱下列內容：
> [!div class="nextstepaction"]
> [快速入門：設定容器的環境](../quick-start/set-up-environment.md)

如需協助您判斷哪些 Azure 服務適合您的案例，請參閱[Azure 容器服務](https://azure.microsoft.com/product-categories/containers/)，並[選擇要用來託管您的應用程式的 azure 服務](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree)。
