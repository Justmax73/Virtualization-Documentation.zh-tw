---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入門
keywords: docker、樹枝、LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 11e3c9f93400af2c1bc2d84136d512ac91cf4477
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999085"
---
# <a name="linux-containers-on-windows-10"></a>Windows 10 上的 Linux 容器

> [!div class="op_single_selector"]
> - [Windows 上的 Linux 容器](quick-start-windows-10-linux.md)
> - [Windows 上的 windows 容器](quick-start-windows-10.md)

此練習將逐步引導您在 Windows 10 上建立及執行 Linux 容器。

在這個快速入門中, 您將完成下列作業:

1. 安裝 Docker 桌面
2. 在 Windows 上使用 Linux 樹枝執行簡單的 Linux 容器 (LCOW)

此快速入門專用於 Windows 10。 您可以在此頁面左側的目錄中找到其他快速入門檔。

## <a name="prerequisites"></a>必要條件

請確定您符合下列需求:
- 一台執行 Windows 10 專業版、Windows 10 企業版或 Windows Server 2019 版本1809或更新版本的物理電腦系統
- 請確定已啟用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

***Hyper-v 隔離:*** Windows 上的 Linux 容器需要 Windows 10 上的 Hyper-v 隔離, 才能為開發人員提供適當的 Linux 內核來執行容器。 您可以在 [[關於 Windows 容器](../about/index.md)] 頁面上找到有關 hyper-v 隔離的詳細資訊。

## <a name="install-docker-desktop"></a>安裝 Docker 桌面

下載[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)並執行安裝程式 (您將需要登入)。 如果您還沒有帳戶, 請建立一個帳戶。 如需[詳細的安裝指示](https://docs.docker.com/docker-for-windows/install)，請參閱 Docker 文件。

> 如果您已經安裝了 Docker, 請確定您有版本18.02 或更新版本, 以支援 LCOW。 執行`docker -v`或檢查是否有*Docker*, 以進行檢查。

> 在 Docker 設定中的「實驗功能」選項, *> 守護*程式必須啟用, 才能執行 LCOW 容器。

## <a name="run-your-first-lcow-container"></a>執行您的第一個 LCOW 容器

在這個範例中, 將會部署 BusyBox 容器。 首先, 嘗試執行「Hello World」 BusyBox 影像。

```console
docker run --rm busybox echo hello_world
```

請注意, 當 Docker 嘗試拉入影像時, 會傳回錯誤。 發生這種情況是因為 Dockers 需要透過`--platform`標誌的指令來確認影像與主機作業系統已適當地相符。 因為 Windows 容器模式中的預設平臺是 Windows, 所以請`--platform linux`新增一個標誌來提取並執行容器。

```console
docker run --rm --platform linux busybox echo hello_world
```

在以指定的平臺提取圖像之後, 就不再需要`--platform`該標誌了。 執行命令而不進行測試。

```console
docker run --rm busybox echo hello_world
```

[ `docker images`執行] 以傳回已安裝影像的清單。 在這種情況下, 就是 Windows 和 Linux 的影像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> 附加版: 請參閱在執行 LCOW 上的 Docker 對應的[博客文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [瞭解如何建立範例應用程式](./building-sample-app.md)
