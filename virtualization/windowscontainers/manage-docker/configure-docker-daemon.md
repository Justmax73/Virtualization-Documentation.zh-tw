---
title: 在 Windows 中設定 Docker
description: 在 Windows 中設定 Docker
keywords: docker, 容器
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: bbc405fc2a490cfe5082be112fde724707e24785
ms.sourcegitcommit: 21d93e5febd9b1b47ae1aa59d08086e6ec1691e0
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2019
ms.locfileid: "9121050"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Docker 引擎及用戶端並未隨附於 Windows，並需要進行個別安裝及設定。 此外，Docker 引擎可接受許多自訂設定。 部分範例包括設定精靈接受連入要求的方式、預設網路功能選項，以及偵錯/記錄設定。 在 Windows 中，這些設定可以在設定檔中指定，或使用 Windows 服務控制管理員指定。 這份文件詳細說明如何安裝及設定 Docker 引擎，並提供一些常用設定的範例。


## <a name="install-docker"></a>安裝 Docker
需要有 Docker 才能使用 Windows 容器。 Docker 是由 Docker 引擎(dockerd.exe) 及 Docker 用戶端 (docker.exe) 所組成。 您可以在快速入門指南中找到安裝所有項目最簡單的方法。 他們可協助您取得所有項目設定和執行您的第一個容器。 

* [Windows Server 2019 上 Windows 容器](../quick-start/quick-start-windows-server.md)
* [Windows 10 上的 Windows 容器](../quick-start/quick-start-windows-10.md)

