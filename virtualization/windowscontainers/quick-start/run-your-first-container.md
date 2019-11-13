---
title: Windows 10 上的 windows 和 Linux 容器
description: 容器部署快速入門
keywords: docker、樹枝、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288126"
---
# <a name="get-started-run-your-first-windows-container"></a>快速入門：執行您的第一個 Windows 容器

本主題說明如何在設定您的環境之後執行您的第一個 Windows 容器，如 [開始使用] 中所述[： [準備適用于容器的 Windows](./set-up-environment.md)]。 若要執行容器，您必須先安裝基本映射，以將作業系統服務的基本層級提供給您的容器。 接著，您會根據基本影像建立並執行容器影像。 如需詳細資訊，請繼續閱讀。

## <a name="install-a-container-base-image"></a>安裝容器基底影像

所有容器都是從容器影像建立。 Microsoft 提供數個 starter 影像（稱為基本影像）來選擇 [from] （如需詳細資訊，請參閱[容器基礎圖像](../manage-containers/container-base-images.md)）。 此程式會提取（下載及安裝）羽量級 Nano Server 基礎映射。

1. 開啟命令提示字元視窗（例如內建的命令提示字元、PowerShell 或[Windows 終端](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)），然後執行下列命令以下載並安裝基本影像：

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > 如果您看到錯誤訊息`no matching manifest for unknown in the manifest list entries`，請確認 Docker 沒有設定為執行 Linux 容器。

2. 完成下載後，請在您等待時閱讀[EULA](../images-eula.md) ，方法是查詢您的本機 docker 影像儲存庫，以驗證您的系統是否存在該檔案。 執行命令`docker images`會傳回已安裝影像的清單。

   以下是顯示 Nano Server 映射之輸出的範例。

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>執行 Windows 容器

在這個簡單的範例中，將會建立並部署「Hello World」容器影像。 為了獲得最佳體驗，請在提升許可權的命令提示字元視窗中執行這些命令（但不要使用 Windows PowerShell ISE），因為容器顯示為 [掛起]，所以它不適用於樹枝的互動式會話。

1. 在命令提示字元視窗中輸入下列命令`nanoserver` ，以從影像啟動含有互動式會話的容器：

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. 容器啟動後，[命令提示字元] 視窗會變更容器的內容。 在容器中，我們會建立一個簡單的「Hello World」文字檔，然後輸入下列命令以結束容器：

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. 透過執行[docker ps](https://docs.docker.com/engine/reference/commandline/ps/)命令，取得您剛剛退出之容器的容器識別碼：

   ```console
   docker ps -a
   ```

4. 建立新的 [HelloWorld] 影像，其中包含您執行的第一個容器中的變更。 若要這樣做，請執行[docker commit](https://docs.docker.com/engine/reference/commandline/commit/)命令， `<containerid>`並以您的容器識別碼取代：

   ```console
   docker commit <containerid> helloworld
   ```

   完成之後，您的自訂映像中就會包含 'Hello World' 指令碼。 您可以在[docker 影像](https://docs.docker.com/engine/reference/commandline/images/)命令中看到這種情況。

   ```console
   docker images
   ```

   以下是輸出的範例：

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 最後，使用[docker [執行](https://docs.docker.com/engine/reference/commandline/run/)] 命令以及可在命令列（ `--rm` cmd.exe）停止之後自動移除容器的參數，來執行新的容器。

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   結果是，容器是從「HelloWorld」影像建立，在讀取檔案並將檔案內容輸出至 shell 的容器中啟動了 cmd.exe 的實例，而容器則會停止並移除。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [瞭解如何 containerize 範例應用程式](./building-sample-app.md)
