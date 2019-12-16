---
title: 關於 Windows 容器
description: 容器是一種跨內部部署和雲端中的不同環境封裝和執行應用程式 (包括 Windows 應用程式) 的技術。 本主題討論 Microsoft、Windows 和 Azure 如何協助您在容器中開發及部署應用程式，包括使用 Docker 和 Azure Kubernetes Service。
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 4fad299db2c897a6be860ef0cc71e80969c75357
ms.sourcegitcommit: 8dedb887b038fbff872327f51c7416454b301b86
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/06/2019
ms.locfileid: "74909408"
---
# <a name="windows-and-containers"></a>Windows 和容器

容器是一種跨內部部署和雲端中的不同環境封裝和執行 Windows 和 Linux 應用程式的技術。 容器提供了輕量型、隔離的環境，讓應用程式更容易開發、部署和管理。 容器可快速啟動和停止，使其適合需要快速適應變動需求的應用程式。 容器的輕量本質也會使其成為可增加基礎結構密度和使用率的實用工具。

![此圖形顯示容器在雲端或內部部署環境中執行的方式，並支援以幾乎任何語言撰寫的整合型應用程式或微服務。](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Microsoft 容器生態系統

Microsoft 提供一些工具和平台，協助您在容器中開發和部署應用程式：

- 使用 [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) (其會使用 Windows 內建的容器功能)，<strong>在 Windows 10 上執行 Windows 型或 Linux 型容器</strong>以便進行開發和測試。 您也可以[在 Windows Server 上以原生方式執行容器](../quick-start/set-up-environment.md?tabs=Windows-Server)。
- 使用 [Visual Studio 中的強大的容器支援](https://docs.microsoft.com/visualstudio/containers/overview)和 [Visual Studio Code](https://code.visualstudio.com/docs/azure/docker) (其包括對 Docker、Docker Compose、Kubernetes、Helm 和其他實用技術的支援)，<strong>開發、測試、發佈及部署 Windows 型容器</strong>。
- <strong>將您的應用程式當作容器映像發佈</strong>至公用 DockerHub 供其他人使用，或發佈至私人 [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) 供貴組織本身進行開發和部署，並直接從 Visual Studio 和 Visual Studio Code 進行推送和提取。
- <strong>在 Azure 上或其他雲端大規模部署容器</strong>：

  - 從容器登錄 (例如 Azure Container Registry) 提取您的應用程式 (容器映像)，然後使用協調器 (例如 [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) (Windows 型應用程式預覽版) 或 [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/)) 進行大規模部署及管理。
  - Azure Kubernetes Service 會將容器部署至 Azure 虛擬機器，並進行大規模管理 (不論是數十個、數百個或甚至數千個容器)。 Azure 虛擬機器會執行自訂的 Windows Server 映像 (如果您要部署 Windows 型應用程式) 或自訂的 Ubuntu Linux 映像 (如果您要部署 Linux 型應用程式)。
- 使用 [Azure Stack 搭配 AKS 引擎](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) (Linux 容器的預覽版) 或 [Azure Stack 搭配 OpenShift](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack)，<strong>部署內部部署容器</strong>。 您也可以在 Windows Server 上自行設定 Kubernetes (請參閱 [Windows上的 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md))，而我們也致力於[在 RedHat OpenShift 容器平台上執行 Windows 容器](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821)的支援。

## <a name="how-containers-work"></a>容器的運作方式

容器是一個隔離且輕量型的定址接收器，用於在主機作業系統上執行應用程式。 容器是以主機作業系統的核心 (可視為作業系統的「地下管線」) 為基礎，如下圖所示。

![顯示如何在核心上執行容器的架構圖](media/container-diagram.svg)

當容器共用主機作業系統的核心時，該容器不會取得其自由的存取權。 然而，容器會取得隔離的 (在某些情況下為虛擬化) 的系統檢視。 例如，容器可以存取虛擬化版本的檔案系統和登錄，但任何變更都只會影響容器並在容器停止時予以捨棄。 若要儲存資料，容器可以掛接持續性儲存體，例如 [Azure 磁碟](https://azure.microsoft.com/services/storage/disks/)或檔案共用 (包括 [Azure 檔案](https://azure.microsoft.com/services/storage/files/))。

容器是以核心為基礎，但核心並未提供應用程式執行時需要的所有 API 和服務，這些大多數是由以使用者模式在核心之上執行的系統檔案 (程式庫) 所提供。 由於容器與主機的使用者模式環境隔離，因此容器需要自己的使用者模式系統檔案複本，而這些檔案會封裝成所謂的基底映像。 基底映像可作為建立容器的基礎層，並為其提供核心所未提供的作業系統服務。 但我們稍後會詳細討論容器映像。

## <a name="containers-vs-virtual-machines"></a>容器與虛擬機器

與容器相比，虛擬機器 (VM) 會執行完整的作業系統 (包括它自己的核心)，如下圖所示。

![顯示 VM 如何在主機作業系統旁執行完整作業系統的架構圖](media/virtual-machine-diagram.svg)

容器和虛擬機器都有其用途：事實上，許多容器部署都會使用虛擬機器作為主機作業系統，而不是直接在硬體上執行，尤其是在雲端中執行容器時。

如需這些互補技術的相似性與差異詳細資訊，請參閱[容器與虛擬機器](containers-vs-vm.md)。

## <a name="container-images"></a>容器映像

所有容器都是從容器映像建立而來。 容器映像是一套組織成層級堆疊的檔案組合，其位於本機電腦或遠端容器登錄中。 容器映像包含支援您應用程式所需的使用者模式作業系統檔案、您的應用程式、您應用程式的任何執行階段或相依性，以及您的應用程式正常執行時所需的任何其他組態檔。

Microsoft 提供數個映像 (稱為基底映像)，您可將其作為建立自有容器映像的起點：

* <strong>Windows</strong> - 包含一組完整的 Windows API 和系統服務 (沒有伺服器角色)。
* <strong>Windows Server Core</strong> - 較小的映像，其包含 Windows Server API 的子集 – 也就是完整的 .NET 架構。 它也包含大部分的伺服器角色 (雖然很少)，但不包含傳真伺服器。
* <strong>Nano Server</strong> - 最小的 Windows Server 映像，支援 .NET Core API 和某些伺服器角色。
* <strong>Windows 10 IoT Core</strong> - 硬體製造商針對執行 ARM 或 x86/x64 處理器的小型物聯網 (IoT) 裝置所使用的 Windows 版本。

如先前所述，容器映像是由一系列的層級所組成。 每一層都包含一組檔案，若覆疊在一起，即代表您的容器映像。 由於容器的分層本質，您不需要一律以基底映像為目標來建立 Windows 容器。 然而，您可以將目標設為已有您想要架構的另一個映像。 例如，.NET 小組會發佈帶有 .NET 核心執行階段的 [.NET 核心映像](https://hub.docker.com/_/microsoft-dotnet-core)。 這可讓使用者不需要重複進行安裝 .NET 核心的程序，而是可以重複使用此容器映像的層級。 .NET 核心映像本身是根據 Nano Server 所建立。

如需詳細資訊，請參閱[容器基底映像](../manage-containers/container-base-images.md)。

## <a name="container-users"></a>容器使用者

### <a name="containers-for-developers"></a>開發人員的容器

容器可協助開發人員更快速地建置及提供更高品質的應用程式。 使用容器，開發人員可以建立在幾秒內部署的容器映像 (在不同的環境中都相同)。 容器的作用是跨小組共用程式碼及啟動開發環境的簡單機制，但不會影響主機檔案系統。

容器具有可攜性和廣泛性，可執行以任何語言撰寫的應用程式，而且與任何執行 Windows 10、1607 版或更新版本或 Windows Server 2016 或更新版本的電腦相容。 開發人員可以在其膝上型或桌上型電腦本機建立及測試容器，然後將相同的容器映像部署到其公司的私人雲端、公用雲端或服務提供者。 容器既有的靈活性可支援大規模、虛擬化雲端環境中的新型應用程式開發模式。

### <a name="containers-for-it-professionals"></a>IT 專業人員的容器

容器可協助系統管理員建立更容易更新和維護並可充分利用硬體資源的基礎結構。 IT 專業人員可以使用容器為其開發、品管及生產小組提供標準化的環境。 藉由使用容器，系統管理員可抽離作業系統安裝與基礎結構的差異。

## <a name="container-orchestration"></a>容器協調流程

在設定容器型環境時，協調器是基礎結構的重要部分。 雖然您可以使用 Docker 和 Windows 手動管理一些容器，但應用程式通常會使用五個、十個或甚至數百個容器 (協調器的來源)。

為了協助大規模管理生產環境中的容器，於是建立了容器協調器。 協調器可提供下列功能：

- 大規模部署
- 工作負載排程
- 健全狀況監視
- 當節點失敗時容錯移轉
- 擴大或縮小規模
- 網路功能
- 服務探索
- 協調應用程式升級
- 叢集節點親和性

有許多不同的協調器可供您用於 Windows 容器，以下是 Microsoft 提供的選項：
- [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) - 使用受控 Azure Kubernetes 服務
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) - 使用受控服務
- [Azure Stack 搭配 AKS 引擎](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) - 在內部部署環境使用 Azure Kubernetes Service
- [Windows 上的 Kubernetes](../kubernetes/getting-started-kubernetes-windows.md) - 在 Windows 上自行設定 Kubernetes

## <a name="try-containers-on-windows"></a>在 Windows 上試用容器

若要開始在 Windows Server 或 Windows 10 上使用容器，請參閱下列各項：
> [!div class="nextstepaction"]
> [入門：針對容器設定您的環境](../quick-start/set-up-environment.md)

若要協助判斷您的案例適合那些 Azure 服務，請參閱 [Azure 容器服務](https://azure.microsoft.com/product-categories/containers/)和 [選擇要用於裝載應用程式的 Azure 服務](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree)。
