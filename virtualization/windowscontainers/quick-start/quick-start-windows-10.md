---
title: Windows 10 上的 Windows 容器
description: 容器部署快速入門
keywords: docker, 容器
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: d831b5950d84c9f82e2a4874827b2ffb107ad50e
ms.sourcegitcommit: 4090d158dd3573ea90799de5b014c131a206b000
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/07/2018
ms.locfileid: "6121578"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10 上的 Windows 容器

本練習將逐步引導您了解 Windows 10 專業版或企業版 (Anniversary Edition) 上 Windows 容器功能的基本部署與使用。 完成之後，您將會安裝好 Docker for Windows，並執行簡單的容器。 如果您需要熟悉一下容器，可以在[關於容器](../about/index.md)中找到這項資訊。

此快速入門專用於 Windows 10。 在此頁面左側的目錄中，可以找到其他的快速入門文件。

***Hyper-V 隔離：*** Windows Server 容器需要 Windows 10 的 Hyper-V 隔離，才能為開發人員提供用於生產環境的相同核心版本和設定。您可以在[關於 Windows 容器](../about/index.md)頁面上找到詳細資訊。

**必要條件：**

- 執行 Windows 10 Anniversary Edition 或 Creators Update (專業版或企業版) 的一個實體電腦系統。   
- 本快速入門可以在 Windows 10 虛擬機器上執行，但是需要啟用巢狀虛擬化。 在[巢狀虛擬化指南](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)中可以找到詳細資訊。

> 您必須安裝 Windows 容器的重大更新才能運作。
> 若要檢查您的作業系統版本，請執行 `winver.exe`，然後比較顯示的版本與 [Windows 10 更新記錄](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)。

> 請先確認您有 14393.222 或更新版本再繼續。  此版本對應至 Windows 10 版本 1607 開始，因此應該完全支援任何版本 1607年上方。

## <a name="1-install-docker-for-windows"></a>1. 安裝 Docker for Windows

[下載 Docker for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) ，然後執行安裝程式 （您將會需要登入。 建立帳戶如果您沒有一個已經）。 如需[詳細的安裝指示](https://docs.docker.com/docker-for-windows/install)，請參閱 Docker 文件。

## <a name="2-switch-to-windows-containers"></a>2. 切換到 Windows 容器

安裝完成後，Docker for Windows 預設會執行 Linux 容器。 若要切換到 Windows 容器，請使用 Docker 系統匣功能表，或是在 PowerShell 提示中執行下列命令：`& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`。

![](./media/docker-for-win-switch.png)

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

## <a name="4-run-your-first-container"></a>4. 執行您的第一個容器

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

## <a name="next-steps"></a>後續步驟

繼續進行下一個教學課程以查看[建置範例應用程式](./building-sample-app.md)的範例
