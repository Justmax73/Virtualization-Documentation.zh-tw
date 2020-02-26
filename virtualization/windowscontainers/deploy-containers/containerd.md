---
title: Windows 容器平臺
description: 深入瞭解 Windows 中可用的新容器建立區塊。
keywords: LCOW，linux 容器，docker，容器，containerd，cri，runhcs，runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439275"
---
# <a name="container-platform-tools-on-windows"></a>Windows 上的容器平臺工具

Windows 容器平臺正在擴充！ Docker 是容器旅程的第一個部分，現在我們要建立其他容器平臺工具。

* [containerd/cri](https://github.com/containerd/cri) -windows Server 2019/windows 10 1809 中的新功能。
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -與 runc 對應的 Windows 容器主機。
* [hcs](https://docs.microsoft.com/virtualization/api/) -主機計算服務 + 便利的填充碼，讓您更容易使用。
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

本文將討論 Windows 和 Linux 容器平臺，以及每個容器平臺工具。

## <a name="windows-and-linux-container-platform"></a>Windows 和 Linux 容器平臺

在 Linux 環境中，容器管理工具（例如 Docker）是以更細微的容器工具集為基礎： [runc](https://github.com/opencontainers/runc)和[containerd](https://containerd.io/)。

![Linux 上的 Docker 架構](media/docker-on-linux.png)

`runc` 是 Linux 命令列工具，可根據[OCI 容器執行時間規格](https://github.com/opencontainers/runtime-spec)來建立和執行容器。

`containerd` 是一個可管理容器生命週期的守護程式，從下載容器映射並將其解壓縮至容器執行和監督。

在 Windows 上，我們採取了不同的方法。  當我們開始使用 Docker 來支援 Windows 容器時，我們會直接在 HCS （主機計算服務）上建立。  [這篇 blog 文章](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332)已完整說明我們為何建立 HCS，以及為何我們一開始就將此方法用於容器。

![Windows 上的初始 Docker 引擎架構](media/hcs.png)

此時，Docker 仍然會直接呼叫 HCS。 不過，繼續進行的容器管理工具會擴充為包含 Windows 容器，而 Windows 容器主機則可以呼叫 containerd 和 runhcs，方法是在 Linux 上的 containerd 和 runc 上呼叫。

## <a name="runhcs"></a>runhcs

`runhcs` 是 `runc`的分叉。  如同 `runc`，`runhcs` 是一種命令列用戶端，用來執行根據開放容器計畫（OCI）格式封裝的應用程式，並且是符合開放容器計畫規格的規範。

Runc 與 runhcs 之間的功能差異包括：

* `runhcs` 會在 Windows 上執行。  它會與[HCS](containerd.md#hcs)通訊，以建立及管理容器。
* `runhcs` 可以執行各種不同的容器類型。

  * Windows 和 Linux [hyper-v 隔離](../manage-containers/hyperv-container.md)
  * Windows 進程容器（容器映射必須符合容器主機）

**實例**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 是您要啟動之容器實例的名稱。 名稱在您的容器主機上必須是唯一的。

配套目錄（使用 `-b bundle`）是選擇性的。  
如同 runc，容器是使用配套進行設定。 容器的配套是具有容器的 OCI 規格檔案 "config.xml" 的目錄。  「組合」的預設值是目前的目錄。

OCI 規格檔 "config.xml" 必須要有兩個欄位，才能正確執行：

* 容器的臨時空間路徑
* 容器層目錄的路徑

Runhcs 中可用的容器命令包括：

* 用來建立和執行容器的工具
  * **執行**會建立並執行容器
  * **建立**容器的建立

* 管理容器中執行之進程的工具：
  * **開始**在已建立的容器中執行使用者定義的進程
  * **exec**會在容器內執行新的進程
  * **暫停**暫停會暫停容器內的所有進程
  * **resume**會繼續所有先前已暫停的進程
  * **ps** ps 會顯示在容器內執行的處理常式

* 用來管理容器狀態的工具
  * **狀態**輸出容器的狀態
  * **kill**將指定的信號（預設值： SIGTERM）傳送至容器的 init 進程
  * **delete**刪除經常與卸離的容器搭配使用的容器所持有的任何資源

唯一可視為多容器的命令是**清單**。  它會列出 runhcs 使用指定的根啟動的執行中或已暫停的容器。

### <a name="hcs"></a>HCS

GitHub 上有兩個包裝函式可與 HCS 互動。 由於 HCS 是 C API，因此包裝函式可讓您輕鬆地從較高層級的語言呼叫 HCS。  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim 是以 Go 撰寫的，它是 runhcs 的基礎。
從 AppVeyor 取得最新版本，或自行建立。
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-COMPUTEVIRTUALIZATION 是 HCS C#的包裝函式。

如果您想要使用 HCS （直接或透過包裝函式），或想要在 HCS 周圍建立 Rust/Haskell/InsertYourLanguage 包裝函式，請留下批註。

如需深入瞭解 HCS，請觀賞[John Stark 的 DockerCon 簡報](https://www.youtube.com/watch?v=85nCF5S8Qok)。

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> CRI 支援僅適用于 Server 2019/Windows 10 1809 和更新版本。  我們也正在積極開發 Windows containerd。
> 僅限開發/測試。

雖然 OCI 規格會定義單一容器， [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) （容器執行時間介面）會將容器描述為共用沙箱環境中的工作負載（稱為 pod）。  Pod 可以包含一或多個容器工作負載。  Pod 可讓容器協調器（例如 Kubernetes 和 Service Fabric 網狀）處理群組的工作負載，這些工作負載應該與一些共用資源（例如記憶體和 Vnet）位於相同的主機上。

containerd/cri 會啟用 pod 的下列相容性矩陣：

| 主機 OS | 容器作業系統 | 實施 | Pod 支援？ |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | 是-支援真正的多容器 pod。 |
|  | Windows Server 2019/1809 | `process`* 或 `hyperv` | 是-如果每個工作負載容器 OS 都符合公用程式 VM 作業系統，支援真正的多容器 pod。 |
|  | Windows Server 2016、</br>Windows Server 1709、</br>Windows Server 1803 | `hyperv` | 部分：支援 pod 沙箱，如果容器 OS 符合公用程式 VM 作業系統，則可支援每個公用程式 VM 有單一進程隔離的容器。 |

\*Windows 10 主機僅支援 Hyper-v 隔離

CRI 規格的連結：

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod 規格
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) -工作負載規格

![以 Containerd 為基礎的容器環境](media/containerd-platform.png)

雖然 runHCS 和 containerd 都可以在任何 Windows 系統伺服器2016或更新版本上進行管理，但支援 pod （容器群組）需要在 Windows 中進行容器工具的重大變更。  CRI 支援適用于 Windows Server 2019/Windows 10 1809 及更新版本。
