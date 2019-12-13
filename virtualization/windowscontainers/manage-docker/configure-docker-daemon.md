---
title: 在 Windows 中設定 Docker
description: 在 Windows 中設定 Docker
keywords: docker, 容器
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: c84a6652b5918238ee8ef6e1fa7a9b2aa596aefd
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910158"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Docker 引擎和用戶端不包含在 Windows 中，而且需要個別安裝及設定。 此外，Docker 引擎可接受許多自訂設定。 部分範例包括設定精靈接受連入要求的方式、預設網路功能選項，以及偵錯/記錄設定。 在 Windows 中，這些設定可以在設定檔中指定，或使用 Windows 服務控制管理員指定。 本檔詳細說明如何安裝和設定 Docker 引擎，同時也提供一些常用設定的範例。

## <a name="install-docker"></a>安裝 Docker

您需要 Docker 才能使用 Windows 容器。 Docker 是由 Docker 引擎(dockerd.exe) 及 Docker 用戶端 (docker.exe) 所組成。 取得所有已安裝專案的最簡單方式是在快速入門手冊中，其可協助您設定並執行您的第一個容器。

- [安裝 Docker](../quick-start/set-up-environment.md)

如需編寫腳本的安裝，請參閱[使用腳本安裝 DOCKER EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

在您可以使用 Docker 之前，您必須先安裝容器映射。 如需詳細資訊，請參閱[容器基底映射的](../manage-containers/container-base-images.md)檔。

## <a name="configure-docker-with-a-configuration-file"></a>使用設定檔設定 Docker

建議使用設定檔在 Windows 上設定 Docker 引擎。 設定檔位於 'C:\ProgramData\Docker\config\daemon.json'。 您可以建立此檔案（如果尚未存在）。

>[!NOTE]
>並非所有可用的 Docker 設定選項都適用于 Windows 上的 Docker。 下列範例顯示適用的設定選項。 如需 Docker Engine 設定的詳細資訊，請參閱[docker daemon 設定檔](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

```json
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

您只需要將所需的設定變更新增至設定檔。 例如，下列範例會將 Docker 引擎設定為接受埠2375上的連入連線。 其他所有設定選項將使用預設值。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同樣地，下列範例會設定 Docker daemon，將映射和容器保留在替代路徑中。 如果未指定，則預設值為 `c:\programdata\docker`。

```json
{    
    "data-root": "d:\\docker"
}
```

下列範例會將 Docker daemon 設定為只接受透過埠2376的安全連線。

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>在 Docker 服務上設定 Docker

您也可以使用 `sc config`修改 Docker 服務來設定 Docker 引擎。 若使用此方法，會直接在 Docker 服務上設定 Docker 引擎旗標。 在命令提示字元 (cmd.exe 而非 PowerShell) 執行下列命令︰

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>如果您的 daemon. json 檔案已包含 `"hosts": ["tcp://0.0.0.0:2375"]` 專案，則不需要執行此命令。

## <a name="common-configuration"></a>一般設定

下列設定檔範例會顯示 Docker 的一般設定。 這些可以合併成單一設定檔。

### <a name="default-network-creation"></a>預設網路建立

若要設定 Docker Engine，使其不會建立預設 NAT 網路，請使用下列設定。

```json
{
    "bridge" : "none"
}
```

如需詳細資訊，請參閱 [Manage Docker Networks](../container-networking/network-drivers-topologies.md) (管理 Docker 網路)。

### <a name="set-docker-security-group"></a>設定 Docker 安全性群組

當您已登入 Docker 主機並在本機執行 Docker 命令時，這些命令會透過具名管道來執行。 依預設，只有系統管理員群組的成員可以透過具名管道存取 Docker 引擎。 若要指定具有此存取權的安全性群組，請使用 `group` 旗標。

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Proxy 設定

若要設定 `docker search` 與 `docker pull` 的 Proxy 資訊,，請以名稱 `HTTP_PROXY` 或 `HTTPS_PROXY` 建立 Windows 環境變數，並設定 Proxy 資訊的值。 此作業可透過類似下列所示的 PowerShell 命令完成︰

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

完成變數設定之後，請重新啟動 Docker 服務。

```powershell
Restart-Service docker
```

如需詳細資訊，請參閱[Docker.com 上的 Windows 設定檔](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

## <a name="how-to-uninstall-docker"></a>如何卸載 Docker

本節將告訴您如何卸載 Docker，並從您的 Windows 10 或 Windows Server 2016 系統執行 Docker 系統元件的完整清除。

>[!NOTE]
>您必須從提高許可權的 PowerShell 會話執行這些指示中的所有命令。

### <a name="prepare-your-system-for-dockers-removal"></a>準備您的系統以進行 Docker 的移除

卸載 Docker 之前，請確定您的系統上沒有正在執行的容器。

執行下列 Cmdlet 來檢查執行中的容器：

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

在移除 Docker 之前，從系統中移除所有容器、容器映射、網路和磁片區也是很好的作法。 若要這麼做，您可以執行下列 Cmdlet：

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>解除安裝 Docker

接下來，您必須實際卸載 Docker。

卸載 Windows 10 上的 Docker

- 前往 Windows 10 電腦上的 [**設定**] > **應用程式**
- 在 [**應用程式 & 功能**] 底下，尋找**適用於 Windows 的 Docker**
- 移至**適用於 Windows 的 Docker** > **卸載**

若要在 Windows Server 2016 上卸載 Docker：

從提高許可權的 PowerShell 會話中，使用**卸載封裝**和**卸載模組**Cmdlet，從您的系統中移除 Docker 模組及其對應的套件管理提供者，如下列範例所示：

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>您可以使用 `PS C:\> Get-PackageProvider -Name *Docker*` 來尋找用來安裝 Docker 的套件提供者

### <a name="clean-up-docker-data-and-system-components"></a>清除 Docker 資料和系統元件

卸載 Docker 之後，您必須移除 Docker 的預設網路，如此一來，在 Docker 消失之後，它們的設定就不會保留在您的系統上。 若要這麼做，您可以執行下列 Cmdlet：

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

執行下列 Cmdlet，從您的系統移除 Docker 的程式資料：

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

您可能也會想要移除與 Windows 上 Docker/容器相關聯的 Windows 選用功能。

這包括「容器」功能，在安裝 Docker 時，會在任何 Windows 10 或 Windows Server 2016 上自動啟用。 也可能包含 "Hyper-V" 功能，在已安裝 Docker 的 Windows 10 上，此功能會自動啟用，但在 Windows Server 2016 上必須明確啟用。

>[!IMPORTANT]
>[Hyper-v 功能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)是一般的虛擬化功能，不僅可提供容器。 停用 Hyper-v 功能之前，請確定您的系統上沒有其他需要 Hyper-v 的虛擬化元件。

若要移除 Windows 10 上的 Windows 功能：

- 移至 **控制台** ** > 程式 > ** **程式和功能** > **開啟或關閉 Windows 功能**。
- 尋找您要停用之功能或功能的名稱，在此案例中為 [**容器**] 和（選擇性） [ **hyper-v**]。
- 取消核取您想要停用的功能名稱旁的方塊。
- 選取 [**確定**]

若要移除 Windows Server 2016 上的 Windows 功能：

從提高許可權的 PowerShell 會話中，執行下列 Cmdlet 以停用您系統中的**容器**和（選擇性） **hyper-v**功能：

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>重新開機您的系統

若要完成卸載和清除，請從提升許可權的 PowerShell 會話執行下列 Cmdlet，以重新開機您的系統：

```powershell
Restart-Computer -Force
```
