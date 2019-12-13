---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入門
keywords: docker、容器、LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909578"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

本練習將逐步解說如何在 Windows 10 上建立和執行 Linux 容器。

在本快速入門中，您將會完成：

1. 安裝 Docker Desktop
2. 在 Windows 上使用 Linux 容器執行簡單的 Linux 容器（LCOW）

此快速入門專用於 Windows 10。 您可以在此頁面左側的目錄中找到其他的快速入門檔。

## <a name="prerequisites"></a>必要條件

請確認您符合下列需求：
- 一部執行 Windows 10 專業版、Windows 10 企業版或 Windows Server 2019 1809 或更新版本的實體電腦系統
- 請確定已啟用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

***Hyper-v 隔離：*** Windows 上的 Linux 容器需要 Windows 10 上的 Hyper-v 隔離，才能為開發人員提供適當的 Linux 核心來執行容器。 如需 Hyper-v 隔離的詳細資訊，請參閱[關於 Windows 容器](../about/index.md)頁面。

## <a name="install-docker-desktop"></a>安裝 Docker Desktop

下載[Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)並執行安裝程式（您將需要登入。 如果您還沒有帳戶，請建立一個。 如需[詳細的安裝指示](https://docs.docker.com/docker-for-windows/install)，請參閱 Docker 文件。

> 如果您已安裝 Docker，請確定您有18.02 版或更新版本，以支援 LCOW。 藉由執行 `docker -v` 或檢查*Docker*來進行檢查。

> [ *Docker > 設定*] 中的 [實驗性功能] 選項必須啟用，才能執行 LCOW 容器。

## <a name="run-your-first-lcow-container"></a>執行您的第一個 LCOW 容器

在此範例中，將會部署 BusyBox 容器。 首先，嘗試執行 ' Hello World ' BusyBox 映射。

```console
docker run --rm busybox echo hello_world
```

請注意，當 Docker 嘗試提取映射時，這會傳回錯誤。 之所以會發生這種情況，是因為 Dockers 需要指示詞透過 `--platform` 旗標來確認映射和主機作業系統是否符合適當的比對。 由於 Windows 容器模式中的預設平臺是 Windows，請新增 `--platform linux` 旗標來提取並執行容器。

```console
docker run --rm --platform linux busybox echo hello_world
```

一旦使用指示的平臺提取映射之後，就不再需要 `--platform` 旗標。 執行命令而不進行測試。

```console
docker run --rm busybox echo hello_world
```

執行 `docker images` 以傳回已安裝的映射清單。 在此情況下，是 Windows 和 Linux 映射。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 額外資訊：請參閱 Docker 在執行 LCOW 上的對應的相關[blog 文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [瞭解如何建立範例應用程式](./building-sample-app.md)
