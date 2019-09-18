---
title: Windows 上的 Linux 容器
description: 瞭解您可以使用 Hyper-v 在 Windows 上執行 Linux 容器的不同方法，就如同它們是本機的一樣。
keywords: LCOW、linux 容器、docker、容器
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 9100d2218c160bbe9fbf24f3524c8ff5e3dd826c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2019
ms.locfileid: "10135321"
---
# <a name="linux-containers-on-windows"></a>Windows 上的 Linux 容器

Linux 容器占整個容器生態系統的巨大百分比，而且是開發人員體驗與生產環境的基礎。  因為樹枝與容器主機共用內核，所以直接在 Windows 上執行 Linux 容器並不是一個選項[*](linux-containers.md#other-options-we-considered)。  這就是虛擬化進入圖片的地方。

現在，有兩種方式可使用 Windows 和 Hyper-v 的 Docker 來執行 Linux 容器：

- 在完整 Linux VM 中執行 Linux 容器-這是目前的 Docker。
- 使用[hyper-v 隔離](../manage-containers/hyperv-container.md)（LCOW）執行 Linux 容器-這是 Windows 的 Docker 中的新選項。

本文將簡要說明每個方法的運作方式，提供有關何時選擇要使用哪個解決方案以及共用作業進行中的指導方針。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM 中的 Linux 容器

若要在 Linux VM 中執行 Linux 容器，請依照[Docker 快速入門手冊](https://docs.docker.com/docker-for-windows/)中的指示操作。

在使用 Hyper-v 上執行的[LinuxKit](https://github.com/linuxkit/linuxkit)虛擬機器，2016在 windows 電腦上第一次發行之後，Docker 就能在 windows 桌上型電腦上執行 Linux 容器。

在這個模型中，Docker 用戶端會在 Windows 電腦上執行，但在 Linux VM 上呼叫到 Docker 守護程式。

![Moby VM 做為容器主機](media/MobyVM.png)

在這個模型中，所有 Linux 容器都共用單一的 Linux 基礎容器主機及所有 Linux 容器：

* 彼此共用內核與 Moby VM，但不與 Windows 主機共用。
* 在 Linux 上執行的 Linux 樹枝有一致的儲存和網路屬性（因為它們是在 Linux VM 上執行）。

這也表示 Linux 容器主機（Moby VM）需要執行 Docker 守護程式及所有 Docker 守護程式的相依性。

若要查看您是否正在使用 Moby VM 執行，請使用 Hyper-v Manager UI 或`Get-VM`在提升的 PowerShell 視窗中執行，以檢查 Moby VM 的 hyper-v 管理員。

## <a name="linux-containers-with-hyper-v-isolation"></a>含 Hyper-v 隔離的 Linux 容器

若要在 Windows （LCOW）上試用 Linux 容器，請按照[windows 10 上的 linux](../quick-start/quick-start-windows-10-linux.md)容器中的 linux 容器指示進行。

使用 Hyper-v 隔離的 Linux 容器在已優化的 Linux VM 中執行每個 Linux 容器，只有足夠的 OS，才能執行容器。 與 Moby VM 的做法相反，每個 Linux 容器都有自己的內核及其自己的 VM 沙箱。 它們也是由 Windows 上的 Docker 直接管理。

![使用 Hyper-v 隔離（LCOW）的 Linux 容器](media/lcow-approach.png)

深入瞭解容器管理在 Moby VM 做法和 LCOW 之間有何不同，在 LCOW 模型容器管理中會保留在 Windows 上，且每個 LCOW 管理都是透過 GRPC 和 containerd 進行。  這表示 Linux distro 容器用於 LCOW 的庫存量可能較小。  現在，我們使用 LinuxKit 來優化 distro 容器，但其他專案（例如 Kata）也會建立類似的高調諧 Linux distros （明文 Linux）。

以下是每個 LCOW 的深入瞭解：

![LCOW 架構](media/lcow.png)

若要查看您是否正在執行 LCOW，請`C:\Program Files\Linux Containers`流覽至。 如果 Docker 設定為使用 LCOW，將會有幾個檔案，其中包含在 Hyper-v 隔離下執行的每個容器中執行的最小 LinuxKit distro。  請注意，優化的 VM 元件小於 100 MB，比 Moby VM 中的 LinuxKit 影像要小得多。

### <a name="work-in-progress"></a>進行中的工作

LCOW 處於 [作用中開發] 下。 追蹤[GitHub](https://github.com/moby/moby/issues/33850)上 Moby 專案中的正在進行的進度

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

這些應用程式都需要大量對應，且無法正常啟動或執行。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>額外資訊

[LCOW 的 Docker 博客說明](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 容器影片](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-內核 plus 組建指示](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>何時使用 Moby VM vs LCOW

### <a name="when-to-use-moby-vm"></a>何時使用 Moby VM

我們建議您在下列情況下，執行 Linux 樹枝的 Moby VM 方法：

- 需要穩定的容器環境。  這是 Windows 預設的 Docker。
- 執行 Windows 或 Linux 容器，但同時很少同時執行兩次。
- 在 Linux 容器之間有複雜或自訂的網路需求。
- 在 Linux 容器之間不需要內核隔離（Hyper-v 隔離）。

### <a name="when-to-use-lcow"></a>何時使用 LCOW

現在，我們建議您 LCOW 給下列人員：

- 想要測試我們最新的技術。
- 同時執行 Windows 和 Linux 容器。
- 在 Linux 容器之間需要內核隔離（Hyper-v 隔離）。

## <a name="other-options-we-considered"></a>我們考慮的其他選項

當我們想瞭解如何在 Windows 上執行 Linux 容器時，我們認為 WSL。 最後，我們選擇了虛擬化的方法，讓 Windows 上的 Linux 容器與 Linux 上的 Linux 容器保持一致。 使用 Hyper-v 也能讓 LCOW 更安全。 我們可能會在未來重新評估，但現在 LCOW 會繼續使用 Hyper-v。

如果您有任何想法，請透過 GitHub 或 UserVoice 傳送意見反應。  我們特別感謝您想要查看的特定體驗意見。
