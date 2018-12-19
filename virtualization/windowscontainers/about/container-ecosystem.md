---
title: 容器生態系統
description: 建置容器生態系統。
keywords: 中繼資料, 容器
author: PatrickLang
ms.date: 04/20/2016
ms.topic: about-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 29fbe13a-228a-4eaa-9d4d-90ae60da5965
ms.openlocfilehash: fec8b08d5bd86102d21ce93b726dc0b931413441
ms.sourcegitcommit: 69228f28723f366c6b2933c7701077c8df74b32b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/19/2018
ms.locfileid: "8976744"
---
# <a name="building-a-container-ecosystem"></a>建置容器生態系統

為了解建置容器生態系統的重要性，我們要先討論 Docker。

## <a name="docker"></a>Docker

容器的概念 (命名空間隔離和資源控管) 行之已久，可回溯到 BSD Jails、Solaris Zones 和基本 UNIX chroot (變更根) 機制。   Docker 做的是提供常用的工具集、封裝模型和部署機制。  如此一來，Docker 簡化的容器化和散發的應用程式。  然後，這些應用程式就可以在任何 Linux 主機上的任何位置執行，這也是我們在 Windows 上提供的功能。

普遍的封裝模型和部署技術提供對任何主機相同的管理命令，以簡化管理，並建立順暢性的絕佳機會。

從開發人員的桌面，到測試機器，乃至於一組實際執行機器，皆可建立 Docker 映像，進而快速地在任何環境以相同方式部署。 在此發展下，封裝在 Docker 容器中的應用程式建立起大規模且持續增長的生態系統；所使用的是 DockerHub，這是 Docker 所維護的公用容器化應用程式登錄。

Docker 為開發提供了絕佳基礎。

現在我們要談談應用程式的生態系統，以及如何根據 Docker 的概念建立適合您的需求的開發和部署工作流程。

## <a name="components-in-a-container-ecosystem"></a>容器生態系統中的元件

Windows 容器是大型容器生態系統的要件。 我們正著力於此產業，以期能讓開發人員選擇各種層級的解決方案組合。

容器生態系統可用來管理容器、共用容器，以及開發在容器中執行的應用程式。

![](media/containerEcosystem.png)

Microsoft 想要讓開發人員在建置這些新一代的應用程式時有所選擇並保有生產力。  我們的目標是要推升開發人員生產力，亦即讓應用程式能以任何 Microsoft 雲端為目標，且無需修改、重寫或重新設定程式碼。

Microsoft 致力於開放環境與生態系統的營造。  我們歡迎眾多志同道合的開發人員加入生態系統 (例如 Windows 和 Linux)，共同推動創新。

在接下來的幾個月，我們將提供關於此開發生態系統中其他合作夥伴的詳細資訊。
