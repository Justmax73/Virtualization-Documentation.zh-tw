---
title: Windows 和 Windows 10 上的 Linux 容器
description: 容器部署快速入門
keywords: docker，容器 LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 926e5cd64053b5ea795bb2c75a231700aed443ca
ms.sourcegitcommit: f6457ee0635864e8e8bb07da43a6f76388ee3cd1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2019
ms.locfileid: "9734652"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

> [!div class="op_single_selector"]
> - [Windows 上的 Linux 容器](quick-start-windows-10-linux.md)
> - [在 Windows 上的 Windows 容器](quick-start-windows-10.md)

本練習將逐步引導建立和執行 Windows 10 上的 Linux 容器。

本快速入門中，您將會完成：

1. 安裝 Docker 桌面
2. 執行簡單的 Linux 容器上 Windows (LCOW) 所使用的 Linux 容器

此快速入門專用於 Windows 10。 在此頁面左側的目錄中，就可以找到其他的快速入門文件。

## <a name="prerequisites"></a>必要條件

請確定您符合下列需求：
- 一部執行 Windows 10 專業版、 Windows 10 企業版或 Windows Server 2019 版本 1809年或更新版本的實體電腦系統
- 請確定[HYPER-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)已啟用。

***HYPER-V 隔離：*** 在 Windows 上的 Linux 容器需要 Windows 10 上的 HYPER-V 隔離，才能為開發人員來執行容器適當的 Linux 核心提供。 更多關於 HYPER-V 隔離上可找到[關於 Windows 容器](../about/index.md)頁面。

## <a name="install-docker-desktop"></a>安裝 Docker 桌面

下載[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)並執行安裝程式 （您將需要登入。 建立帳戶如果沒有已經）。 如需[詳細的安裝指示](https://docs.docker.com/docker-for-windows/install)，請參閱 Docker 文件。

> 如果您已經安裝 Docker，請確定您有 18.02 或更新版本，才能支援 LCOW 版本。 執行檢查`docker -v`或檢查*有關 Docker*。

> 若要執行 LCOW 容器，就必須啟用*Docker 設定 > 精靈*中的 「 實驗性功能' 選項。

## <a name="run-your-first-lcow-container"></a>執行您的第一個 LCOW 容器

針對此範例中，將會部署 BusyBox 容器。 首先，嘗試執行 ' Hello World' BusyBox 映像。

```console
docker run --rm busybox echo hello_world
```

請注意，這會傳回錯誤，Docker 嘗試提取映像時。 這是因為 Dockers 需要的指示詞，透過`--platform`旗標，以確認映像和主機作業系統會適當地比。 因為 Windows 預設平台在 Windows 容器模式中的，新增`--platform linux`旗標，以提取並執行容器。

```console
docker run --rm --platform linux busybox echo hello_world
```

一旦映像已被提取與平台的指示，`--platform`旗標，不再需要。 執行測試這沒有它命令。

```console
docker run --rm busybox echo hello_world
```

執行`docker images`傳回已安裝的映像的清單。 在此情況下，Windows 和 Linux 映像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 獎勵︰ 在執行 LCOW 看到 Docker 的相對應的[部落格文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何建置範例應用程式](./building-sample-app.md)
