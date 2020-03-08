---
title: 容器化 .NET Core 應用程式
description: 瞭解如何使用容器建立範例 .NET core 應用程式
keywords: Docker, 容器
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 587e8de5f0d593f92f6301c87bf68e08a8bbd839
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78854002"
---
# <a name="containerize-a-net-core-app"></a>容器化 .NET Core 應用程式

本主題說明如何在設定環境之後，將現有的範例 .NET 應用程式封裝為 Windows 容器，如[開始使用：準備適用于容器的 windows](set-up-environment.md)和執行您的第一個容器（如[執行您的第一個 Windows 容器](run-your-first-container.md)中所述）。

您也需要安裝在您電腦上的 Git 原始檔控制系統。 若要安裝它，請造訪[Git](https://git-scm.com/download)。

## <a name="clone-the-sample-code-from-github"></a>從 GitHub 複製範例程式碼

所有容器範例原始程式碼都會保留在名為 `windows-container-samples`之資料夾中的[虛擬化檔](https://github.com/MicrosoftDocs/Virtualization-Documentation)git 存放庫（稱為「儲存機制」）底下。

1. 開啟 PowerShell 會話，並將目錄變更為您要儲存此存放庫的資料夾。 （其他的命令提示字元視窗類型也可以運作，但我們的範例命令會使用 PowerShell）。
2. 將存放庫複製到您目前的工作目錄：

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 流覽至 `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` 底下找到的範例目錄，並使用下列命令建立 Dockerfile。

   [Dockerfile](https://docs.docker.com/engine/reference/builder/)就像是 makefile，它是告訴容器引擎如何建立容器映射的指示清單。

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>撰寫 Dockerfile

使用您喜歡的任何文字編輯器來開啟您剛才建立的 Dockerfile，然後新增下列內容：

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

讓我們逐行細分，並說明每個指示的作用。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

第一組程式碼行會宣告我們將使用哪個基本映像做為建置容器的基礎。 如果本機系統原本沒有這個映像，則 Docker 將自動嘗試擷取該映像。 `mcr.microsoft.com/dotnet/core/sdk:2.1` 隨附于已安裝的 .NET core 2.1 SDK 中，因此您可以建立以版本2.1 為目標的 ASP .NET core 專案。 下一個指令會將容器中的工作目錄變更為 `/app`，因此此動作後面的所有命令都會在此內容下執行。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

接下來，這些指示會將 .csproj 檔案複製到 `build-env` 容器的 `/app` 目錄中。 複製此檔案之後，.NET 會從該檔案讀取，然後再提取專案所需的所有相依性和工具。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

.NET 將所有相依性提取到 `build-env` 容器之後，下一個指令會將所有專案來源檔案複製到容器中。 接著，我們會告訴 .NET 以發行設定發行應用程式，並在中指定輸出路徑。

編譯應會成功。 現在，我們必須建立最終的映射。 

> [!TIP]
> 本快速入門會從來源建立 .NET core 專案。 建立容器映射時，最好_只_在容器映射中包含生產裝載和其相依性。 我們不想讓 .NET core SDK 包含在最終的映射中，因為我們只需要 .NET core 執行時間，因此 dockerfile 會撰寫成使用與 SDK 一起封裝的暫存容器，稱為 `build-env` 來建立應用程式。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

因為我們的應用程式是 ASP.NET，所以我們會指定包含此執行時間的映射。 然後，我們要將所有檔案從暫存容器的輸出目錄複製到最終容器中。 我們會將容器設定為在容器啟動時，以新的應用程式作為其進入點來執行

我們已撰寫 dockerfile 來執行_多階段組建_。 執行 dockerfile 時，它會使用暫存容器（`build-env`）搭配 .NET core 2.1 SDK 來建立範例應用程式，然後將輸出的二進位檔複製到僅包含 .NET core 2.1 執行時間的另一個容器，以便將最終容器的大小最小化。

## <a name="build-and-run-the-app"></a>建置和執行應用程式

撰寫 Dockerfile 之後，我們可以將 Docker 指向我們的 Dockerfile，並告訴它建立並執行我們的映射：

1. 在 [命令提示字元] 視窗中，流覽至 dockerfile 所在的目錄，然後執行[docker build](https://docs.docker.com/engine/reference/commandline/build/)命令以從 dockerfile 建立容器。

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 若要執行新建立的容器，請執行[docker run](https://docs.docker.com/engine/reference/commandline/run/)命令。

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   讓我們來仔細分析此命令：

   * `-d` 會告訴 Docker 執行執行容器「卸離」，表示沒有主控台連接到容器內的主控台。 容器會在背景中執行。 
   * `-p 5000:80` 會指示 Docker 將主機上的埠5000對應至容器中的埠80。 每個容器都會取得自己的 IP 位址。 根據預設，ASP .NET 會在埠80上接聽。 埠對應可讓我們移至對應埠上主機的 IP 位址，而 Docker 會將所有流量轉送至容器內的目的地埠。
   * `--name myapp` 會告訴 Docker 提供此容器一個方便查詢的名稱，而不需要查閱 Docker 在執行時間指派的 contaienr 識別碼。
   * `my-asp-app` 是我們想要 Docker 執行的映射。 這是 `docker build` 進程的高潮所產生的容器映射。

3. 開啟網頁瀏覽器，並流覽至 `http://localhost:5000` 以查看您的容器化應用程式，如下列螢幕擷取畫面所示：

   >![ASP.NET Core 網頁，從容器中的 localhost 執行](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>後續步驟

1. 下一步是使用 Azure Container Registry 將您的容器化 ASP.NET web 應用程式發佈至私人登錄。 這可讓您將它部署在您的組織中。

   > [!div class="nextstepaction"]
   > [建立私人容器登錄](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   當您進入將[容器映射推送至](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)登錄的區段時，請指定您剛封裝的 ASP.NET 應用程式的名稱（`my-asp-app`）和您的容器登錄（例如： `contoso-container-registry`）：

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   若要查看更多應用程式範例和其相關聯的 dockerfile，請參閱[其他容器範例](../samples.md)。

2. 將應用程式發佈至容器登錄之後，下一步就是將應用程式部署到您使用 Azure Kubernetes Service 建立的 Kubernetes 叢集。

   > [!div class="nextstepaction"]
   > [建立 Kubernetes 叢集](https://docs.microsoft.com/azure/aks/windows-container-cli)
