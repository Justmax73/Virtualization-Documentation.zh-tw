---
title: Containerize .NET Core 應用程式
description: 瞭解如何使用容器建立範例 .NET 核心應用程式
keywords: Docker, 容器
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288066"
---
# <a name="containerize-a-net-core-app"></a>Containerize .NET Core 應用程式

本主題說明如何將現有的範例 .NET app 封裝為 Windows 容器，請在按照「開始使用」中所述的步驟設定您的環境後[： [準備適用于容器的視窗](set-up-environment.md)]，並執行您的第一個容器，如[執行您的第一個 Windows 容器](run-your-first-container.md)中所述。

您也需要在電腦上安裝 Git 來源控制系統。 若要安裝它，請造訪[Git](https://git-scm.com/download)。

## <a name="clone-the-sample-code-from-github"></a>從 GitHub 克隆範例程式碼

在名`windows-container-samples`為的資料夾中，所有容器範例原始程式碼都保留在[虛擬化-檔](https://github.com/MicrosoftDocs/Virtualization-Documentation)git 儲存庫（非正式稱為儲存庫）底下。

1. 開啟 PowerShell 會話，然後將目錄變更為您要儲存資料庫的資料夾。 （其他命令提示字元視窗類型也能正常運作，但我們的範例命令會使用 PowerShell。）
2. 將存放庫克隆到您目前的工作目錄：

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 使用下列命令流覽至 [建立`Virtualization-Documentation\windows-container-samples\asp-net-getting-started` Dockerfile] 底下所找到的範例目錄。

   [Dockerfile](https://docs.docker.com/engine/reference/builder/)就像是 makefile 一樣，它是一份指示容器引擎如何建立容器影像的指示。

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>撰寫 Dockerfile

開啟您剛剛使用您喜歡的任何文字編輯器建立的 Dockerfile，然後新增下列內容：

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

讓我們逐行劃分並說明每個指示的功能。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

第一組程式碼行會宣告我們將使用哪個基本映像做為建置容器的基礎。 如果本機系統原本沒有這個映像，則 Docker 將自動嘗試擷取該映像。 `mcr.microsoft.com/dotnet/core/sdk:2.1`隨附于已安裝 .net CORE 2.1 SDK 的封裝，所以它是由建立以版本2.1 為目標的 ASP .net 核心專案所產生的工作。 下一個指令會將`/app`容器中的工作目錄變更成，因此在這個內容下的所有命令都要執行。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

接著，這些指示會將 .csproj 檔案複製到`build-env`容器的`/app`目錄中。 複製此檔案之後，.NET 就會從該檔案中讀取，然後取出並提取專案所需的所有相依性與工具。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

一旦 .NET 將所有相依性移入`build-env`容器，下一個指令就會將所有專案來源檔案複製到容器中。 接著，我們會告知 .NET 以發行設定發佈應用程式，並在中指定輸出路徑。

編譯應該會成功。 現在，我們必須建立最終的影像。 

> [!TIP]
> 此快速入門從來源建立 .NET 核心專案。 建立容器影像時，最好_只_在容器影像中包含生產負載及其相依性。 我們不想在我們的最終影像中包含 .NET core SDK，因為我們只需要 .NET 核心執行時間，所以 dockerfile 是使用與用`build-env`來建立應用程式的 SDK 封裝的臨時容器一起撰寫。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

因為我們的應用程式是 ASP.NET，所以我們會指定包含此執行時間的影像。 然後，我們要將所有檔案從暫存容器的輸出目錄複製到最終容器中。 我們將容器設定為在容器啟動時使用新的應用程式作為進入點來執行

我們已撰寫 dockerfile 來執行_多階段組建_。 當 dockerfile 執行時，它將會使用臨時容器`build-env`，搭配 .net CORE 2.1 SDK 來建立範例應用程式，然後將輸出二進位檔案複製到另一個容器，只包含 .net core 2.1 執行時間，因此我們將最終容器的大小最小化。

## <a name="build-and-run-the-app"></a>建置和執行應用程式

在撰寫 Dockerfile 的情況下，我們可以將 Docker 指向我們的 Dockerfile，並告訴它建立並執行影像：

1. 在命令提示字元視窗中，流覽至 dockerfile 所在的目錄，然後執行[docker 建立](https://docs.docker.com/engine/reference/commandline/build/)命令，以從 dockerfile 建立容器。

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 若要執行新建立的容器，請執行[docker [執行](https://docs.docker.com/engine/reference/commandline/run/)] 命令。

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   讓我們 dissect 這個命令：

   * `-d` 告訴 Docker tun 執行容器「已分離」，這表示容器內的主機上沒有任何主機掛接到主機。 容器在背景中執行。 
   * `-p 5000:80` 指示 Docker 將主機上的埠5000對應到容器中的埠80。 每個容器都會取得自己的 IP 位址。 ASP .NET 預設會在埠80上偵聽。 埠對應可讓我們移至對應埠中主機的 IP 位址，而 Docker 會將所有流量轉寄到容器內的目的地埠。
   * `--name myapp` 告訴 Docker 可讓這個容器輕鬆地進行查詢，而不需要在執行時間使用 Docker 指派的 contaienr 識別碼。
   * `my-asp-app` 是我們想要 Docker 執行的影像。 這是程式的 culmination `docker build`所產生的容器影像。

3. 開啟網頁瀏覽器網頁瀏覽器，然後`http://localhost:5000`流覽至要查看您的容器化應用程式，如下列螢幕擷取畫面所示：

   >![從容器中的 localhost 執行 ASP.NET 核心網頁](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>後續步驟

1. 下一步是使用 Azure 容器登錄，將您的容器化 ASP.NET web 應用程式發佈到專用的註冊表。 這可讓您將它部署在您的組織中。

   > [!div class="nextstepaction"]
   > [建立私人容器註冊表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   當您到達將[容器影像推入到註冊表](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)的區段時，請指定您剛剛封裝（`my-asp-app`）的 ASP.NET app 名稱，以及您的容器登錄（例如： `contoso-container-registry`）：

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   若要查看更多應用程式範例及其相關 dockerfiles，請參閱[其他容器範例](../samples.md)。

2. 將應用程式發佈到容器登錄後，下一步就是將應用程式部署到您使用 Azure Kubernetes 服務建立的 Kubernetes 群集。

   > [!div class="nextstepaction"]
   > [建立 Kubernetes 群集](https://docs.microsoft.com/azure/aks/windows-container-cli)
