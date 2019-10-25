---
title: Windows 容器常見問題集
description: Windows Server 容器常見問題
keywords: Docker, 容器
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 19ff54ec032d61b24aea9fec4f14e8fce301d33a
ms.sourcegitcommit: 347d7c9d34f4c1d2473eb6c94c8ad6187318a037
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257950"
---
# <a name="frequently-asked-questions-about-containers"></a>有關容器的常見問題

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Linux 與 Windows Server 容器有何不同？

Linux 和 Windows Server 都在其內核和核心作業系統中實現類似的技術。 差異來自於平台以及在容器內執行的工作負載。  

當客戶使用 Windows Server 容器時，它們可以與現有的 Windows 技術（例如 .NET、ASP.NET 和 PowerShell）整合。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>在 Windows 上執行容器的先決條件為何？

已在 Windows Server 2016 的平臺中引入容器。 若要使用容器，您需要 Windows Server 2016 或 Windows 10 周年紀念更新（版本1607）或更新版本。 若要深入瞭解，請閱讀[系統需求](../deploy-containers/system-requirements.md)。

## <a name="what-are-wcow-and-lcow"></a>什麼是 WCOW 和 LCOW？

WCOW 簡稱為「Windows 上的 Windows 容器」。 LCOW 簡稱為「Windows 上的 Linux 容器」。

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>如何授權容器？ 我可以執行的容器數量是否有限制？

Windows 容器影像[EULA](../images-eula.md)描述的使用方式取決於擁有有效授權主機作業系統的使用者。 允許使用者執行的容器數目，取決於主機作業系統版本和與容器一起執行的[隔離模式](../manage-containers/hyperv-container.md)，以及這些容器是否針對開發人員/測試用途或在生產環境中執行。

|主機作業系統                                                         |進程隔離的容器限制                   |Hyper-v-隔離的容器限制                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |無限制                                          |pplx-2                                                  |
|Windows Server Datacenter                                       |無限制                                          |無限制                                          |
|Windows 10 專業版與企業版                                   |無限制 *（僅限測試或開發目的）*|無限制 *（僅限測試或開發目的）*|
|Windows 10 IoT 核心版與企業版）                             |無限制 *（僅限測試或開發目的）*|無限制 *（僅限測試或開發目的）*|

Windows Server 容器影像的使用方式是透過讀取該[版本](/windows-server/get-started-19/editions-comparison-19.md)支援的虛擬化來賓數量來決定。 在 Windows 版 IoT 版本中，容器的產品使用量視其他授許可權制而定。 請閱讀[容器影像的 EULA](../images-eula.md) ，以瞭解所允許的專案與內容。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>作為開發人員，我是否必須針對每個容器類型重新編寫應用程式？

否。 Windows 容器影像在 Windows 伺服器容器與 Hyper-v 隔離中都是常見的。 容器類型的選擇會在您啟動容器時決定。 從開發人員的觀點來看，Windows Server 容器與 Hyper-v 隔離是兩種相同的風格。 它們提供相同的開發、程式設計和管理經驗，而且都是開放且可擴展的，且包含與 Docker 的相同層級整合和支援。

開發人員可以使用 Windows Server 容器建立容器影像，並在 Hyper-v 隔離中進行部署，或者在指定適當的執行時間標誌前，不做任何變更。

Windows Server 容器提供更大的密度和效能，以提高速度，例如較低的旋轉時間，以及與嵌套設定相比，更快的執行時間效能。 Hyper-v 隔離（針對其名稱）提供更大的隔離能力，可確保在某個容器中執行的程式碼無法損除或影響在相同主機上執行的主機作業系統或其他容器。 這對託管不受信任的程式碼有需求的多租戶案例非常有用，包括 SaaS 應用程式和計算主機。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>我可以在 Windows 10 企業版或專業版上，在進程隔離模式中執行 Windows 容器嗎？

從 Windows 10 年 10 2018 月的更新開始，您可以使用程式隔離來執行 Windows 容器，但是您必須先使用執行容器時的`--isolation=process`標誌來直接要求程式隔離`docker run`。

如果您想要以這種方式執行 Windows 容器，您必須確認您的主機執行的是 Windows 10 組建 17763 +，而且您有一個具有引擎18.09 或更新版本的 Docker 版本。

> [!WARNING]
> 此功能僅適用于開發和測試。 您應該繼續使用 Windows Server 作為生產部署的主機。 使用此功能時，您也必須確定主機和容器版本戳記相符，否則容器可能無法啟動或顯示未定義的行為。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>如何讓我的容器影像在 air-有空隙的電腦上可用？

Windows 容器基底映射包含由授許可權制其傳播的專案。 當您在這些影像上建立並將它們推入私人或公用的登錄時，您會注意到基底圖層從未推入。 相反地，我們使用外語的概念，指向位於 Azure 雲端儲存區中的實際基底圖層。

當您有一個有氣流的電腦，而該電腦只能從私人容器登錄位址拉入影像時，這會讓事情複雜化。 在這種情況下，嘗試追蹤外國層來取得基本影像無法運作。 若要覆寫外部層行為，您可以在`--allow-nondistributable-artifacts` Docker 守護程式中使用標誌。

> [!IMPORTANT]
> 此標誌的使用方式不應會妨礙您符合 Windows 容器基本影像授權條款的義務;您不得將 Windows 內容張貼至公用或協力廠商再發行。 允許在您自己的環境中使用。

## <a name="additional-feedback"></a>其他意見反應

想要在常見問題中新增內容嗎？ 在 [批註] 區段中開啟新的意見反應問題，或使用 GitHub 設定此頁面的 [拉取要求]。
