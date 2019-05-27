---
title: 關於 Windows 容器
description: 深入了解 Windows 容器。
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 80514884b4c95657f63cf585ece6aa8c8b23cc44
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674723"
---
# <a name="about-windows-containers"></a>關於 Windows 容器

假設有一間廚房。 這個單一會議室內部是您需要 cook 餐的所有項目： 烤箱、 移動瀏覽、 接收，以及等等。 這是我們的容器。

![使用黑色方塊內黃色桌布完全 furnished 廚房的圖例。](media/box1.png)

現在請想像放置這個廚房建築內，做為一本書滑動到出版品介紹輕鬆。 因為所有項目廚房需要函式已經有，我們需要開始烹飪了連線電力及的配置。

![由兩個堆疊的黑色方塊的 apartment 建置。 這些方塊的四個為廚房範例中使用的相同黃色方塊，並且的隨機地方建置，而其餘部分有任一多色的生活房間或空白，並且反灰。](media/apartment.png)

為什麼要停止那里嗎？ 您可以自訂您的建置任何您喜歡; 的方式使用許多種類的房間填滿、 填滿有相同的房間，或有兩個混合。

此房間像的容器是透過執行應用程式的方式，我們會在我們廚房 cook 作用。 容器會讓應用程式，且所有項目該應用程式需要執行它自己的隔離方塊。 如此一來，隔離的應用程式並不知道應用程式或其容器外存在的處理程序。 因為容器有所有項目來執行應用程式需求，容器可以移動任何地方，其主機佈建新的使用僅資源，而不會觸及其他容器的佈建的任何資源。

下列影片會告訴您更多關於 Windows 容器可以做什麼，以及如何與 Docker 的 Microsoft 的合作關係可協助建立無障礙的開放原始碼容器開發環境：

<iframe width="800" height="450" src="https://www.youtube.com/embed/Ryx3o0rD5lY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## <a name="container-fundamentals"></a>容器的基礎

讓我們了解一些您會發現有用當您開始搭配使用 Windows 容器的條款：

