---
title: "Windows 容器常見問題集"
description: "Windows 容器常見問題集"
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: c3a7decaf087741c82419a8a541867ae01f0f4da

---

# 常見問題集

## 關於 Windows 容器

**什麼是 Windows Server 容器？**

Windows Server 容器是一種輕量型作業系統虛擬化方法，可用來分隔應用程式或服務與其他執行於相同容器主機上的服務。 為了啟用此功能，每個容器在作業系統、程序、檔案系統、註冊和 IP 位址方面，都會有其本身的檢視。  

**什麼是 Hyper-V 容器？**

您可以將 Hyper-V 容器視為在 Hyper-V 磁碟分割內執行的 Windows Server 容器。

Hyper-V 容器在高效率、高密度的 Windows Server 容器與高隔離性的硬體虛擬化 Hyper-V 虛擬機器之間，提供了另一種部署選項。 如果環境中的應用程式來自相同主機上的不同信任界限，則可能需要進一步的隔離。 Hyper-V 容器可藉由最佳化虛擬化和 Windows Server 作業系統，分隔不同的容器以及分隔容器與主機作業系統，而提供更高的隔離性。 這兩種容器部署選項使用相同的管理 API、工具和映像格式，在部署期間，客戶可選擇最符合其需求的部署模式。

**Linux 與 Windows Server 容器有何差異？**

Linux 與 Windows Server 容器很類似 -- 兩者都在其核心和核心作業系統內實作類似的技術。 差異來自於平台以及在容器內執行的工作負載。  
客戶在使用 Windows Server 容器時，可以整合現有的 Windows 技術，例如 .NET、ASP.NET、PowerShell 等等。

**身為開發人員，我是否必須為每一種容器重新撰寫應用程式？**

否，Windows 容器映像在 Windows Server 容器與 Hyper-V 容器之間是通用的。 容器類型的選擇會在您啟動容器時決定。 就開發人員的觀點而言，Windows Server 容器和 Hyper-V 容器只是風格上的不同。  它們提供相同的開發、程式設計和管理體驗，是開放且可延伸的，且未來在使用 Docker 時也將有相同層級的整合能力和支援。

開發人員可以使用 Windows Server 容器來建立容器映像，並將它部署為 Hyper-V 容器 (反之亦然)，除了須指定適當的執行階段旗標以外，不需進行任何變更。

如果速度是關鍵考量，Windows Server 容器將可提供較高的密度和效能 (例如，相較於巢狀設定，加速所需時間較短，執行階段效能更快速)。 Hyper-V 容器可提供更高的隔離性，確保一個容器中執行的程式碼不會危害或影響到主機作業系統或在相同主機上執行的其他容器。 這對於包含 SaaS 應用程式和計算裝載的多租用戶案例 (有裝載非信任程式碼的需求)，會很有幫助。

**Hyper-V/Windows Server 容器是不是附加元件？還是會整合至 Windows Server？**

容器功能將會整合至 Windows Server 2016。 關於正式上市的詳細資訊將在近期提供。  

**Windows Server 容器與 Drawbridge 之間的關係為何？**

Drawbridge 是以往許多可協助我們深入了解容器的研究專案之一。  Windows Server 2016 中絶大部分所誕生的容器技術，來自於以往 Drawbridge 的經驗，我們很榮幸能夠藉由 Windows Server 2016 為我們的客戶提供世界級的容器技術。

**Windows Server 容器和 Hyper-V 容器的必要條件為何？**

Windows Server 容器和 Hyper-V 容器都需要 Windows Server 2016。 這些技術不適用於舊版的 Windows。


## Windows 容器管理

**Hyper-V 容器也可在 Docker 生態系統中使用嗎？**

可以 – Hyper-V 容器對於 Docker 會提供與 Windows Server 容器相同層級的整合性和管理性。  其目的是要提供開放、一致、跨平台的體驗。  
Docker 平台也可大幅簡化並提升跨容器選項的使用體驗。 使用 Windows Server 容器開發的應用程式可直接部署為 hyper-V 容器，而不需要變更。


**Windows 容器是否可在 ESXi 或其他非 Hyper-V Hypervisor 上執行？**

是，Windows 容器可在任何 TP3 Server Core 安裝上執行。  請遵循[就地啟用容器功能](../quick_start/inplace_setup.md) 的指示。

## Microsoft 的開放生態系統

**Microsoft 是否參與開放容器計劃 (Open Container Initiative, OCI)?**

為了確保封裝格式的通用性，Docker 最近發起開放容器計劃 (OCI)，以期能讓容器封裝維持開放和基金會領導的形式，而 Microsoft 也是發起成員之一。

**Microsoft 真的和 Docker 合作了嗎？**

可以。  
我們與 Docker 的合作關係，讓開發人員能夠以相同的 Docker 工具組建立、管理和部署 Windows Server 和 Linux 容器。 以 Windows Server 為目標的開發人員，將不再需要在使用眾多 Windows Server 技術與建置容器化應用程式之間選擇。  

Docker 有兩個重點，即專案的開放原始碼群組和 Docker 這家公司。 我們認為這項合作可達到此一雙重效益。 Docker 的成功有部分來自於藉由 Docker 容器技術建立起來的活躍生態系統。 Microsoft 會對 Docker Project 有所貢獻，進而支援 Windows Server 容器和 Hyper-V 容器。  

如需詳細資訊，請參閱 [New Windows Server containers and Azure support for Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD) (新的 Windows Server 容器與 Docker 的 Azure 支援) 部落格文章。



<!--HONumber=Jun16_HO4-->


