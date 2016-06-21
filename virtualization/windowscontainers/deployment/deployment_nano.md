---
title: 在 Nano Server 上部署 Windows 容器
description: 在 Nano Server 上部署 Windows 容器
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
---

# 容器主機部署 - Nano Server

**這是初版內容，後續可能會變更。** 

開始在 Nano Server 上設定 Windows 容器之前，您需具備執行 Nano Server 的系統，以及此系統的遠端 PowerShell 連線。

如需部署和連接 Nano Server 的詳細資訊，請參閱 [Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx) (開始使用 Nano Server)。

您可以在[這裡](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula)找到 Nano Server 評估版。

## 安裝容器功能

安裝 Nano Server 套件管理提供者。

```none
Install-PackageProvider NanoServerPackage
```

安裝套件提供者之後，請安裝容器的功能。

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

安裝這些功能之後，Nano Server 主機必須重新開機。

## 安裝 Docker

需要先安裝 Docker，才能使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 請使用下列步驟安裝 Docker 精靈。

下載 Docker 精靈，並將它複製到容器主機的 `$env:SystemRoot\system32\`。 Nano Server 目前不支援 `Invoke-Webrequest`，此作業需要透過遠端系統來完成。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

將 Docker 安裝為 Windows 服務。

```none
dockerd.exe --register-service
```

啟動 Docker 服務。

```none
Start-Service Docker
```

## 安裝基本容器映像

基本 OS 映像可作為任何 Windows Server 或 Hyper-V 容器的基底。 基本 OS 映像可作為 Windows Server Core 和 Nano Server 的基礎作業系統，而且可以使用容器映像提供者加以安裝。 如需 Windows 容器映像的詳細資訊，請參閱[管理容器映像](../management/manage_images.md)。

您可以使用下列命令來安裝容器映像提供者。

```none
Install-PackageProvider ContainerImage -Force
```

若要下載並安裝 Nano Server 基本映像，請執行下列命令：

```none
Install-ContainerImage -Name NanoServer
```

**注意：**目前只有 Nano Server 基本映像與 Nano Server 容器主機相容。

重新啟動 Docker 服務。

```none
Restart-Service Docker
```

最後，此映像必須加上「最新」版本的標籤。 若要這樣做，請執行以下命令。

```none
docker tag nanoserver:10.0.14300.1010 nanoserver:latest
```

## Hyper-V 容器主機

若要部署 Hyper-V 容器，需具備 Hyper-V 角色。 如果 Windows 容器主機本身為 Hyper-V 虛擬機器，則必須先啟用巢狀虛擬化，才能安裝 Hyper-V 角色。 如需巢狀虛擬化的詳細資訊，請參閱＜巢狀虛擬化＞。

### 巢狀虛擬化

下列指令碼可設定容器主機的巢狀虛擬化。 您必須在裝載容器主機虛擬機器的 Hyper-V 機器上執行此指令碼。 執行這個指令碼時，容器主機的虛擬機器務必要關閉。

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### 啟用 Hyper-V 角色

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

## 管理 Nano Server 上的 Docker

**準備 Docker 精靈：**

請透過遠端系統來管理 Nano Server 上的 Docker，以獲得最佳體驗。 若要這樣做，必須先完成下列項目。

在容器主機上建立防火牆規則以連接 Docker。 如果是不安全的連線，為連接埠 `2375`；安全的連線則為連接埠 `2376`。

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

將 Docker 精靈設為接受透過 TCP 的連入連線。

首先，在 `c:\ProgramData\docker\config\daemon.json` 建立 `daemon.json` 檔案。

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

接下來，將此 JSON 複製到檔案中。 如此會將 Docker 精靈設為接受透過 TCP 連接埠 2375 的連入連線。 這是不安全的連線，因此並不建議您使用，但可用於隔離測試。

```none
{
    "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
}
```

下列範例會設定安全的遠端連線。 您必須建立 TLS 憑證，並將其複製到適當的位置。 如需詳細資訊，請參閱 [Windows 上的 Docker 精靈](./docker_windows.md)。

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

重新啟動 Docker 服務。

```none
Restart-Service docker
```

**準備 Docker 用戶端：**

下載遠端管理系統上的 Docker 用戶端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:SystemRoot\system32\docker.exe
```

完成之後，即可使用 `Docker -H` 參數存取 Docker 精靈。

```none
docker -H tcp://10.0.0.5:2376 run -it nanoserver cmd
```

您可建立環境變數 `DOCKER_HOST`，並移除 `-H` 參數的需求。 您可以使用下列 PowerShell 命令來完成此作業：

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

使用此變數集時，此命令現在會看起來如下：

```none
docker run -it nanoserver cmd
```

<!--HONumber=May16_HO5-->


