---
title: Windows 容器基底映射
description: 概述 Windows 容器基底映射，以及使用它們的時機。
keywords: docker, 容器, 雜湊
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909778"
---
# <a name="container-base-images"></a>容器基底映射

Windows 提供四個可供使用者建立的容器基底映射。 每個基底映射都是不同的 Windows 作業系統類別，具有不同的磁片使用量，並具有不同的 Windows API 集數量。

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
                        <p>支援傳統 .NET framework 應用程式。</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>針對 .NET Core 應用程式所建立。</p>
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
                        <p>專為 IoT 應用程式而打造。</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>影像探索

所有 Windows 容器基底映射都可透過[Docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images)探索。 Windows 容器基底映射本身是從[mcr.microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/)（Microsoft container REGISTRY，mcr）提供。 這就是為什麼 Windows 容器基底映射的提取命令看起來如下所示：

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR 沒有自己的目錄體驗，目的是要支援 Docker Hub 等現有的目錄。 由於 Azure 的全球使用量和 Azure CDN 的結合，MCR 提供了既一致又快速的映射提取體驗。 Azure 客戶在 Azure 中執行其工作負載，可受益于網路中的效能增強功能，以及與 MCR （Microsoft 容器映射的來源）緊密整合 Azure Marketplace，並在 Azure 中提供的擴充服務數目作為部署套件格式的容器。

## <a name="choosing-a-base-image"></a>選擇基底映射

如何選擇要建立的正確基底映射？ 對於大部分的使用者而言，`Windows Server Core` 和 `Nanoserver` 將是最適合使用的影像。

### <a name="guidelines"></a>指導方針

 雖然您可以自由地以您想要的任何影像為目標，但以下是一些指導方針，可協助您選擇：

- **您的應用程式是否需要完整的 .NET framework？** 如果這個問題的答案是肯定的，您應該以 `Windows Server Core`為目標。
- **您是否正在建立以 .NET Core 為基礎的 Windows 應用程式？** 如果這個問題的答案是肯定的，您應該以 `Nanoserver`為目標。
- **您正在建立 IoT 應用程式嗎？** 如果這個問題的答案是肯定的，您應該以 `IoT Core`為目標。
- **Windows Server Core 容器映射是否遺漏您的應用程式所需的相依性？** 如果這個問題的答案是肯定的，您應該嘗試以 `Windows`為目標。 此映射遠大於其他基底映射，但它會攜帶許多核心 Windows 程式庫（例如 GDI 程式庫）。
- **您是 Windows 測試人員嗎？** 如果是，您應該考慮使用影像的 insider 版本。 請參閱下方的「適用于 Windows 測試人員的基底映射」。

> [!TIP]
> 許多 Windows 使用者都想要容器化相依于 .NET 的應用程式。 除了此處所述的四個基底映射之外，Microsoft 也發行了數個已預先設定熱門 Microsoft framework 的 Windows 容器映射，例如[.net framework](https://hub.docker.com/_/microsoft-dotnet-framework)映射和[ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/)映射。

### <a name="base-images-for-windows-insiders"></a>適用于 Windows 測試人員的基礎映射

Microsoft 會提供每個容器基底映射的「insider」版本。 這些 insider 容器映射會在容器映射中包含最新且最棒的功能開發。 當您執行的主機是 insider 版本的 Windows （Windows 測試人員或 Windows Server Insider）時，最好使用這些映射。 您可以在 Docker Hub 上取得 insider 映射：

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

如需深入瞭解，[請參閱使用容器與 Windows 測試人員程式](../deploy-containers/insider-overview.md)。

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core 與 Nanoserver

`Windows Server Core` 和 `Nanoserver` 是要作為目標的最常見基底映射。 這些映射的主要差異在於，Nanoserver 有一個明顯較小的 API 介面。 PowerShell、WMI 和 Windows 服務堆疊不存在於 Nanoserver 映射中。

Nanoserver 的建立是為了提供足夠的 API 介面，以執行相依于 .NET core 或其他現代化開放原始碼架構的應用程式。 對於較小的 APi 介面，Nanoserver 映射的磁片使用量明顯小於其餘的 Windows 基底映射。 請記住，您隨時都可以視需要在 Nano Server 之上新增層級。 如需範例，請參閱 [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile)。
