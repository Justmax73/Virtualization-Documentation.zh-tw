---
title: Windows 上的 Linux 容器
description: 深入了解您可以使用 HYPER-V 在 Windows 上執行 Linux 容器，因為如果它們原生的不同方式。
keywords: LCOW linux 容器，docker，容器
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 8597a93f035f5e451df8176d1563299120c95cb8
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380172"
---
# <a name="linux-containers-on-windows"></a>Windows 上的 Linux 容器

Linux 容器構成龐大整體容器生態系統的百分比，而是開發人員體驗和生產環境的基礎。  因為容器與容器主機共用核心，不過，直接在 Windows 上執行 Linux 容器之間沒有選項[*](linux-containers.md#other-options-we-considered)。  這是虛擬化會進入圖片。

以滑鼠右鍵，現在有兩種方式來執行 Linux 容器與 Docker for Windows 和 Hyper-v:

1. 這是 Docker 通常什麼現今中完整的 Linux VM-執行 Linux 容器。
1. 這是新的選項，Docker for Windows 中使用[HYPER-V 隔離](../manage-containers/hyperv-container.md)(LCOW)-執行 Linux 容器。

本文將概述每一種方法如何運作、 提供關於何時要選擇哪一個解決方案，指導方針，以及共用進行中的工作。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM 的 Linux 容器

若要在 Linux VM 中執行 Linux 容器，請依照[Docker 的 get 入門指南](https://docs.docker.com/docker-for-windows/)中的指示。

Docker 已經過能夠在 Windows 上執行 Linux 容器桌面，因為它首次發佈 2016年中 （HYPER-V 隔離或 LCOW 供使用之前） 使用[LinuxKit](https://github.com/linuxkit/linuxkit)基礎 HYPER-V 上執行的虛擬機器。

在此模型中，Docker 用戶端會執行在 Windows 桌面上，但會呼叫到 Linux VM 上的 Docker 精靈。

![Moby VM 與容器主機](media/MobyVM.png)

在此模型中，所有的 Linux 容器會共用單一的 linux 容器主機和所有的 Linux 容器：

* 彼此和 Moby VM，但與 Windows 主機共用核心。
* 有一致的儲存體和 Linux 容器 （因為它們在 Linux VM 上執行） 執行 linux 的網路功能的屬性。

這也表示 Linux 容器主機 (Moby VM) 必須執行 Docker 精靈和所有的 Docker 精靈相依性。

若要查看您是否正在執行與 Moby VM，檢查 Moby vm 使用 HYPER-V 管理員 UI，或是執行 HYPER-V 管理員`Get-VM`提升權限的 PowerShell 視窗中。

## <a name="linux-containers-with-hyper-v-isolation"></a>使用 HYPER-V 隔離的 Linux 容器

若要嘗試 LCOW，請依照[本 get 啟動 」 指南](../quick-start/quick-start-windows-10.md)中的 Linux 容器指示

使用 HYPER-V 隔離的 Linux 容器最佳化的 Linux VM 具備足夠的作業系統來執行容器中執行每個 Linux 容器 (LCOW)。  相較於其他 Moby VM 的方法，每個 LCOW 都有它自己的核心和其本身的 VM 沙箱。  他們正在也受 Windows 上的 Docker 直接。

![使用 HYPER-V 隔離 (LCOW) 的 Linux 容器](media/lcow-approach.png)

採取仔細看看容器管理 Moby VM 的方法與 LCOW 之間有何，在 LCOW 模型容器管理停留在 Windows 上，每個 LCOW 管理會透過 GRPC 和 containerd 發生。  這表示針對 LCOW 使用 Linux distro 容器可以有多較小的詳細目錄。  向右現在，我們將使用 LinuxKit 最佳化的 distro 容器使用，但像 Kata 其他專案會建置類似高度微調 Linux 散發版本 (清楚 Linux) 也。

以下是仔細看看每個 LCOW:

![LCOW 架構](media/lcow.png)

若要查看您是否正在執行 LCOW，瀏覽至`C:\Program Files\Linux Containers`。 如果 Docker 會設定為使用 LCOW，會有幾個檔案，以下其中包含最少的 LinuxKit distro HYPER-V 隔離下執行每個容器中執行。  請注意的最佳化的 VM 元件是不超過 100MB，遠小於 Moby VM 的 LinuxKit 映像。

### <a name="work-in-progress"></a>進行中的工作

LCOW 已是開發中。 [GitHub](https://github.com/moby/moby/issues/33850)上追蹤 Moby project 中的持續進度

#### <a name="bind-mounts"></a>繫結裝載

使用 `docker run -v ...` 的繫結裝載磁碟區會將檔案儲存在 Windows NTFS 檔案系統上，所以 POSIX 作業需要一些轉譯。 某些檔案系統作業目前只有一部分或是尚未實作，這可能會導致某些應用程式不相容。

這些作業目前並未針對繫結裝載磁碟區工作：

* MkNod
* XAttrWalk
* XAttrCreate
* Lock
* Getlock
* Auth
* Flush
* INotify

也有一些作業並未完整實作：

* GetAttr – Nlink 計數永遠回報為 2
* Open – 只會實作 ReadWrite、WriteOnly 和 ReadOnly 旗標

這些應用程式都需要磁碟區對應，將不啟動或正確執行。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>額外的資訊

[描述 LCOW docker 部落格](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 容器影片](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW 核心加上建置指示](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>使用 Moby VM vs LCOW 的時機

### <a name="when-to-use-moby-vm"></a>使用 Moby VM 的時機

向右現在，我們建議執行 Linux 容器連接至誰 Moby VM 的方法：

- 想要在穩定的容器環境。  這是 Docker for Windows 預設值。
- Windows 或 Linux 容器，但很少兩者同時執行。
- 很複雜或自訂的網路功能需求之間的 Linux 容器。
- 不需要的 Linux 容器之間的核心隔離 （HYPER-V 隔離）。

### <a name="when-to-use-lcow"></a>使用 LCOW 的時機

向右現在，我們建議以誰 LCOW:

- 想要測試我們最新的技術。
- 在此同時執行 Windows 和 Linux 容器。
- 需要核心隔離 （HYPER-V 隔離） 之間的 Linux 容器。

## <a name="other-options-we-considered"></a>我們考慮其他選項

當我們已看著 Windows 上執行 Linux 容器的方式時，我們就會被視為 WSL。 最後，我們選擇的虛擬化方法，這樣一來在 Windows 上的 Linux 容器與 Linux 上的 Linux 容器一致。 使用 HYPER-V 也可以讓 LCOW 更安全。 我們可能會重新評估在未來，但現在，LCOW 仍將繼續使用 HYPER-V。

如果您有想法，請傳送意見反應，透過 GitHub 或 UserVoice。  尤其是，我們非常感謝您想要查看特定體驗相關意見反應。
