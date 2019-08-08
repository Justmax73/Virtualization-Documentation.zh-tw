---
title: Windows 容器平臺
description: 深入瞭解 Windows 中提供的新容器構造區塊。
keywords: LCOW、linux 容器、docker、容器、containerd、cri、runhcs、runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998415"
---
# <a name="container-platform-tools-on-windows"></a>Windows 上的容器平臺工具

Windows 容器平臺正在擴充! Docker 是容器旅程的第一個部分, 現在我們正在建立其他容器平臺工具。

* [containerd/cri](https://github.com/containerd/cri) -Windows Server 2019/windows 10 1809 中的新增功能。
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -Runc 的 Windows 容器主機對應專案。
* [hcs](https://docs.microsoft.com/virtualization/api/) -主機計算服務 + 簡單的填充程式, 讓您更容易使用。
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

本文將討論 Windows 和 Linux 容器平臺以及每個容器平臺工具的相關資訊。

## <a name="windows-and-linux-container-platform"></a>Windows 和 Linux 容器平臺

在 Linux 環境中, 像 Docker 的容器管理工具是以一組更精細的容器工具 ( [runc](https://github.com/opencontainers/runc)和[containerd](https://containerd.io/)) 為基礎。

![Linux 上的 Docker 架構](media/docker-on-linux.png)

`runc` 是 Linux 命令列工具, 可根據[OCI 容器執行時間規格](https://github.com/opencontainers/runtime-spec)來建立及執行容器。

`containerd` 是一個守護程式, 可管理容器的生命週期, 從下載並將容器影像解包至容器執行與監察。

在 Windows 上, 我們採取了不同的方法。  當我們開始使用 Docker 來支援 Windows 容器時, 我們會直接在 HCS (主機計算服務) 上建立。  [此博客文章](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332)完整說明我們為什麼要建立 HCS, 以及為什麼我們會針對容器進行這種做法。

![Windows 上的初始 Docker 引擎架構](media/hcs.png)

此時, Docker 仍會直接呼叫 HCS。 接著, 我們會將容器管理工具展開以納入 Windows 容器, Windows 容器主機可以呼叫 containerd, 並 runhcs 其在 containerd 上呼叫的方式, 以及在 Linux 上的 runc。

## <a name="runhcs"></a>runhcs

`runhcs` 是的`runc`叉式。  Like `runc`) `runhcs`是一種命令列客戶程式, 用來執行根據開放容器倡議 (OCI) 格式封裝的應用程式, 且是已開啟容器方案規格的相容性實現。

Runc 和 runhcs 之間的功能差異包括:

* `runhcs` 在 Windows 上執行。  它會與[HCS](containerd.md#hcs)進行通訊, 以建立和管理容器。
* `runhcs` 可以執行各種不同的容器類型。

  * Windows 和 Linux [hyper-v 隔離](../manage-containers/hyperv-container.md)
  * Windows 程式容器 (容器影像必須符合容器主機)

**使用方式：**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 是您要啟動之容器實例的名稱。 名稱在您的容器主機上必須是唯一的。

套件目錄 (使用`-b bundle`) 是選擇性的。  
就像 runc 一樣, 樹枝是使用束進行設定。 容器的束套件是容器的 OCI 規格檔案 "config. json" 的目錄。  "束" 的預設值是目前的目錄。

OCI 規格檔案 "config. json" 必須具有兩個欄位才能正確執行:

* 容器暫存空間的路徑
* 容器的圖層目錄路徑

Runhcs 中可用的容器命令包括:

* 建立及執行容器的工具
  * **執行**建立並執行容器
  * **建立**容器

* 管理容器中正在執行的進程的工具:
  * **start**在已建立的容器中執行使用者定義的進程
  * **exec**在容器內執行新的程式
  * **暫停**暫停會暫停容器內的所有進程
  * **resume** : 繼續繼續先前已暫停的所有進程
  * **ps** ps 會顯示正在容器內執行的進程

* 管理容器狀態的工具
  * [**狀態**] 會輸出容器的狀態
  * **kill**會將指定的信號 (預設: SIGTERM) 傳送到容器的 init 進程
  * **delete**刪除容器所佔用的任何資源, 通常與已分離的容器搭配使用

可以視為 [多樹枝] 的唯一命令是 [**清單**]。  它會列出由 runhcs 使用指定根目錄啟動或暫停的容器。

### <a name="hcs"></a>HCS

GitHub 上提供兩個包裝程式, 以與 HCS 介面取得介面。 因為 HCS 是一個 C API, 所以包裝程式可讓您輕鬆地從較高層語言呼叫 HCS。  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim 的撰寫方式, 且是 runhcs 的基礎。
從 AppVeyor 中取得最新的內容, 或自行組建。
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-COMPUTEVIRTUALIZATION 是 HCS 的 c # 包裝器。

如果您想要使用 HCS (直接或透過包裝), 或想要在 HCS 周圍製作 Rust/Haskell/InsertYourLanguage 包裝, 請留言給您的留言。

若要深入瞭解 HCS, 請觀看[John Stark 的 DockerCon 簡報](https://www.youtube.com/watch?v=85nCF5S8Qok)。

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 支援服務僅適用于 Server 2019/Windows 10 1809 及更新版本。  我們也仍在積極地開發 Windows 版 containerd。
> 僅限開發人員/測試。

雖然 OCI 規格定義單一容器, 但[CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (容器執行時間介面) 在共用沙箱環境 (稱為 pod) 中描述容器做為工作負荷。  盒可以包含一或多個容器工作負荷。  箱讓樹枝 orchestrators (例如 Kubernetes 和服務結構網格) 處理的群組工作負載應與一些共用資源 (例如記憶體和 vNETs) 位於同一個主機上。

containerd/cri 啟用下列盒式相容性矩陣:

| 主機作業系統 | 容器作業系統 | 單獨 | Pod 支援？ |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | 是-支援真正的多容器盒式箱。 |
|  | Windows Server 2019/1809 | `process`* 或 `hyperv` | Yes (是): 如果每個工作負荷容器 OS 都符合公用機作業系統作業系統, 就支援 true 個容器箱。 |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | 部分: 支援盒式沙箱, 如果容器作業系統與公用程式作業系統相符, 就能支援針對每個公用程式 VM 的單一進程隔離容器。 |

\ * Windows 10 主機僅支援 Hyper-v 隔離

CRI 規格的連結:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod 規格
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -工作負載規格

![以 Containerd 為基礎的容器環境](media/containerd-platform.png)

雖然 runHCS 和 containerd 都能在任何 Windows system Server 2016 或更新版本上進行管理, 但支援盒 (容器群組) 需要中斷對 Windows 中容器工具所做的變更。  CRI 支援可在 Windows Server 2019/Windows 10 1809 及更新版本中取得。
