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
ms.openlocfilehash: 07f5929505226a50a161b4ae7df5669c2ad89d83
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380422"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10 上的 Windows 容器

> [!div class="op_single_selector"]
> - [Windows 上的 Linux 容器](quick-start-windows-10-linux.md)
> - [在 Windows 上的 Windows 容器](quick-start-windows-10.md)

本練習將逐步引導建立和執行 Windows 10 上的 Windows 容器。

本快速入門中，您將會完成：

1. 安裝 Docker for Windows
2. 執行簡單的 Windows 容器

此快速入門專用於 Windows 10。 在此頁面左側的目錄中，就可以找到其他的快速入門文件。

## <a name="prerequisites"></a>必要條件
請確定您符合下列需求：
- 執行 Windows 10 專業版或企業版使用年度更新版 （版本 1607年） 或更新版本的一個實體電腦系統。 
- 請確定[HYPER-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)已啟用。

***HYPER-V 隔離：*** Windows Server 容器需要 Windows 10 上的 HYPER-V 隔離，才能為開發人員的相同核心版本和設定將用在生產環境中提供，更多關於 HYPER-V 隔離上可找到[關於 Windows 容器](../about/index.md)頁面。

> [!NOTE]
> 在 Windows 10 月更新 2018年版本中，我們不會再不允許使用者從在處理序隔離模式中執行 Windows 容器在 Windows 10 企業版或專業版上的開發人員/測試用途。 請參閱以了解更多[常見問題集](../about/faq.md)。

## <a name="install-docker-for-windows"></a>安裝 Docker for Windows

下載[Docker for Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) ，並執行安裝程式 （您將需要登入。 建立帳戶如果沒有已經）。 如需[詳細的安裝指示](https://docs.docker.com/docker-for-windows/install)，請參閱 Docker 文件。

## <a name="switch-to-windows-containers"></a>切換到 Windows 容器

安裝完成後，Docker for Windows 預設會執行 Linux 容器。 切換到 Windows 容器，請使用 Docker 系統匣功能表，或是在 PowerShell 中執行下列命令提示中：

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
> 請閱讀 Windows 容器 OS 映像[授權條款](../images-eula.md)。

## <a name="run-your-first-windows-container"></a>執行您的第一個 Windows 容器

針對此簡單範例，將會建立和部署 ' Hello World' 容器映像。 請在提高權限的 Windows CMD 殼層或 PowerShell 中執行這些命令，以取得最佳體驗。

> Windows PowerShell ISE 不適用於容器的互動式工作階段。 即使容器為執行中，也會顯示為停止回應。

首先，請利用互動式工作階段，從 `nanoserver` 映像啟動容器。 容器啟動之後，會在容器中顯示命令殼層。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

在容器內，我們會建立簡單的 ' Hello World' 文字檔案。

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

結果`docker run`命令是從 'HelloWorld' 映像建立 HYPER-V 隔離下執行容器，cmd 的執行個體已啟動容器中，且執行我們的檔案 （輸出傳送到殼層），然後在容器的讀取停止並移除。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何建置範例應用程式](./building-sample-app.md)