- 容器主機： 設定 Windows 容器功能的實體或虛擬電腦系統。 容器主機將會執行一或多個 Windows 容器。
- 沙箱： 會擷取所有變更的層您對容器執行 （例如檔案系統修改、 登錄修改或軟體安裝） 時。
- 基本映像： 第一層中提供的容器作業系統環境的容器映像層。 無法修改基底映像。
- 容器映像： 唯讀範本的指示來建立容器。 影像可以根據基本、 未變更的作業系統環境中，但也可以建立從修改過的容器的沙箱。 這些修改映像層上方的基本映像層，其變更，且可以複製這些層級，並重新套用到其他的基本映像來建立新的映像使用那些相同的變更。
- 容器存放庫： 建立新的映像儲存在容器映像和其相依性即每次本機存放庫。 您可以重複使用儲存的影像，為您想要在容器主機上的次數。 您也可以儲存在公用或私人登錄中，例如 Docker Hub 容器映像，使它們可在多種不同容器主機使用。
- 容器協調器： 處理程序會自動執行和管理大量的容器，以及它們如何與彼此互動。 若要深入了解，請參閱[關於 Windows 容器協調器](overview-container-orchestrators.md)。
- Docker： 自動化程序的套件，並傳遞容器映像。 若要深入了解，請參閱[Docker 概觀](docker-overview.md)， [Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)或瀏覽[Docker 網站](https://www.docker.com)。

![示範如何建立容器的流程圖。 應用程式和基本映像來建立沙箱和新的應用程式映像之上建置新的容器基本映像層。](media/containerfund.png)

熟悉虛擬機器有人可能會想要容器和虛擬機器看起來很類似。 容器執行作業系統、 具有檔案系統，也可以透過網路，就像實體或虛擬電腦系統一樣存取。 即便如此，容器的基本技術和概念還是與虛擬機器非常不同。 若要深入了解這些概念，閱讀 Mark Russinovich[部落格文章](https://azure.microsoft.com/blog/containers-docker-windows-and-trends/)，說明的差異的詳細資訊。

### <a name="windows-container-types"></a>Windows 容器類型

您應該知道的另一件事是，有兩種不同的容器類型，也稱為執行階段。

Windows Server 容器可提供透過處理程序和命名空間隔離技術，也就是為什麼這些容器也稱為處理序隔離容器的應用程式隔離。 Windows Server 容器可與容器主機和所有執行於主機上的容器共用核心。 這些處理序隔離容器沒有提供惡意防護界限，不應該用來隔離未受信任的程式碼。 由於共用核心空間，因此這些容器需要相同的核心版本和組態。

HYPER-V 隔離擴充 Windows Server 容器高度最佳化的虛擬機器中執行每個容器所提供的隔離能力。 此設定中，容器主機不會與相同主機上的其他容器共用其核心。 這些容器的設計是要以虛擬機器的相同安全保證來管控惡意多組織用戶共享。 因為這些容器不會與主機或主機上的其他容器共用核心，所以它們可以執行核心具有不同版本和設定 （在支援的版本）。 例如，Windows 10 上的所有 Windows 容器會都使用 HYPER-V 隔離來運用 Windows Server 核心版本和設定。

使用或不使用 HYPER-V 隔離在 Windows 上執行容器是執行階段決策。 您可以一開始建立使用 HYPER-V 隔離的容器，然後稍後在執行階段選擇改為執行 Windows Server 容器。

## <a name="container-users"></a>容器的使用者

### <a name="containers-for-developers"></a>適用於開發人員的容器

容器可協助建置及提供更高品質的應用程式的速度更快的開發人員。 開發人員可以建立以秒為單位在所有的環境完全部署的 Docker 映像。 沒有起大規模且持續增長的生態系統的應用程式封裝在 Docker 容器中。 是 DockerHub，由 Docker，維護的公用容器化應用程式登錄已發行了超過 180000 個應用程式在其公用社群儲存機制，以及該號碼仍有成長。

當開發人員 containerizes 應用程式時，在應用程式，以及它需要執行的元件會結合到映像。 當您需要時，容器可從這個映像建立。 您也可以將映像當作建立另一個映像的基準，使映像建立更為快速。 多個容器都可以共用相同的映像，這表示容器非常快速地啟動，並且使用較少的資源。 例如，開發人員可以使用容器來加速輕量型和可攜式應用程式元件，也稱為微服務，如分散式應用程式，以及個別地快速調整每個服務。

容器是可攜式及多用途，即可在任何語言中，撰寫，而它們可與任何執行 Windows Server 2016 的電腦相容。 開發人員可以建立及測試容器，以在本機上他們的膝上型電腦或桌面，然後將相同的容器映像部署到其公司的私人雲端、 公用雲端或服務提供者。 容器既有的靈活性支援大規模、 虛擬化雲端環境中的新型應用程式開發模式。

### <a name="containers-for-it-professionals"></a>適用於 IT 專業人員的容器

容器可協助系統管理員建立更容易更新和維護的基礎結構。 IT 專業人員可以使用容器為其開發、 品管及生產小組提供標準化的環境。 他們再也不必擔心複雜的安裝和設定程序。 藉由使用容器，系統管理員將可抽離 OS 安裝和基礎結構的差異。

## <a name="containers-101-video-presentation"></a>容器 101 視訊簡報

下列影片呈現將讓您更深入的歷程記錄的概觀與 Windows 容器的實作。

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## <a name="try-windows-server-containers"></a>試用 Windows Server 容器

準備好開始運用容器的強大威力了嗎？ 下列文章將協助您開始使用：

若要設定 Windows Server 上的容器，請參閱[Windows Server 快速入門](../quick-start/quick-start-windows-server.md)。

若要設定 Windows 10 上的容器，請參閱[Windows 10 快速入門](../quick-start/quick-start-windows-10.md)。