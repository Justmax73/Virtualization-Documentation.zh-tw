---
title: "建置範例應用程式"
description: "了解如何運用容器建置範例應用程式"
keywords: "Docker, 容器"
author: cwilhit
ms.date: 07/25/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 2ba3e6409fc60022a55d21c187bfcaefd962908b
ms.sourcegitcommit: 4f5b9f70804bf6282af8bef603cc343c524c3102
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/04/2017
---
# 建置範例應用程式

本練習將逐步引導您取用範例 ASP.net 應用程式並進行轉換，以便在容器中執行。 如果您需要了解如何在 Windows 10 中啟動並執行容器，請瀏覽 [Windows 10 快速入門](./quick-start-windows-10.md)。

此快速入門專用於 Windows 10。 在此頁面左側的目錄中，可以找到其他的快速入門文件。 由於本教學課程的重點是容器，因此我們將省略撰寫程式碼的步驟，而將重點完全放在容器上。 如果您需要從頭開始建置的教學課程，可以在 [ASP.NET Core 文件](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app-xplat/)中找到。

如果您的電腦尚未安裝 Git 原始檔控制，可以在這裡下載：[Git](https://git-scm.com/download)

## 開始使用

此範例專案是使用 [VSCode](https://code.visualstudio.com/) 所設定。 我們也將使用 PowerShell。 現在，我們先從 GitHub 取得示範程式碼。 您可以複製 Git 存放庫，也可以直接從 [SampleASPContainerApp](https://github.com/cwilhit/SampleASPContainerApp) 下載專案。

```Powershell
git clone https://github.com/cwilhit/SampleASPContainerApp.git
```

接下來，我們瀏覽到專案目錄並建立 Dockerfile。 [Dockerfile](https://docs.docker.com/engine/reference/builder/) 就像是 Makefile，其中包含描述必須如何建置容器映像的指示清單。

```Powershell
#Create the dockerfile for our proj
New-Item C:/Your/Proj/Location/Dockerfile -type file
```

## 撰寫我們的 Dockerfile

現在，我們來開啟在專案根資料夾中建立的 Dockerfile (可使用您喜歡的任何文字編輯器) 並加入一些邏輯。 然後，我們將逐行解說每一行的作用。

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

第一組程式碼行會宣告我們將使用哪個基本映像做為建置容器的基礎。 如果本機系統原本沒有這個映像，則 Docker 將自動嘗試擷取該映像。 Aspnetcore-build 是與編譯專案的相依性封裝在一起。 接著，我們要將容器的工作目錄變更為 '/app'，好讓 Dockerfile 中所有隨後的命令都能在該目錄中執行。

_注意_：因為我們必須建置專案，所以我們所建立的第一個容器就是單純用來建置專案的暫存容器，結束後便加以捨棄。

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app
```

接下來，我們要將 .csproj 檔案複製到暫存容器的 '/app' 目錄中。 這樣做的原因是 .csproj 檔案包含專案所需之套件參照的清單。

複製這個檔案之後，dotnet 將讀取其內容，然後開始擷取專案所需的所有相依性和工具。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

一旦我們獲得所有相依性之後，就要將它們複製到暫存容器中。 然後，我們使用發行組態並指定輸出路徑讓 dotnet 來發佈我們的應用程式。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

至此，我們的專案應該已經順利編譯完成了。 現在，我們需要建置最終的容器。 因為我們的應用程式是 ASP.NET，所以我們要將包含這些程式庫的映像指定為來源。 然後，我們要將所有檔案從暫存容器的輸出目錄複製到最終容器中。 我們要將容器設定為啟動時要使用我們所編譯的新 .dll 來執行。

_注意_：這個最終容器的基本映像與上述的 ```FROM``` 命令很相似但有所不同：它沒有具備_建置_ ASP.NET 應用程式功能的程式庫，只能執行。

```Dockerfile
FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

現在，我們已經順利執行所謂的_多階段組建_。 我們使用了暫存容器來建置映像，然後將已發佈的 dll 移動到另一個容器中，以便將最終結果的磁碟使用量減到最低。 我們希望這個容器將執行所需的相依性保持在絕對最低限度；如果我們繼續使用第一個映像，它就會與其他不重要的層級 (用於建置 ASP.NET 應用程式) 封裝在一起，因而增加映像的大小。

## 執行應用程式

現在 Dockerfile 已撰寫完畢，剩下的步驟就是告知 Docker 建置應用程式，然後執行容器。 我們要指定發佈的目標連接埠，然後為容器提供一個標記 "myapp"。 在 PowerShell 中，執行下列命令：

```Powershell
docker build -t myasp .
docker run -d -p 5000:80 --name myapp myasp
```

若要查看執行中的應用程式，我們需要瀏覽執行所在的位址。 讓我們執行此命令以取得 IP 位址。

```Powershell
 docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" myapp
```

執行此命令將會產生執行中容器的 IP 位址。 此輸出可能看起來會像以下的範例：

```Powershell
 172.19.172.12
```

在您選擇的網頁瀏覽器中輸入這個 IP 位址，迎接您的就是在容器中順利執行的應用程式！

<center style="margin: 25px">![](media/SampleAppScreenshot.png)</center>

按一下導覽列中的 "MvcMovie" 就會將您帶到可輸入、編輯和刪除影片項目的網頁。

## 後續步驟

我們已經成功地取用 ASP.NET Web 應用程式、使用 Docker 加以設定並建置，而且成功地將它部署到執行中的容器中。 但是，您還可以繼續執行其他步驟！ 您可以將這個 Web 應用程式細分為其他多個元件：執行 Web API 的容器、執行前端的容器，以及執行 SQL Server 的容器。

現在您已經掌握容器的竅門，接下來就可以大展身手，運用容器建置出色的軟體了！ 以下是其他範例容器的清單：

[範例容器](../samples.md)
