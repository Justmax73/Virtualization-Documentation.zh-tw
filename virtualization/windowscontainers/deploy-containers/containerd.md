---
title: 建置容器堆疊
description: 深入了解新容器建置組塊可以使用 Windows 中。
keywords: LCOW linux 容器，docker、 容器、 containerd、 cri、 runhcs，runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 8a68bf9e5e78add65aedb51fff8521ee258e353e
ms.sourcegitcommit: 9a61fc06c25d17ddb61124a21a3ca821828b833d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/20/2018
ms.locfileid: "7460493"
---
# <a name="container-platform-tools-on-windows"></a>在 Windows 上的容器平台工具

Windows 容器平台正在展開 ！  Docker 是旅程的容器，第一個部分，現在我們正在建置其他容器平台工具。

1. [containerd/cri](https://github.com/containerd/cri) -新的 Windows Server 2019/Windows 10 1809年。
1. [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) -runc 的 Windows 容器主機對應項。
1. [hcs](https://docs.microsoft.com/virtualization/api/) -為了方便使用的主機運算服務 + 好用的相容性修正。

    * [hcsshim](https://github.com/microsoft/hcsshim)
    * [dotnet computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

這篇文章將討論 Windows 和 Linux 容器平台，以及每個容器平台工具。

## <a name="windows-and-linux-container-platform"></a>Windows 和 Linux 容器平台

在 Linux 環境中，容器管理工具，例如 Docker 內建上更細微的一組容器工具- [runc](https://github.com/opencontainers/runc)和[containerd](https://containerd.io/)。

![在 Linux 上的 docker 架構](media/docker-on-linux.png)

`runc` 是建立和執行[OCI 容器執行階段規格](https://github.com/opencontainers/runtime-spec)根據容器的 Linux 命令列工具。

`containerd` 是容器生命週期管理下載並解壓縮容器執行及監督的情況下透過的容器映像精靈。

在 Windows 中，我們所花不同的方法。  當我們開始運作來支援 Windows 容器的 Docker 使用時，我們建置直接在 HCS （主機運算服務）。  [此部落格文章](https://blogs.technet.microsoft.com/virtualization/2017/01/27/introducing-the-host-compute-service-hcs/)是完整的為什麼我們建置 HCS 和為什麼我們所花這種方式容器一開始的相關資訊。

![在 Windows 上的初始 Docker 引擎架構](media/hcs.png)

到目前為止，Docker 仍然會直接將 HCS 呼叫。 往後，不過，容器的管理工具擴展到包含 Windows 容器和 Windows 容器主機無法呼叫到 containerd 和 runhcs 它們呼叫 containerd 和 linux runc 上的方式。

## <a name="runhcs"></a>runhcs

RunHCS 是 runc 的分支。  Runc，例如 runhcs 是命令列用戶端來執行根據開放容器計劃 (OCI) 格式的已封裝的應用程式且符合的開放容器計劃規格的實作。  

功能性 runc 和 runhcs 之間的差異包括：

* 在 Windows 上執行 runhcs
* runhcs 可以執行 Windows 和 Linux [HYPER-V 容器](../manage-containers/hyperv-container.md)除了 Windows 處理程序容器。

**使用方式：**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` 是您在您剛開始的容器執行個體的名稱。 名稱必須是在您的容器主機上唯一的。

在套件組合的目錄 (使用`-b bundle`) 是選擇性的。  
如同 runc，容器是使用套件組合來設定。 容器的套件組合是利用容器的 OCI 規格檔案，目錄 」 config.json 」。  預設值為"bundle"是目前的目錄。

OCI 規格檔案、 「 config.json 」，具備正確執行的兩個欄位：

1. 容器的可用空間路徑
1. 容器的層目錄路徑

Runhcs 中可用的容器命令包括：

* 工具來建立及執行容器
  * **執行**會建立並執行容器
  * **建立**建立一個容器

* 工具來管理在容器中執行的處理程序：
  * **開始**建立容器中執行的使用者定義的程序
  * **exec**執行新的處理序在容器內
  * **暫停**暫停暫停容器內的所有處理程序
  * **恢復**繼續先前已被暫停所有處理程序
  * **ps** ps 會顯示在容器內執行的處理程序

* 工具來管理容器的狀態
  * **狀態**會輸出為容器的狀態
  * **終止**會傳送指定的訊號 (預設值： SIGTERM) 到容器的初始化程序
  * **刪除**刪除保有常用於分離的容器的容器的任何資源

可以被視為多容器的唯一命令是**清單**。  它會列出執行中 （或暫停） 啟動的指定根 runhcs 的容器。

## <a name="containerdcri"></a>containerd/cri

> !注意： CRI 支援僅適用於 Server 2019 Windows 10 1809年及更新版本。

[CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) （容器執行階段介面） 而 OCI 規格定義單一容器，描述中共用的沙箱 workload(s) 為容器環境稱為 pod。  Pod 可以包含一或多個容器工作負載。  Pod 讓容器協調器，像是 Kubernetes 和服務網狀架構網格處理分組應該是含有一些共用的資源，例如記憶體和 vNETs 在相同主機的工作負載。

CRI 規格的連結：

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) Pod 規格
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47)工作負載規格

![Containerd 為基礎的容器環境](media/containerd-platform.png)

雖然 runHCS 和 containerd 可以管理 Server 2016 或更新版本的任何 Windows 系統上，支援 Pod （容器既有的群組） 需要 Windows 容器工具重大變更。  CRI 支援是在 Windows Server 2019/Windows 10 1809年可用和更新版本。

## <a name="hcs"></a>HCS

我們有 GitHub 上的兩個包裝函式可用 HCS 介面。 因為 HCS C API，包裝函式讓您輕鬆呼叫 HCS 從較高的層級語言。  

### <a name="hcsshim"></a>HCSShim

HCSShim 以移至撰寫，它是 runhcs 的基礎。
抓取 AppVeyor 的最新版本或建置它。

查看它在[GitHub](https://github.com/microsoft/hcsshim)中。

### <a name="dotnet-computevirtualization"></a>dotnet computevirtualization

> !請注意，這是參考實作-開發人員/測試只會使用它。

dotnet computevirtualization 是 C# 的包裝函式 HCS。

快來一探[github](https://github.com/microsoft/dotnet-computevirtualization)。

如果您想要使用 HCS （直接或透過包裝函式），或您想要讓鏽/Haskell/InsertYourLanguage 包裝函式來 HCS，請保持註解。

如 HCS 深入查看，請觀看[John Stark DockerCon 簡報](https://www.youtube.com/watch?v=85nCF5S8Qok)。