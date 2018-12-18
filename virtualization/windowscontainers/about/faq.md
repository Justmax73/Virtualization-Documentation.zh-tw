---
title: Windows 容器常見問題集
description: Windows 容器常見問題集
keywords: docker, 容器
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 7ac5622ad0f6767fa8d55798e0bf99b84f0d0a2c
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973698"
---
# <a name="frequently-asked-questions"></a>常見問題集

## <a name="general"></a>一般

### <a name="what-is-wcow-what-is-lcow"></a>什麼是 WCOW？ 什麼是 LCOW？

WCOW 是縮寫，適用於 Windows 和 LCOW 上 Windows 容器是在 Windows 上的 Linux 容器的縮寫。

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Linux 與 Windows Server 容器有何差異？

Linux 與 Windows Server 容器很類似 -- 兩者都在其核心和核心作業系統內實作類似的技術。 差異來自於平台以及在容器內執行的工作負載。  
客戶在使用 Windows Server 容器時，可以整合現有的 Windows 技術，例如 .NET、ASP.NET、PowerShell 等等。

### <a name="as-a-developer-do-i-have-to-re-write-my-app-for-each-type-of-container"></a>身為開發人員，我是否必須為每一種容器重新撰寫應用程式？

否，Windows 容器映像在 Windows Server 容器與 Hyper-V 容器之間是通用的。 容器類型的選擇會在您啟動容器時決定。 就開發人員的觀點而言，Windows Server 容器和 Hyper-V 容器只是風格上的不同。 它們提供相同的開發、程式設計和管理體驗，是開放且可延伸的，且未來在使用 Docker 時也將有相同層級的整合能力和支援。

開發人員可以使用 Windows Server 容器來建立容器映像，並將它部署為 Hyper-V 容器 (反之亦然)，除了須指定適當的執行階段旗標以外，不需進行任何變更。

如果速度是關鍵考量，Windows Server 容器將可提供較高的密度和效能 (例如，相較於巢狀設定，加速所需時間較短，執行階段效能更快速)。 Hyper-V 容器可提供更高的隔離性，確保一個容器中執行的程式碼不會危害或影響到主機作業系統或在相同主機上執行的其他容器。 這對於包含 SaaS 應用程式和計算裝載的多租用戶案例 (有裝載非信任程式碼的需求)，會很有幫助。

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>在 Windows 上執行容器的必要條件為何？

容器是平台從 Windows Server 2016 中引進。 您必須執行 Windows Server 2016 或 Windows 10 年度更新版 （版本 1607年） 或更新版本來使用容器。

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>我能夠 Windows 容器中執行處理序隔離模式 Windows 10 企業版或專業版？

從 10 年 10 月 2018 年開始使用 Windows Update，我們不會再不允許從執行 Windows 容器的處理序隔離的使用者。 您必須直接要求的處理序隔離，藉由使用`--isolation=process`旗標時執行您的容器，透過`docker run`。

如果這是您感興趣的項目，您需要請確定您的主機執行 Windows 10 組建 17763 +，而且您有使用引擎 18.09 的 docker 版本或更新版本。

> [!WARNING]
> 這項功能僅適用於開發/測試。 您應該繼續使用 Windows Server 與主機，生產環境部署。
>
> 藉由使用這項功能，您也必須確保，您的主機和容器版本標記相符，否則容器可能無法啟動或可能會呈現未定義的行為。

## <a name="windows-container-management"></a>Windows 容器管理

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>如何不要讓我的容器映像可用在空中其電腦上？

Windows 容器基本映像包含其散發授權受限制的成品。 當您建置這些映像上，並且它們推送到私人或公開登錄中時，您會注意到的基底層永遠不會推入。 相反地，我們會使用外部的層級會指向位於 Azure 雲端儲存空間中的實際基底層的概念。

當您有可以_只_從_您_的私人容器登錄的位址提取映像在空中其電腦時，這可以呈現問題。 嘗試依照外部的層，以取得基礎映像會在此情況下失敗。 若要覆寫外部層行為，您可以使用`--allow-nondistributable-artifacts`Docker 精靈中的旗標。

> [!IMPORTANT]
> 這個旗標的使用方式不應該以防止遵守 Windows 容器基本映像授權; 規定您義務您必須不張貼公用或第 3 方重新發佈的 Windows 內容。 允許您自己的環境中的使用方式。

## <a name="microsofts-open-ecosystem"></a>Microsoft 的開放生態系統

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Microsoft 是否參與開放容器計劃 (Open Container Initiative, OCI)?

為了確保封裝格式的通用性，Docker 最近發起開放容器計劃 (OCI)，以期能讓容器封裝維持開放和基金會領導的形式，而 Microsoft 也是發起成員之一。

> ![提示]必須為常見問題集的新增的建議？ 我們鼓勵您新的意見反應問題下方或開啟您建議針對這些文件 PR ！
