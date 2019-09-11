---
title: Containerize .NET Core 應用程式
description: 瞭解如何使用容器建立範例 .NET 核心應用程式
keywords: Docker, 容器
author: cwilhit
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: f5a51fd1211868195126f06d917c0bef6e496c3d
ms.sourcegitcommit: f3b6b470dd9cde8e8cac7b13e7e7d8bf2a39aa34
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2019
ms.locfileid: "10077469"
---
# <a name="containerize-a-net-core-app"></a>Containerize .NET Core 應用程式


在這個快速入門中，您將瞭解如何 containerize 簡單的 .NET core 應用程式。 您會：

> [!div class="checklist"]
> * 從 GitHub 克隆範例應用程式來源
> * 建立 dockerfile 以使用 app 來源建立容器影像
> * 在本機 Docker 環境中測試以容器為的 .NET 核心應用程式

## <a name="before-you-begin"></a>在您開始前

此快速入門假設您的開發環境已設定為使用容器。 如果您沒有為容器設定環境，請造訪[Windows 10 快速入門](./quick-start-windows-10.md)，瞭解如何開始使用。

您必須在電腦上安裝 Git 來源控制系統。 您可以在此抓取： [Git](https://git-scm.com/download)

## <a name="getting-started"></a>開始使用

在名`windows-container-samples`為的資料夾中，所有容器範例原始程式碼都保留在[虛擬化-檔](https://github.com/MicrosoftDocs/Virtualization-Documentation)git 存放庫底下。 將此 git 存放庫克隆到您的 curent 工作目錄。

```Powershell
git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
```

流覽至在`<directory where clone occured>\Virtualization-Documentation\windows-container-samples\asp-net-getting-started`中找到的範例目錄，並建立 Dockerfile。 [Dockerfile](https://docs.docker.com/engine/reference/builder/)就像是一個 makefile，就像是一個資訊清單，指示容器引擎如何建立容器影像。

```Powershell
#Create the dockerfile for our project
New-Item -name dockerfile -type file
```

## <a name="write-the-dockerfile"></a>撰寫 dockerfile

開啟您剛建立的 dockerfile （使用您有任何您喜愛的任何文字編輯器），然後新增下列內容。

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

我們已撰寫 dockerfile 來執行_多階段組建_。 當 dockerfile 執行時，它會使用臨時容器`build-env`，搭配 .net CORE 2.1 SDK 來建立範例應用程式，然後將輸出二進位檔案複製到另一個容器，只包含 .net core 2.1 執行時間，讓我們最大限度地減少最終容器。

## <a name="run-the-app"></a>執行應用程式

在撰寫 dockerfile 的情況下，我們可以將 Docker 指向我們的 dockerfile，並告訴它建立我們的影像。 

>[!IMPORTANT]
>下列執行的命令必須在 dockerfile 所在的目錄中執行。

```Powershell
docker build -t my-asp-app .
```

若要執行容器，請執行下列命令。

```Powershell
docker run -d -p 5000:80 --name myapp my-asp-app
```

讓我們 dissect 這個命令：

* `-d` 告訴 Docker tun 執行容器「已分離」，這表示容器內的主機上沒有任何主機掛接到主機。 容器在背景中執行。 
* `-p 5000:80` 指示 Docker 將主機上的埠5000對應到容器中的埠80。 每個容器都會取得自己的 IP 位址。 ASP .NET 預設會在埠80上偵聽。 埠對應可讓我們移至對應埠中主機的 IP 位址，而 Docker 會將所有流量轉寄到容器內的目的地埠。
* `--name myapp` 告訴 Docker 可讓這個容器輕鬆地進行查詢，而不需要在執行時間使用 Docker 指派的 contaienr 識別碼。
* `my-asp-app` 是我們想要 Docker 執行的影像。 這是程式的 culmination `docker build`所產生的容器影像。

開啟網頁瀏覽器網頁瀏覽器，然後`https://localhost:5000`流覽至要由您的容器化應用程式 greeted。

>![](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>後續步驟

我們已成功地以 ASP.NET web app 為容器。 若要查看更多應用程式範例及其相關 dockerfiles，請按一下下方的按鈕。

> [!div class="nextstepaction"]
> [查看更多容器範例](../samples.md)
