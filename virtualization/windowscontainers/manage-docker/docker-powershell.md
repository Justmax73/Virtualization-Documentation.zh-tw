---
title: 適用於 Docker 的 PowerShell
description: 如何使用 PowerShell 管理 Docker 容器
keywords: docker, 容器, powershell
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 4a0e907d-0d07-42f8-8203-2593391071da
ms.openlocfilehash: bcbc2e4e76c48a3d9a1a9720b09ef366a396bf30
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998177"
---
### <a name="powershell-for-docker"></a>適用於 Docker 的 PowerShell

不論是您，或是來自論壇、Twitter、GitHub 的使用者，都提過這個問題 (甚至也有人親自提問過)：為何不能透過 PowerShell 查看 Docker 容器？ 

而我們也與您探討過其中的利弊，結論是容器 PowerShell 模組需要更新… 因此，我們即將淘汰已在 Windows Server 2016 預覽版中發佈的容器 PowerShell 模組，並著手以適用於 Docker 的全新 PowerShell 模組加以取代。  這個新模組的開發已經進行，但我們採用與過去截然不同的新方式 – 我們是以開放的方式進行開發。  我們期望此模組是社群共同作業的成果，並可提供透過 Docker 引擎的絕佳 PowerShell 容器體驗。  這個新模組是直接建置在 Docker 引擎的 REST 介面之上，使用者可以選擇使用 Docker CLI、PowerShell 或兩者。

建置絕佳的 PowerShell 模組並不容易，因為一方面要確保所有程式碼正確、取得物件和參數集的平衡，還要注意 Cmdlet 名稱，這些要素都缺一不可。  因此，在採用這個新模組的過程中，我們必須仰賴您 – 我們的使用者以及大量 PowerShell 和 Docker 社群，來協助塑造此模組。  您認為哪些參數集非常重要？  是要採用與 “docker run” 對等的方法，或是將 new-container 輸送至 start-container – 您喜歡哪一種方法？  若要深入瞭解此模組並參與開發, 請參閱 GitHub 頁面 (https://github.com/Microsoft/Docker-PowerShell/)並加入。

開發進行期間，當我們取得穩定的 Alpha 品質模組時，即會將其發佈至 PowerShell 資源庫，並更新此頁面與使用方式說明。