如需使用指令碼進行安裝，請參閱[使用指令碼安裝 Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

才能使用 Docker 容器映像需要安裝。 如需詳細資訊，請參閱[使用映像快速入門指南](../quick-start/quick-start-images.md)。

## <a name="configure-docker-with-configuration-file"></a>使用設定檔設定 Docker

建議使用設定檔在 Windows 上設定 Docker 引擎。 設定檔位於 'C:\ProgramData\Docker\config\daemon.json'。 若尚未有此檔案，請加以建立。

請注意，並非所有可用的 Docker 設定選項都適用於 Windows 上的 Docker。 下列所示範例為適用的選項。 如需 Docker 引擎設定的完整文件，請參閱 [Docker 精靈的組態檔](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

```
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
    "data-root": "",
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

```
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

此範例同樣會設定 Docker 精靈將映像和容器保留在的替代路徑中。 若未指定，則預設路徑為 c:\programdata\docker。

```
{    
    "data-root": "d:\\docker"
}
```

同樣地，此範例會將 Docker 精靈設定為只接受透過連接埠 2376 連接的安全連線。

```
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>在 Docker 服務設定 Docker

也可以透過使用 `sc config` 修改 Docker 服務來設定 Docker 引擎。 若使用此方法，會直接在 Docker 服務上設定 Docker 引擎旗標。 在命令提示字元 (cmd.exe 而非 PowerShell) 執行下列命令︰


```
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

注意︰若您的 daemon.json 檔案已包含 `"hosts": ["tcp://0.0.0.0:2375"]` 項目，就不需要執行此命令。

## <a name="common-configuration"></a>一般設定

下列設定檔範例會顯示 Docker 的一般設定。 這些可以合併成單一設定檔。

### <a name="default-network-creation"></a>建立預設網路 

若要設定 Docker 引擎而不建立預設 NAT 網路，請使用下列項目。 如需詳細資訊，請參閱 [Manage Docker Networks](../container-networking/network-drivers-topologies.md) (管理 Docker 網路)。

```
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>設定 Docker 安全性群組

當登入 Docker 主機並在本機執行 Docker 命令時，會透過具名管道執行這些命令。 依預設，只有系統管理員群組的成員可以透過具名管道存取 Docker 引擎。 若要指定具有此存取權的安全性群組，請使用 `group` 旗標。

```
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Proxy 組態

若要設定 `docker search` 與 `docker pull` 的 Proxy 資訊,，請以名稱 `HTTP_PROXY` 或 `HTTPS_PROXY` 建立 Windows 環境變數，並設定 Proxy 資訊的值。 此作業可透過類似下列所示的 PowerShell 命令完成︰

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

完成變數設定之後，請重新啟動 Docker 服務。

```powershell
Restart-Service docker
```

如需詳細資訊，請參閱 [Docker.com Windows 的組態檔](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

## <a name="uninstall-docker"></a>解除安裝 Docker
*使用本節中的步驟，解除安裝 Docker，並從 Windows 10 或 Windows Server 2016 系統執行 Docker 系統元件完全清除。*

> 注意：下列步驟中的所有命令都必須從**提升權限的** PowerShell 工作階段執行。

### <a name="step-1-prepare-your-system-for-dockers-removal"></a>步驟 1：準備系統以移除 Docker 
如果您尚未執行此步驟，最好先確認系統上未執行任何容器，然後移除 Docker。 以下是一些執行此作業的實用命令：
```
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```
同時也最好先從您的系統移除所有容器、容器映像、網路和磁碟區，然後移除 Docker：
```
docker system prune --volumes --all
```

### <a name="step-2-uninstall-docker"></a>步驟 2：解除安裝 Docker 

#### ***<a name="steps-to-uninstall-docker-on-windows-10"></a>下列步驟用來解除安裝 Windows 10 上的 Docker：10:***
- 移至 Windows 10 電腦上的 **\[設定\] > \[應用程式\]**
- 在 **\[應用程式與功能\]** 下，尋找 **「Windows Docker」**
- 按一下 **\[Docker for Windows\] > \[解除安裝\]**

#### ***<a name="steps-to-uninstall-docker-on-windows-server-2016"></a>下列步驟用來解除安裝 Windows Server 2016 上的 Docker：16:***
從提升權限的 PowerShell 工作階段，使用 `Uninstall-Package` 與 `Uninstall-Module` Cmdlet，從系統移除 Docker 模組及其對應的套件管理提供者。 
> 秘訣：您可以使用下列命令，找到您用來安裝 Docker 的套件提供者： `PS C:\> Get-PackageProvider -Name *Docker*`

*例如*：
```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

### <a name="step-3-cleanup-docker-data-and-system-components"></a>步驟 3：清理 Docker 資料與系統元件
移除 Docker 的*預設網路，*，這樣其設定在 Docker 消失後不會留在系統：
```
Get-HNSNetwork | Remove-HNSNetwork
```
從系統移除 Docker 的*程式資料*：
```
Remove-Item "C:\ProgramData\Docker" -Recurse
```
您也可以移除與 Windows 上的 Docker/容器相關的 *Windows 選用功能*。 

這至少包含「容器」功能，在已安裝 Docker 的任何 Windows 10 或 Windows Server 2016 上，此功能會自動啟用。 也可能包含 "Hyper-V" 功能，在已安裝 Docker 的 Windows 10 上，此功能會自動啟用，但在 Windows Server 2016 上必須明確啟用。

> **停用 HYPER-V 有關的重要事項：**[Hyper-V 功能](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/)不只會啟用容器，也是普遍的虛擬化功能！ 停用 Hyper-V 功能之前，請確定系統上沒有任何其他虛擬化元件需要它。

#### ***<a name="steps-to-remove-windows-features-on-windows-10"></a>下列步驟用來移除 Windows 10 上的 Windows 功能：10:***
- 移至 Windows 10 電腦上的 **\[控制台\] > \[程式和功能\] > \[開啟或關閉 Windows 功能\]**。
- 找出您想要停用的功能名稱--在此案例中是 **\[容器\]** 以及 (選擇性) **\[Hyper-V\]**
- **取消選取**您想要停用的功能名稱旁邊的方塊
- 按一下 **\[確定\]**。

#### ***<a name="steps-to-remove-windows-features-on-windows-server-2016"></a>下列步驟用來移除 Windows Server 2016 上的 Windows 功能：16:***
從提升權限的 PowerShell 工作階段，使用下列命令來停用 **\[容器\]** 以及 (選擇性) **\[Hyper-V\]** 系統功能：
```
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V 
```

### <a name="step-4-reboot-your-system"></a>步驟 4：重新啟動系統
若要完成這些解除安裝/清理步驟，從提升權限的 PowerShell 工作階段，執行：
```
Restart-Computer -Force
```
