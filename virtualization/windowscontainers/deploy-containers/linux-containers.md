---
title: Windows 上的 Linux 容器
description: 瞭解您可以使用 Hyper-v 在 Windows 上執行 Linux 容器的不同方式，就像是原生一樣。
keywords: LCOW，linux 容器，docker，容器
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910568"
---
# <a name="linux-containers-on-windows"></a>Windows 上的 Linux 容器

Linux 容器占整體容器生態系統的龐大百分比，同時也是開發人員經驗和生產環境的基礎。  由於容器與容器主機共用核心，因此，在 Windows 上直接執行 Linux 容器不是[*](linux-containers.md#other-options-we-considered)的選項。  這就是虛擬化進入此目標的地方。

現在有兩種方式可以使用適用於 Windows 的 Docker 和 Hyper-v 來執行 Linux 容器：

- 在完整的 Linux VM 中執行 Linux 容器-這是 Docker 通常會執行的工作。
- 使用[hyper-v 隔離](../manage-containers/hyperv-container.md)執行 Linux 容器（LCOW）-這是適用於 Windows 的 Docker 中的新選項。

本文概述每種方法的運作方式、提供何時選擇哪些解決方案，以及共用進行中工作的指引。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM 中的 Linux 容器

若要在 Linux VM 中執行 Linux 容器，請遵循[Docker 的入門指南](https://docs.docker.com/docker-for-windows/)中的指示。

Docker 已能夠在 Windows 桌面上執行 Linux 容器，因為它是第一次在2016中發行（在 Hyper-v 隔離或 Windows 上的 Linux 容器可用之前）使用在 Hyper-v 上執行的[LinuxKit](https://github.com/linuxkit/linuxkit)型虛擬機器。

在此模型中，Docker 用戶端會在 Windows 桌面上執行，但會在 Linux VM 上呼叫 Docker Daemon。

![作為容器主機的 Moby VM](media/MobyVM.png)

在此模型中，所有 Linux 容器都會共用一個以 Linux 為基礎的容器主機和所有 Linux 容器：

* 與彼此和 Moby VM 共用核心，但不與 Windows 主機共用。
* 在 linux 上執行的 Linux 容器具有一致的儲存體和網路屬性（因為它們是在 Linux VM 上執行）。

這也表示 Linux 容器主機（Moby VM）必須執行 Docker Daemon 和所有 Docker Daemon 的相依性。

若要查看您是否使用 Moby VM 執行，請使用 Hyper-v 管理員 UI 或在提高許可權的 PowerShell 視窗中執行 `Get-VM`，以檢查 Moby VM 的 Hyper-v 管理員。

## <a name="linux-containers-with-hyper-v-isolation"></a>具有 Hyper-v 隔離的 Linux 容器

若要嘗試 Windows 上的 Linux 容器（LCOW），請遵循[windows 10 上 linux](../quick-start/quick-start-windows-10-linux.md)容器中的 linux 容器指示。

具有 Hyper-v 隔離的 Linux 容器會在優化的 Linux VM 中執行每個 Linux 容器，只要有足夠的 OS 即可執行容器。 相對於 Moby VM 方法，每個 Linux 容器都有自己的核心和自己的 VM 沙箱。 它們也是由 Windows 上的 Docker 直接管理。

![具有 Hyper-v 隔離的 Linux 容器（LCOW）](media/lcow-approach.png)

若要深入瞭解容器管理在 Moby VM 方法與 LCOW 之間的差異，請參閱 LCOW 模型容器管理會保留在 Windows 上，而且每個 LCOW 管理都會透過 GRPC 和 containerd 進行。  這表示用於 LCOW 的 Linux 散發版本容器可以有較小的清查。  目前，我們將使用 LinuxKit 來進行優化的散發版本容器，但 Kata 等其他專案也會建立類似的高度微調 Linux 散發版本（Clear Linux）。

以下是每個 LCOW 的深入探討：

![LCOW 架構](media/lcow.png)

若要查看您是否正在執行 LCOW，請流覽至 `C:\Program Files\Linux Containers`。 如果 Docker 設定為使用 LCOW，則這裡會有一些檔案，其中包含在 Hyper-v 隔離下執行的每個容器中執行的最小 LinuxKit 散發版本。  請注意，優化的 VM 元件小於 100 MB，遠小於 Moby VM 中的 LinuxKit 映射。

### <a name="work-in-progress"></a>進行中的工作

LCOW 正在開發中。 在[GitHub](https://github.com/moby/moby/issues/33850)上的 Moby 專案中追蹤進行中的進度

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

這些應用程式都需要磁片區對應，而且將無法正確啟動或執行。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>額外資訊

[描述 LCOW 的 Docker blog](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux 容器影片](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-核心 plus 組建指示](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>使用 Moby VM 與 LCOW 的時機

### <a name="when-to-use-moby-vm"></a>使用 Moby VM 的時機

目前，我們建議您將執行 Linux 容器的 Moby VM 方法用於下列人員：

- 需要穩定的容器環境。  這是適用於 Windows 的 Docker 預設值。
- 執行 Windows 或 Linux 容器，但很少同時這麼做。
- 在 Linux 容器之間具有複雜或自訂的網路需求。
- 不需要 Linux 容器之間的內核隔離（Hyper-v 隔離）。

### <a name="when-to-use-lcow"></a>使用 LCOW 的時機

目前，我們建議 LCOW 給下列人員：

- 想要測試我們最新的技術。
- 同時執行 Windows 和 Linux 容器。
- 需要 Linux 容器之間的內核隔離（Hyper-v 隔離）。

## <a name="other-options-we-considered"></a>我們考慮的其他選項

當我們查看在 Windows 上執行 Linux 容器的方式時，我們會將 WSL 視為。 最後，我們選擇以虛擬化為基礎的方法，讓 Windows 上的 Linux 容器與 Linux 上的 Linux 容器一致。 使用 Hyper-v 也會讓 LCOW 更安全。 我們可能會在未來重新評估，但目前 LCOW 將繼續使用 Hyper-v。

如果您有任何想法，請透過 GitHub 或 UserVoice 傳送意見反應。  我們特別感謝您想要查看之特定體驗的意見反應。
