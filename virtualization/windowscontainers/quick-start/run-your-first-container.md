---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入門
keywords: docker、容器、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909568"
---
# <a name="get-started-run-your-first-windows-container"></a>開始使用：執行您的第一個 Windows 容器

本主題說明在設定環境之後，如何執行您的第一個 Windows 容器，如[開始使用：準備適用于容器的 Windows](./set-up-environment.md)中所述。 若要執行容器，您必須先安裝基底映射，為您的容器提供基本的作業系統服務層級。 接著，您可以建立並執行以基底映射為基礎的容器映射。 如需詳細資訊，請參閱。

## <a name="install-a-container-base-image"></a>安裝容器基底映射

所有容器都是從容器映射建立。 Microsoft 提供數個入門映射（稱為基底映射）以供選擇（如需詳細資訊，請參閱[容器基底映射](../manage-containers/container-base-images.md)）。 此程式會提取（下載並安裝）輕量 Nano Server 基底映射。

1. 開啟 [命令提示字元] 視窗（例如內建的命令提示字元、PowerShell 或[Windows 終端](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)機），然後執行下列命令來下載並安裝基底映射：

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > 如果您看到錯誤訊息，指出 `no matching manifest for unknown in the manifest list entries`，請確定 Docker 未設定為執行 Linux 容器。

2. 映射下載完成之後，請在等候時閱讀[EULA](../images-eula.md) -藉由查詢本機 docker 映射存放庫來確認它是否存在於您的系統上。 執行命令 `docker images` 會傳回已安裝的映射清單。

   以下是顯示 Nano 伺服器映射的輸出範例。

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>執行 Windows 容器

在這個簡單的範例中，將會建立並部署「Hello World」容器映射。 為了獲得最佳體驗，請在提高許可權的命令提示字元視窗中執行這些命令（但不要使用 Windows PowerShell ISE-它不適用於容器的互動式會話，因為容器似乎會停止回應）。

1. 在 [命令提示字元] 視窗中輸入下列命令，從 `nanoserver` 映射啟動具有互動式會話的容器：

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. 啟動容器之後，[命令提示字元] 視窗會將內容變更為容器。 在容器內，我們將建立簡單的 ' Hello World ' 文字檔，然後輸入下列命令來結束容器：

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. 藉由執行[docker ps](https://docs.docker.com/engine/reference/commandline/ps/)命令，取得您剛結束之容器的容器識別碼：

   ```console
   docker ps -a
   ```

4. 建立新的「HelloWorld」映射，其中包含您所執行之第一個容器中的變更。 若要這麼做，請執行[docker commit](https://docs.docker.com/engine/reference/commandline/commit/)命令，以您的容器識別碼取代 `<containerid>`：

   ```console
   docker commit <containerid> helloworld
   ```

   完成之後，您的自訂映像中就會包含 'Hello World' 指令碼。 這可以透過[docker images](https://docs.docker.com/engine/reference/commandline/images/)命令來查看。

   ```console
   docker images
   ```

   以下是輸出的範例：

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 最後，使用[docker run](https://docs.docker.com/engine/reference/commandline/run/)命令搭配 `--rm` 參數執行新的容器，以在命令列（cmd.exe）停止後自動移除容器。

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   結果是已從 ' HelloWorld ' 映射建立容器，cmd.exe 的實例已在讀取檔案的容器中啟動，並將檔案內容輸出至 shell，然後容器已停止且已移除。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [瞭解如何容器化範例應用程式](./building-sample-app.md)
