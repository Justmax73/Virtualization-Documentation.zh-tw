



# Docker 和 Windows

**這是初版內容，後續可能會變更。**

Docker 是一個適用於 Linux 和 Windows 容器的容器部署和管理平台。 Docker 可用來建立、管理和刪除容器及容器映像。 Docker 可讓您將容器映像儲存在公用登錄 (Docker Hub) 和私人登錄 (Docker 信任登錄) 中。 此外，Docker 還透過 Docker Swarm 提供容器主機叢集功能，並透過 Docker Compose 提供部署自動化功能。 如需有關 Docker 和 Docker 工具組的詳細資訊，請造訪 [Docker.com](https://www.docker.com/)。

> 必須先啟用 Windows 容器功能，Docker 才可用來建立及管理 Windows Server 和 Hyper-V 容器。 如需有關啟用這項功能的指示，請參閱[容器主機部署指南](./docker_windows.md)。

## Windows Server

### 安裝 Docker

Docker 精靈和 CLI 並未隨附於 Windows 或 Windows Server Core，且不會隨 Windows 容器功能一起安裝。 Docker 必須另外安裝。 本文將逐步引導您手動安裝 Docker 精靈和 Docker 用戶端。 此外也會提供完成這些工作的自動化方法。

Docker 精靈和 Docker 命令列介面是以 Go 語言開發的。 目前，docker.exe 不會安裝為 Windows 服務。 有數種方法可用來建立 Windows 服務，這裡顯示的一個範例使用 `nssm.exe`。

從 `https://aka.ms/tp4/docker` 下載 docker.exe，將它放在容器主機的 System32 目錄中。

```powershell
PS C:\> wget https://aka.ms/tp4/docker -OutFile $env:SystemRoot\system32\docker.exe
```

建立名為 `c:\programdata\docker` 的目錄。 在此目錄中，建立名為 `runDockerDaemon.cmd` 的檔案。

```powershell
PS C:\> New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

將下列文字複製到 `runDockerDaemon.cmd` 檔案中。 這個批次檔會使用 `docker daemon -D -b “Virtual Switch”` 命令啟動 Docker 精靈。 注意：此檔案中的虛擬交換器名稱，必須符合容器將用於網路連線的虛擬交換器名稱。

```powershell
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (goto :secure)

docker daemon -D -b "Virtual Switch"
goto :eof

:secure
docker daemon -D -b "Virtual Switch" -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```
從 [https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip) 下載 nssm.exe。

```powershell
PS C:\> wget https://nssm.cc/release/nssm-2.24.zip -OutFile $env:ALLUSERSPROFILE\nssm.zip
```

解壓縮檔案，並將 `nssm-2.24\win64\nssm.exe` 複製到 `c:\windows\system32` 目錄中。

```powershell
PS C:\> Expand-Archive -Path $env:ALLUSERSPROFILE\nssm.zip $env:ALLUSERSPROFILE
PS C:\> Copy-Item $env:ALLUSERSPROFILE\nssm-2.24\win64\nssm.exe $env:SystemRoot\system32
```
執行 `nssm install` 以設定 Docker 服務。

```powershell
PS C:\> start-process nssm install
```

將下列資料輸入到 NSSM 服務安裝程式的對應欄位中。

應用程式索引標籤：

- **路徑：**C:\Windows\System32\cmd.exe

- **啟動目錄：**C:\Windows\System32

- **引數：** /s /c C:\ProgramData\docker\runDockerDaemon.cmd < nul

- **服務名稱** - Docker

![](media/nssm1.png)

詳細資料索引標籤：

- **顯示名稱：**Docker

- **描述：**Docker 精靈可為 docker 用戶端提供容器的管理功能。


![](media/nssm2.png)

IO 索引標籤：

- **輸出 (stdout)：**C:\ProgramData\docker\daemon.log

- **錯誤 (stderr)：**C:\ProgramData\docker\daemon.log


![](media/nssm3.png)

完成時，請按一下 `[安裝服務]` 按鈕。

此作業成後，當 Windows 啟動時，Docker 精靈 (服務) 也也啟動。

### 移除 Docker

如果依照本指南從 docker.exe 建立 Windows 服務，下列命令將會移除服務。

```powershell
PS C:\> sc.exe delete Docker

[SC] DeleteService SUCESS
```

## Nano Server

### 安裝 Docker

從 `https://aka.ms/tp4/docker` 下載 docker.exe，並將其複製到 Nano Server 容器主機的 `windows\system32` 資料夾中。

執行下列命令以啟動 docker 精靈。 容器主機每次啟動時，都必須執行此動作。 此命令會啟動 Docker 精靈、指定容器連線的虛擬交換器，並設定精靈在連接埠 2375 上接聽傳入 Docker 要求。 在此設定中，Docker 可從遠端電腦來管理。

```powershell
PS C:\> start-process cmd "/k docker daemon -D -b <Switch Name> -H 0.0.0.0:2375”
```

### 移除 Docker

若要從 Nano Server 中移除 docker 精靈和 cli，請從 Windows\system32 目錄中刪除 `docker.exe`。

```powershell
PS C:\> Remove-Item $env:SystemRoot\system32\docker.exe
```

### 互動式 Nano 工作階段

> 如需從遠端管理 Nano Server 的詳細資訊，請參閱[開始使用 Nano Server](https://technet.microsoft.com/en-us/library/mt126167.aspx#bkmk_ManageRemote)。

以互動方式管理 Nano Server 主機上的容器時，您可能會收到此錯誤。

```powershell
docker : cannot enable tty mode on non tty input
+ CategoryInfo          : NotSpecified: (cannot enable tty mode on non tty input:String) [], RemoteException
+ FullyQualifiedErrorId : NativeCommandError 
```

這種情形可能會發生於嘗試使用 -it 以互動式工作階段執行容器時：

```powershell
Docker run -it <image> <command>
```
或嘗試附加到執行中的容器：

```powershell
Docker attach <container name>
```

若要使用 Docker 在 Nano Server 主機上建立的容器來建立互動式工作階段，必須從遠端管理 Docker 精靈。 若要這樣做，請從[這個位置](https://aka.ms/ContainerTools)下載 docker.exe 並將它複製到遠端系統。

首先，您必須在 Nano Server 中設定 Docker 精靈，以接聽遠端命令。 在 Nano Server 中執行以下命令，即可執行這項操作：

```powershell
docker daemon -D -H <ip address of Nano Server>:2375
```

現在，在您的電腦上開啟 PowerShell 或 CMD 工作階段，然後執行 Docker 命令以 `-H` 指定遠端主機。

```powershell
.\docker.exe -H tcp://<ip address of Nano Server>:2375
```

例如，如果您想要查看可用的映像：

```powershell
.\docker.exe -H tcp://<ip address of Nano Server>:2375 images
```






<!--HONumber=Feb16_HO4-->


