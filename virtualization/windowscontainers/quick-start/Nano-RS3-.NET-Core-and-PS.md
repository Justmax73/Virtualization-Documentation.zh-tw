# <a name="build-and-run-an-application-with-or-without-net-core-20-or-powershell-core-6"></a>使用或不使用 .NET Core 2.0 或 PowerShell Core 6 建置並執行應用程式

這個版本的 Nano Server 基本 OS 容器映像已經移除 .NET Core 和 PowerShell，不過支援 .NET Core 與 PowerShell 做為基本 Nano Server 容器之上的附加元件層容器。  

如果您的容器要執行機器碼或開放式架構 (例如 Node.js、Python、Ruby 等)，基本 Nano Server 容器就已足夠。  不同於 Windows Server 2016 版本的其中一項細微差別是，某些機器碼可能會由於這個版本的[節省磁碟使用量](https://docs.microsoft.com/windows-server/get-started/nano-in-semi-annual-channel)而無法執行。 如果您發現任何迴歸問題，請在[論壇](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)中告訴我們。 

若要根據 Dockerfile 建置您的容器，請使用 docker build；若要執行容器，請使用 docker run。  下列命令將會下載 Nano Server 容器基本 OS 映像 (可能需要幾分鐘的時間)，並且在主機主控台上列印 “Hello World!” 訊息。

```
docker run microsoft/nanoserver-insider cmd /c echo Hello World!
```

您可以使用 [Windows 上的 Dockerfile](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile) 搭配 FROM、RUN、COPY、ADD、CMD 等 Dockerfile 語法來建置更複雜的應用程式。雖然您無法立即透過此基本映像執行某些命令，不過您現在可以建立只包含讓應用程式運作所需項目的容器映像。

基本 Nano Server 容器 OS 映像不提供 .NET Core 與 PowerShell 所產生的挑戰在於，如何以採用壓縮 Zip 格式的內容建置容器。 您可以透過 Docker 17.05 所提供的[多階段組建](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)功能，運用另一個容器的 PowerShell 來解壓縮內容並複製到 Nano 容器中。 此方法可用來建立 .NET Core 容器和 PowerShell 容器。 

您可以使用這個命令來提取 PowerShell 容器映像：

```
docker pull microsoft/nanoserver-insider-powershell
```

您可以使用這個命令來提取 .NET Core 容器映像：

```
docker pull microsoft/nanoserver-insider-dotnet
```

下面舉例說明我們如何使用多階段組建來建立這些容器映像。

## <a name="deploy-apps-based-on-net-core-20"></a>部署以 .NET Core 2.0 為基礎的應用程式
您可以運用測試人員版本中的 .NET Core 2.0 容器映像來執行您的 .NET Core 應用程式，但前提是您的 .NET Core 應用程式是在其他地方建置，而且您想要在容器中執行此應用程式。  您可以在 [.NET Core GitHub](https://github.com/dotnet/dotnet-docker-nightly) 找到更多有關如何使用 .NET Core 容器映像來執行 .NET Core 應用程式的資訊。  如果您要在容器內部開發應用程式，就應該改用 .NET Core SDK。  如果是進階使用者，您可以使用 [dotnet-docker-nightly](https://github.com/dotnet/dotnet-docker-nightly/tree/master/2.0) 中指定的 .NET Core 2.0 版本、Dockerfile 和 URL 建置自己的 .NET Core 2.0 容器。 若要這樣做，您可以使用 Windows Server Core 容器來完成下載與解壓縮功能。  Dockerfile 範例如 [.NET Core Runtime Dockerfile](https://github.com/dotnet/dotnet-docker-nightly/blob/master/2.0/runtime/nanoserver-insider/amd64/Dockerfile) 所示。


透過這個 Dockerfile，您就可以使用下列命令來建置 .NET Core 2.0 容器。

```
docker build -t nanoserverdnc -f Dockerfile-dotnetRuntime .
```

## <a name="run-powershell-core-6-in-a-container"></a>在容器中執行 PowerShell Core 6
您可以透過相同的[多階段組建](https://docs.docker.com/engine/userguide/eng-image/multistage-build/)方法，使用[這個 PowerShell Dockerfile](https://github.com/PowerShell/PowerShell-Docker/blob/master/release/stable/nanoserver/docker/Dockerfile) 來建置 PowerShell Core 6 容器。


然後，發出 docker build 以建立 PowerShell 容器映像。

``` 
docker build -t nanoserverPowerShell6 -f Dockerfile-PowerShell6 .
```

您可以在 [PowerShell GitHub](https://github.com/PowerShell/PowerShell-Docker/tree/master/release) 找到詳細資訊。  值得一提的是，PowerShell Zip 包含建置 PowerShell Core 6 必要的 .NET Core 2.0 子集。  如果您的 PowerShell 模組相依於 .NET Core 2.0，可以放心地在 Nano .NET Core 容器 (而非基本 Nano 容器) 之上建置 PowerShell 容器，也就是在 Dockerfile 中使用 FROM microsoft/nanoserver-insider-dotnet。 

## <a name="next-steps"></a>後續步驟
- 使用 Docker Hub 中提供以 Nano Server 為基礎的其中一個新容器映像，即基本 Nano Server 映像、Nano 與 .NET Core 2.0 以及 Nano 與 PowerShell Core 6
- 使用本指南的 Dockerfile 範例內容，根據新的 Nano Server 容器基本 OS 映像建置您自己的容器映像 
