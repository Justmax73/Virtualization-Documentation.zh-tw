---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入門
keywords: docker、樹枝、LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 3d651a4a68acefa25f1b647b1b33618bbfb91ae9
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129362"
---
# <a name="get-started-run-your-first-container"></a>快速入門：執行您的第一個容器

在[前一段](./set-up-environment.md)中，我們設定了執行容器的環境。 此練習將說明如何拉入容器影像並執行它。

## <a name="install-container-base-image"></a>安裝容器基底影像

所有容器都是以`container images`具現化的。 Microsoft 提供數個 [starter] 影像（ `base images`稱為）來選擇。 下列命令會提取 Nano Server 基本映像。

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!TIP]
> 如果您看到錯誤訊息指出`no matching manifest for unknown in the manifest list entries`，請確定 Docker 未設定為執行 Linux 容器。

在拉入影像之後，您可以查詢本機 docker 影像儲存庫，以驗證您電腦上的現有內容。 執行命令`docker images`時，會傳回已安裝影像的清單（在此案例中為 Nano Server 映射）。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> 請閱讀 Windows 容器作業系統影像[EULA](../images-eula.md)。

## <a name="run-your-first-windows-container"></a>執行您的第一個 Windows 容器

在這個簡單的範例中，將會建立並部署「Hello World」容器影像。 若要獲得最佳體驗，請在提升許可權的 Windows CMD 命令介面或 PowerShell 中執行這些命令。

> Windows PowerShell ISE 不適用於容器的互動式工作階段。 即使容器為執行中，也會顯示為停止回應。

首先，請利用互動式工作階段，從 `nanoserver` 映像啟動容器。 容器啟動後，您會在容器中顯示命令 shell。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

在容器中，我們會建立一個簡單的「Hello World」文字檔。

```cmd
echo "Hello World!" > Hello.txt
```   

完成之後，請結束容器。

```cmd
exit
```

從修改過的容器中建立新的容器影像。 若要查看正在執行或已退出的容器清單，請執行下列動作並記下容器識別碼。

```console
docker ps -a
```

請執行下列命令來建立新的 'HelloWorld' 映像。 以您的容器識別碼取代 `<containerid>`。

```console
docker commit <containerid> helloworld
```

完成之後，您的自訂映像中就會包含 'Hello World' 指令碼。 您可以使用下列命令確認。

```console
docker images
```

最後，使用`docker run`命令來執行容器。

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

`docker run`命令的結果是，容器是從「HelloWorld」影像建立，在容器中啟動 cmd 的實例，並執行檔案的讀取（輸出回顯至 shell），然後停止並移除容器。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [瞭解如何 containerize 範例應用程式](./building-sample-app.md)
