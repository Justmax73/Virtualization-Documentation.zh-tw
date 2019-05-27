---
title: Windows 容器常見問題集
description: Windows Server 容器常見問題集
keywords: Docker, 容器
author: PatrickLang
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 90894278885fde54feab222bb2bf44ca3eba331b
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674733"
---
# <a name="frequently-asked-questions-about-containers"></a>關於容器常見問題集

## <a name="what-are-wcow-and-lcow"></a>WCOW 和 LCOW 為何？

WCOW 是簡稱為 「 Windows Windows 上的容器。 」 LCOW 是縮寫 」 上的 Linux 容器 Windows。 」

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>什麼是 Linux 和 Windows Server 容器之間的差異？

Linux 和 Windows Server 實作其核心和核心作業系統中的類似技術。 差異來自於平台以及在容器內執行的工作負載。  

當客戶使用 Windows Server 容器時，他們可以整合現有的 windows 技術，例如.NET、 ASP.NET，以及 PowerShell。

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>我是否必須身為開發人員，我的應用程式為每一種容器重新撰寫？

否。 Windows 容器映像在 Windows Server 容器與 HYPER-V 隔離之間是通用的。 容器類型的選擇會在您啟動容器時決定。 將開發人員的觀點來看、 Windows Server 容器和 HYPER-V 隔離是動作的風格上的相同。 它們提供相同的開發、 程式設計，與管理體驗，並開啟並可延伸，包含相同層級的整合性和與 Docker 的支援。

開發人員可以建立容器映像使用 Windows Server 容器，並將它部署在 HYPER-V 隔離 （反之亦然） 不需要指定適當的執行階段旗標以外的任何變更。

Windows Server 容器可提供較高的密度和效能時速度是關鍵，例如較低的微調向上時間和更快速的執行階段效能相較於巢狀的設定。 HYPER-V 隔離，true 其名稱、 提供更高的隔離性，確保一個容器中執行的程式碼不會危害或影響主機作業系統或在相同主機上執行的其他容器。 這是適用於多租用戶案例裝載未受信任的程式碼，包含 SaaS 應用程式和計算裝載的需求。

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>在 Windows 上執行容器的必要條件為何？

容器是平台使用 Windows Server 2016 中引進。 若要使用的容器，您將需要 Windows Server 2016 或 Windows 10 年度更新版 （版本 1607年） 或更新版本。

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>我能夠 Windows 容器中執行獨立處理序模式 Windows 10 企業版或專業版？

從 Windows 10 年 10 月 2018 年更新，您可以使用處理序隔離執行 Windows 容器，但您必須先直接要求處理序隔離，藉由使用`--isolation=process`旗標時執行您的容器與`docker run`。

如果您想要執行您的 Windows 容器，如此一來，您將需要請確定您的主機執行 Windows 10 組建 17763 +，而且您有使用引擎 18.09 的 Docker 版本或更新版本。

> [!WARNING]
> 這項功能僅適用於開發/測試。 您應該繼續使用 Windows Server 與主機，用於生產環境部署。 藉由使用這項功能，您也必須確定您的主機和容器版本標記相符，，否則容器可能會失敗開始或會呈現未定義的行為。

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>我如何讓我的容器映像可用在空中其電腦上呢？

Windows 容器基本映像包含其散發授權受限制的成品。 當您在這些映像上建置並推送它們的私人或公用登錄到時，您會注意到的基底層永遠不會推入。 相反地，我們會使用指向位於 Azure 雲端儲存空間中的實際基底層外部層的概念。

這可以讓更加複雜件事，當您有只可以提取映像從您的私人容器登錄位址在空中其電腦。 在此情況下，嘗試依照以取得基礎映像的外部層將無法運作。 若要覆寫外部層行為，您可以使用`--allow-nondistributable-artifacts`Docker 精靈中的旗標。

> [!IMPORTANT]
> 這個旗標的使用方式不應該以防止遵守 Windows 容器基本映像授權; 規定您義務您必須不張貼公用或第三方重新發佈的 Windows 內容。 允許您自己的環境中的使用方式。

## <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft 是否參與開放容器計劃 (Open Container Initiative, OCI)?

為了確保封裝格式通用性，Docker 最近發起開放容器計劃 (OCI)，其目標是為了確保容器封裝會保持開啟且基金會格式，與 Microsoft 做為其是發起成員之一。

## <a name="additional-feedback"></a>其他意見反應

想要將項目新增到常見問題集嗎？ 在註解區段中開啟新的意見反應問題，或設定此頁面與 GitHub 的提取要求 ！
