---
title: "Windows 容器快速入門"
description: "Windows 容器快速入門。"
keywords: "docker, 容器"
author: enderb-ms
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-contianers
ms.service: windows-containers
ms.assetid: 4878f5d2-014f-4f3c-9933-97f03348a147
ms.openlocfilehash: 577618112339e274b3d25908afc5428bd1c36173
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
---
# <a name="windows-containers-quick-start"></a>Windows 容器快速入門

Windows 容器快速入門會介紹產品和容器術語、逐步說明簡單的容器部署範例，也提供更進階主題的參考。 如果您不熟悉容器或 Windows 容器，逐步檢視本快速入門中的每個步驟將提供您實際操作技術的體驗。

## <a name="1-what-are-containers"></a>1.什麼是容器？

它們是獨立且由資源控制的可攜式作業環境。

基本上，容器是可讓應用程式執行而不會影響系統的其餘部分，且不會讓系統影響應用程式的隔離位置。 容器是虛擬化下一步的發展。

當您在容器內時，您會感覺像是在全新安裝的實體電腦或虛擬機器內。 同時，若是 [Docker](https://www.docker.com/)，Windows 容器可以用和其他任何容器一樣的方式進行管理。

## <a name="2-windows-container-types"></a>2.Windows 容器類型

Windows 容器包含兩種不同的容器類型，或執行階段。

**Windows Server 容器** – 透過程序和命名空間隔離技術，提供應用程式隔離功能。 Windows Server 容器可與容器主機和所有執行於主機上的容器共用核心。

**Hyper-V 容器** – 藉由在高度最佳化的虛擬機器中執行每個容器，擴充 Windows Server 容器所提供的隔離能力。 在此設定中，容器主機的核心不會與其他 Hyper-V 容器共用。

## <a name="3-container-fundamentals"></a>3.容器的基礎

當您開始使用容器時，您會發現容器和虛擬機器之間有許多相似之處。 容器可像實體或虛擬電腦系統一樣執行作業系統、具有檔案系統，並且可透過網路來存取。 即便如此，容器的基本技術和概念還是與虛擬機器非常不同。 在您開始建立及使用 Windows 容器時，下列主要概念將有所幫助。 

**容器主機** – 設定了 Windows 容器功能的實體或虛擬電腦系統。

**容器 OS 映像** – 容器會從映像進行部署。 在可能構成容器的眾多映像層中，容器 OS 映像是第一層。 此映像提供作業系統環境。

**容器映像** – 容器映像包含基礎作業系統、應用程式，以及快速部署容器所需的所有應用程式相依性。 

**容器登錄** – 容器映像儲存在容器登錄中，而且可以隨選下載。 

**Dockerfile** - Dockerfile 用來自動建立容器映像。

## <a name="next-step"></a>後續步驟：

[Windows Server 容器快速入門](quick-start-windows-server.md)  

[Windows 10 容器快速入門](quick-start-windows-10.md)

