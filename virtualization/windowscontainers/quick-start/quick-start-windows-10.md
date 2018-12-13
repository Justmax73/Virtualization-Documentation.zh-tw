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
ms.openlocfilehash: 4202908f8797a2b98ab657c45cd9a6b33191bd6f
ms.sourcegitcommit: 9dfef8d261f4650f47e8137a029e893ed4433a86
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/09/2018
ms.locfileid: "6224897"
---
# <a name="windows-and-linux-containers-on-windows-10"></a>Windows 和 Windows 10 上的 Linux 容器

本練習將逐步引導建立和執行 Windows 10 上的 Windows 和 Linux 容器。 完成之後，您將會：

1. 已安裝的 Docker for Windows
2. 執行簡單的 Windows 容器
3. 執行簡單的 Linux 容器上 Windows (LCOW) 所使用的 Linux 容器

此快速入門專用於 Windows 10。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

***HYPER-V 隔離：*** Windows Server 容器需要 Windows 10 上的 HYPER-V 隔離，才能為開發人員的相同核心版本和設定將用在生產環境中提供，更多關於 HYPER-V 隔離上可找到[關於 Windows 容器](../about/index.md)頁面。

**先決條件：**

- 執行 Windows 10 Fall Creators Update （版本 1709年） 或更新版本 （專業版或企業）[可以執行 HYPER-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)的一個實體電腦系統

> 如果您不想要執行 LCOW 本教學課程中，Windows 容器無法執行 Windows 10 年度更新版 （版本 1607年） 或更新版本。

## <a name="1-install-docker-for-windows"></a>1. 安裝 Docker for Windows

[下載 Docker for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) ，然後執行安裝程式 （您將會需要登入。 建立帳戶如果您沒有一個已經）。 如需[詳細的安裝指示](https://docs.docker.com/docker-for-windows/install)，請參閱 Docker 文件。

> 如果您已經安裝 Docker，請確定您有 18.02 或更新版本，才能支援 LCOW 版本。 執行檢查`docker -v`或檢查*有關 Docker*。

> 實驗性功能' 選項*Docker 設定 > 精靈*必須啟用，才能執行 LCOW 容器。

## <a name="2-switch-to-windows-containers"></a>2. 切換到 Windows 容器

安裝完成後，Docker for Windows 預設會執行 Linux 容器。 若要切換到 Windows 容器，請使用 Docker 系統匣功能表，或是在 PowerShell 提示中執行下列命令：`& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`。

![](./media/docker-for-win-switch.png)
> 請注意，Windows 容器模式允許適用於 Windows 容器除了 LCOW 容器。

## <a name="3-install-base-container-images"></a>3. 安裝基礎容器映像

Windows 容器是從基本映像建置而來。 下列命令會提取 Nano Server 基本映像。

```
docker pull microsoft/nanoserver
```

提取映像之後再執行 `docker images`，將會傳回已安裝的映像清單，在此案例中為 Nano Server 映像。

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> 請參閱「Windows 容器 OS 映像授權條款」，位於這裡：[授權條款](../images-eula.md)。

## <a name="4-run-your-first-windows-container"></a>4.執行您的第一個 Windows 容器

在這個簡單的範例中，將會建立及部署 'Hello World' 容器映像。 請在提高權限的 Windows CMD 殼層或 PowerShell 中執行這些命令，以取得最佳體驗。
> Windows PowerShell ISE 不適用於容器的互動式工作階段。 即使容器為執行中，也會顯示為停止回應。

首先，請利用互動式工作階段，從 `nanoserver` 映像啟動容器。 容器啟動之後，會在容器中顯示命令殼層。  

```
docker run -it microsoft/nanoserver cmd
```

我們將在容器中建立簡單的 'Hello World' 指令碼。

```
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

完成之後，請結束容器。

```
exit
```

接下來，請使用修改後的容器建立新的容器映像。 如需容器的清單，請執行下列命令並記下容器的識別碼。

```
docker ps -a
```

請執行下列命令來建立新的 'HelloWorld' 映像。 以您的容器識別碼取代 <containerid>。

```
docker commit <containerid> helloworld
```

完成之後，您的自訂映像中就會包含 'Hello World' 指令碼。 您可以使用下列命令確認。

```
docker images
```

最後，若要執行該容器，請使用 `docker run` 命令。

```
docker run --rm helloworld powershell c:\helloworld.ps1
```

`docker run` 命令的結果，在從 'HelloWorld’ 映像建立 Hyper-V 容器，接著執行範例 'HelloWorld’ 指令碼 (將輸出傳送到殼層)，然後停止容器並加以移除。
後續的 Windows 10 及容器快速啟動將會深入探究在 Windows 10 上的容器中建立及部署應用程式。

## <a name="run-your-first-lcow-container"></a>執行您的第一個 LCOW 容器

針對此範例中，將會部署 BusyBox 容器。 首先，嘗試執行 ' Hello World' BusyBox 映像。

```
docker run --rm busybox echo hello_world
```

請注意，這會傳回錯誤，Docker 會嘗試提取映像時。 這是因為 Dockers 需要的指示詞，透過`--platform`旗標，以確認映像和主機作業系統會適當地比。 因為 Windows 預設平台在 Windows 容器模式中的，新增`--platform linux`旗標，以提取並執行容器。

```
docker run --rm --platform linux busybox echo hello_world
```

一旦映像已被提取與平台的指示，`--platform`旗標，不再需要。 執行測試這沒有它命令。

```
docker run --rm busybox echo hello_world
```

執行`docker images`傳回已安裝的映像的清單。 在此情況下，Windows 與 Linux 映像。

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>後續步驟

獎勵︰ 上執行 LCOW 看到 Docker 的對應[部落格文章](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)

繼續進行下一個教學課程以查看[建置範例應用程式](./building-sample-app.md)的範例
