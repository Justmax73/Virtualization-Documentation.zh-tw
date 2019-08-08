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
ms.openlocfilehash: 094d7adde67b243a4bcadb1580e239d2175562c7
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998180"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10 上的 Windows 容器

> [!div class="op_single_selector"]
> - [Windows 上的 Linux 容器](quick-start-windows-10-linux.md)
> - [Windows 上的 windows 容器](quick-start-windows-10.md)

此練習將逐步引導您在 Windows 10 上建立及執行 Windows 容器。

在這個快速入門中, 您將完成下列作業:

1. 安裝 Docker 桌面
2. 執行簡單的 Windows 容器

此快速入門專用於 Windows 10。 您可以在此頁面左側的目錄中找到其他快速入門檔。

## <a name="prerequisites"></a>必要條件
請確定您符合下列需求:
- 一種執行 Windows 10 專業版或企業版的物理電腦系統, 並含周年紀念更新 (版本 1607) 或更新版本。 
- 請確定已啟用[hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) 。

***Hyper-v 隔離:*** Windows Server 容器在 Windows 10 上需要 Hyper-v 隔離功能, 以便為開發人員提供將在生產中使用的相同內核版本和設定, 如需有關 Hyper-v 隔離的詳細資訊, 請參閱[關於 Windows 容器](../about/index.md)頁面。

> [!NOTE]
> 在 Windows 10 月更新2018的版本中, 我們不再允許使用者在 Windows 10 企業版或專業版 (適用于開發人員/測試目的) 上, 以進程隔離模式執行 Windows 容器。 如需深入瞭解, 請參閱[常見問題](../about/faq.md)。

## <a name="install-docker-desktop"></a>安裝 Docker 桌面

下載[Docker 桌面](https://store.docker.com/editions/community/docker-ce-desktop-windows)並執行安裝程式 (您將需要登入)。 如果您還沒有帳戶, 請建立一個帳戶。 如需[詳細的安裝指示](https://docs.docker.com/docker-for-windows/install)，請參閱 Docker 文件。

## <a name="switch-to-windows-containers"></a>切換到 Windows 容器

安裝 Docker 桌上型電腦預設為執行 Linux 容器。 使用 Docker 託盤功能表, 或在 PowerShell 提示中執行下列命令, 以切換到 Windows 容器:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>安裝基礎容器映像

Windows 容器是從基本映像建置而來。 下列命令會提取 Nano Server 基本映像。

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

提取映像之後再執行 `docker images`，將會傳回已安裝的映像清單，在此案例中為 Nano Server 映像。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> 請閱讀 Windows 容器作業系統影像[EULA](../images-eula.md)。

## <a name="run-your-first-windows-container"></a>執行您的第一個 Windows 容器

在這個簡單的範例中, 將會建立並部署「Hello World」容器影像。 請在提高權限的 Windows CMD 殼層或 PowerShell 中執行這些命令，以取得最佳體驗。

> Windows PowerShell ISE 不適用於容器的互動式工作階段。 即使容器為執行中，也會顯示為停止回應。

首先，請利用互動式工作階段，從 `nanoserver` 映像啟動容器。 容器啟動之後，會在容器中顯示命令殼層。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

在容器中, 我們會建立一個簡單的「Hello World」文字檔。

```cmd
echo "Hello World!" > Hello.txt
```   

完成之後，請結束容器。

```cmd
exit
```

接下來，請使用修改後的容器建立新的容器映像。 如需容器的清單，請執行下列命令並記下容器的識別碼。

```console
docker ps -a
```

請執行下列命令來建立新的 'HelloWorld' 映像。 以您的容器識別碼取代 <containerid>。

```console
docker commit <containerid> helloworld
```

完成之後，您的自訂映像中就會包含 'Hello World' 指令碼。 您可以使用下列命令確認。

```console
docker images
```

最後，若要執行該容器，請使用 `docker run` 命令。

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

`docker run`命令的結果是, 在 hyper-v 隔離下執行的容器是從「HelloWorld」影像所建立, 在容器中啟動了 cmd 的一個實例, 並執行了檔案的讀取 (向 shell 輸出, 然後是容器)已停止並移除。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [瞭解如何建立範例應用程式](./building-sample-app.md)
