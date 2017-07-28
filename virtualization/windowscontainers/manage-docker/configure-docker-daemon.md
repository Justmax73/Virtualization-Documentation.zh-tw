---
title: "在 Windows 中設定 Docker"
description: "在 Windows 中設定 Docker"
keywords: "docker, 容器"
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: f266404f12e47c8605436af44e636c54ec6ef8e5
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/21/2017
---
# Windows 上的 Docker 引擎

Docker 引擎和代理程式並未隨附於 Windows，且需要個別安裝及設定。 此外，Docker 引擎可接受許多自訂設定。 部分範例包括設定精靈接受連入要求的方式、預設網路功能選項，以及偵錯/記錄設定。 在 Windows 中，這些設定可以在設定檔中指定，或使用 Windows 服務控制管理員指定。 本文詳細說明如何安裝及設定 Docker 引擎，且提供常用設定的範例。


## 安裝 Docker
需要有 Docker 才能使用 Windows 容器。 Docker 是由 Docker 引擎(dockerd.exe) 及 Docker 用戶端 (docker.exe) 所組成。 您可以在快速入門指南中找到安裝所有項目最簡單的方法。 指南會協助您設定好所有項目，以及執行您的第一個容器。 

* [Windows Server 2016 上的 Windows 容器](../quick-start/quick-start-windows-server.md)
* [Windows 10 上的 Windows 容器](../quick-start/quick-start-windows-10.md)


### 手動安裝
如果您要改用開發中的 Docker 引擎及用戶端版本，可以使用後續步驟。 這會安裝 Docker 引擎及用戶端。 如果您是開發人員而要測試新功能或使用 Windows 測試人員組建，可能就需要使用開發中的 Docker 版本。 否則，請依照上述＜安裝 Docker＞一節所示步驟取得最新發行版本。

> 如果您已安裝 Docker for Windows，請務必先將其移除，再遵循以下手動安裝步驟。 

下載 Docker 引擎

最新版本一律位於 https://master.dockerproject.org。 這個範例使用 master 分支提供的最新版本。 

```powershell
$version = (Invoke-WebRequest -UseBasicParsing https://raw.githubusercontent.com/docker/docker/master/VERSION).Content.Trim()
Invoke-WebRequest "https://master.dockerproject.org/windows/x86_64/docker-$($version).zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

將該 zip 封存展開到 Program Files。

```powershell
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

將 Docker 目錄加入至系統路徑。 完成時，重新啟動 PowerShell 工作階段，以便識別修改過的路徑。

```powershell
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

若要將 Docker 安裝為 Windows 服務，請執行下列命令。

```none
dockerd --register-service
```

安裝之後，就可以啟動服務。

```powershell
Start-Service Docker
```

必須先安裝容器映像，才能使用 Docker。 如需詳細資訊，請參閱[使用映像快速入門指南](../quick-start/quick-start-images.md)。

## 使用設定檔設定 Docker

建議使用設定檔在 Windows 上設定 Docker 引擎。 設定檔位於 'C:\ProgramData\Docker\config\daemon.json'。 若尚未有此檔案，請加以建立。

請注意，並非所有可用的 Docker 設定選項都適用於 Windows 上的 Docker。 下列所示範例為適用的選項。 如需 Docker 引擎設定的完整文件，請參閱 [Docker 精靈的組態檔](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

```none
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "graph": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

只需將所需的設定變更加入設定檔中。 例如，此範例將 Docker 引擎設定為接受連接埠 2375 上的連入連線。 其他所有設定選項將使用預設值。

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

此範例同樣會設定 Docker 精靈將映像和容器保留在的替代路徑中。 若未指定，則預設路徑為 c:\programdata\docker。

```none
{    
    "graph": "d:\\docker"
}
```

同樣地，此範例會將 Docker 精靈設定為只接受透過連接埠 2376 連接的安全連線。

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## 在 Docker 服務設定 Docker

也可以透過使用 `sc config` 修改 Docker 服務來設定 Docker 引擎。 若使用此方法，會直接在 Docker 服務上設定 Docker 引擎旗標。 在命令提示字元 (cmd.exe 而非 PowerShell) 執行下列命令︰


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

注意︰若您的 daemon.json 檔案已包含 `"hosts": ["tcp://0.0.0.0:2375"]` 項目，就不需要執行此命令。

## 一般設定

下列設定檔範例會顯示 Docker 的一般設定。 這些可以合併成單一設定檔。

### 建立預設網路 

若要設定 Docker 引擎而不建立預設 NAT 網路，請使用下列項目。 如需詳細資訊，請參閱 [Manage Docker Networks](../manage-containers/container-networking.md) (管理 Docker 網路)。

```none
{
    "bridge" : "none"
}
```

### 設定 Docker 安全性群組

當登入 Docker 主機並在本機執行 Docker 命令時，會透過具名管道執行這些命令。 依預設，只有系統管理員群組的成員可以透過具名管道存取 Docker 引擎。 若要指定具有此存取權的安全性群組，請使用 `group` 旗標。

```none
{
    "group" : "docker"
}
```

## Proxy 組態

若要設定 `docker search` 與 `docker pull` 的 Proxy 資訊,，請以名稱 `HTTP_PROXY` 或 `HTTPS_PROXY` 建立 Windows 環境變數，並設定 Proxy 資訊的值。 此作業可透過類似下列所示的 PowerShell 命令完成︰

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

完成變數設定之後，請重新啟動 Docker 服務。

```powershell
Restart-Service docker
```

如需詳細資訊，請參閱 [Docker.com Windows 的組態檔](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

