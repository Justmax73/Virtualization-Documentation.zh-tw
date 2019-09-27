---
title: Windows 容器基底影像
description: Windows 容器基本影像及使用時機的概覽。
keywords: docker, 容器, 雜湊
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: f5dcaf4958828b1bcf31a96e5fb70eda0508eb96
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161745"
---
# <a name="container-base-images"></a>容器基底影像

Windows 提供可供使用者建立的四個容器基底影像。 每個基底影像都有不同的 Windows 作業系統風格，且具有不同的磁碟空間，並攜帶不同數量的 Windows API 集。

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>支援傳統的 .NET framework 應用程式。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano 伺服器</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>專為 .NET Core 應用程式建立。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>提供完整的 Windows API 集。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IoT 核心版</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>為 IoT 應用程式專門建立。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>影像探索

透過[Docker 中樞](https://hub.docker.com/_/microsoft-windows-base-os-images)可探索所有 Windows 容器基底影像。 Windows 容器基底影像本身是從[mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/)（Microsoft 容器 REGISTRY （mcr））提供。 這就是為什麼 Windows 容器基本影像的 pull 命令看起來像這樣：

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR 沒有自己的目錄體驗，而且是要支援現有的目錄，例如 Docker 中樞。 感謝您的 Azure 全球版佔用及結合 Azure CDN，MCR 可提供一致且快速的影像拉體驗。 Azure 客戶在 Azure 中執行工作負荷，受益于網路效能增強，以及與 MCR （Microsoft 容器影像的來源）、Azure Marketplace，以及在 Azure 中不斷整合的服務數（可提供）[容器] 做為部署套件格式。

## <a name="choosing-a-base-image"></a>選擇基本影像

如何選擇要建立的正確基礎圖像？ 對於大部分的使用者`Windows Server Core`來說`Nanoserver` ，就是最適合使用的影像。

### <a name="guidelines"></a>指導方針

 雖然您可以自由地以任何您想要的方式設定目標，但以下是協助您選擇的一些指導方針：

- **您的應用程式是否需要完整的 .NET 架構？** 如果此問題的答案是 [是]，您應該`Windows Server Core`進行目標。
- **您是根據 .NET Core 來建立 Windows 應用程式嗎？** 如果此問題的答案是 [是]，您應該`Nanoserver`進行目標。
- **您正在建立 IoT 應用程式嗎？** 如果此問題的答案是 [是]，您應該`IoT Core`進行目標。
- **Windows Server 核心容器影像缺少您的應用程式需要的相依性嗎？** 如果此問題的答案是 [是]，您應該嘗試進行`Windows`目標。 這個影像比其他基底影像大許多，但它會攜帶許多核心 Windows 文件庫（例如 GDI 文件庫）。

> [!TIP]
> 許多 Windows 使用者想要 containerize 在 .NET 中有相依性的應用程式。 除了此處所述的四個基本影像之外，Microsoft 還會發佈幾個 Windows 容器影像，這些影像是預先設定的最常見的 Microsoft 架構，例如[.net framework](https://hub.docker.com/_/microsoft-dotnet-framework)影像和[ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)影像。

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core vs Nanoserver

`Windows Server Core` 而且`Nanoserver`是最常見的基本影像。 這些影像之間的主要差異是，Nanoserver 有明顯較小的 API 表面。 Nanoserver 映射中不存在 PowerShell、WMI 及 Windows 服務堆疊。

Nanoserver 的建立是為了提供足夠的 API surface，以執行在 .NET core 或其他現代的開放來源架構上有相依性的 app。 隨著較小 APi 表面的平衡，Nanoserver 影像的磁碟空間大小會比其他 Windows 基底影像小得多。 請記住，您隨時都可以視需要在 Nano Server 之上新增層級。 如需範例，請參閱 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)。
