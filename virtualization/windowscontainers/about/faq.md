---
title: Windows 容器常見問題集
description: Windows Server 容器常見問題
keywords: docker, 容器
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910798"
---
# <a name="frequently-asked-questions-about-containers"></a>關於容器的常見問題

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux 與 Windows Server 容器有何差異？

Linux 和 Windows Server 都在其核心和核心作業系統內執行類似的技術。 差異來自於平台以及在容器內執行的工作負載。  

當客戶使用 Windows Server 容器時，可以整合現有的 Windows 技術，例如 .NET、ASP.NET 和 PowerShell。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>在 Windows 上執行容器的必要條件為何？

容器已使用 Windows Server 2016 引進平臺。 若要使用容器，您需要 Windows Server 2016 或 Windows 10 年度更新版（版本1607）或更新版本。 若要深入瞭解，請參閱[系統需求](../deploy-containers/system-requirements.md)。

## <a name="what-are-wcow-and-lcow"></a>什麼是 WCOW 和 LCOW？

「Windows 上的 Windows 容器」的 WCOW 是簡短的。 「Windows 上的 Linux 容器」的 LCOW 是簡短的。

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>如何授權容器？ 我可以執行的容器數目是否有限制？

Windows 容器映射[EULA](../images-eula.md)描述的使用方式，取決於擁有有效授權主機 OS 的使用者。 使用者可以執行的容器數目取決於主機 OS 版本和執行容器的[隔離模式](../manage-containers/hyperv-container.md)，以及這些容器是否針對開發/測試目的或在生產環境中執行。

|主機作業系統                                                         |進程隔離的容器限制                   |Hyper-v-隔離的容器限制                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |無限制                                          |2                                                  |
|Windows Server Datacenter                                       |無限制                                          |無限制                                          |
|Windows 10 專業版和企業版                                   |無限制 *（僅適用于測試或開發用途）*|無限制 *（僅適用于測試或開發用途）*|
|Windows 10 IoT 核心版和企業                             |無窮大                                         |無窮大                                          |

Windows Server 容器映射使用量是藉由讀取該[版本](/windows-server/get-started-19/editions-comparison-19.md)所支援的虛擬化來賓數目來決定。 <br/>

>[!NOTE]
>在 IoT 版本的 Windows 上，\*容器的生產環境使用取決於您是否已同意 Windows 10 核心執行時間映射的 Microsoft 商業使用規定，或 Windows 10 IoT 企業版裝置授權（「Windows IoT 商業合約」）。 Windows IoT 商業協定中的其他條款和限制適用于在生產環境中使用容器映射。 請閱讀[容器映射 EULA](../images-eula.md) ，以瞭解允許的確切功能和不正確的功能。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>身為開發人員，我是否必須為每一種容器類型重寫應用程式？

不。 Windows 容器映射在 Windows Server 容器和 Hyper-v 隔離之間是共通的。 容器類型的選擇會在您啟動容器時決定。 從開發人員的觀點來看，Windows Server 容器和 Hyper-v 隔離是兩種相同的東西。 它們提供相同的開發、程式設計和管理體驗，並開放且可擴充，並包含與 Docker 相同的整合和支援層級。

開發人員可以使用 Windows Server 容器來建立容器映射，並將它部署在 Hyper-v 隔離中，或反之亦然，而不需要指定適當的執行時間旗標來進行任何變更。

當速度為關鍵時，Windows Server 容器可提供更高的密度和效能，例如較低的微調時間，以及比嵌套設定更快的執行時間效能。 Hyper-v 隔離，其名稱為 true，可提供較大的隔離，確保在一個容器中執行的程式碼無法危害或影響主機作業系統或在相同主機上執行的其他容器。 這適用于具有裝載不受信任程式碼之需求的多租使用者案例，包括 SaaS 應用程式和計算裝載。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>我可以在 Windows 10 上的進程隔離模式中執行 Windows 容器嗎？

從 Windows 10 2018 年10月更新開始，您可以執行具有進程隔離的 Windows 容器，但必須先使用 `--isolation=process` 旗標，在搭配 `docker run`執行容器時，直接要求進程隔離。 進程-隔離在 Windows 10 專業版、Windows 10 企業版、Windows 10 IoT 核心版和 Windows 10 IoT 企業版上都相容。

如果您想要以這種方式執行 Windows 容器，您必須確定您的主機正在執行 Windows 10 build 17763 +，而且您擁有具有引擎18.09 或更新版本的 Docker 版本。

> [!WARNING]
> 除了 IoT 核心版和 IoT 企業主機（接受其他條款和限制之後）之外，這項功能僅適用于開發和測試。 您應該繼續使用 Windows Server 作為生產環境部署的主機。 藉由使用這項功能，您也必須確定您的主機和容器版本標籤相符，否則容器可能無法啟動或展示未定義的行為。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>如何? 讓我的容器映射可在空中電腦上使用嗎？

Windows 容器基底映射包含其散發受到授許可權制的構件。 當您建立這些映射並將其推送至私用或公用登錄時，您會注意到基底層永遠不會推送。 相反地，我們使用外部層的概念，指向位於 Azure 雲端儲存體中的實際基底層。

當您有一部空中電腦，且只能從私人容器登錄的位址提取映射時，這可能會讓事情複雜化。 在此情況下，嘗試遵循外部層以取得基底映射將無法使用。 若要覆寫外部層行為，您可以使用 Docker daemon 中的 `--allow-nondistributable-artifacts` 旗標。

> [!IMPORTANT]
> 使用此旗標不會讓您的義務符合 Windows 容器基底映射授權的條款;您不得張貼 Windows 內容以進行公用或協力廠商的重新發佈。 允許在您自己的環境中使用。

## <a name="additional-feedback"></a>其他意見反應

想要在常見問題中加入一些專案嗎？ 在 [批註] 區段中開啟新的意見反應問題，或使用 GitHub 設定此頁面的提取要求。
