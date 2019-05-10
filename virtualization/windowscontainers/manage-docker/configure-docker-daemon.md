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
ms.openlocfilehash: a04d356415e7bed84980747edc927cc1eaa1e7c1
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621086"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Docker 引擎及用戶端不是隨附於 Windows，需要進行個別安裝及設定。 此外，Docker 引擎可接受許多自訂設定。 部分範例包括設定精靈接受連入要求的方式、預設網路功能選項，以及偵錯/記錄設定。 在 Windows 中，這些設定可以在設定檔中指定，或使用 Windows 服務控制管理員指定。 本文詳細說明如何安裝及設定 Docker 引擎，且也會提供一些常用設定的範例。

## <a name="install-docker"></a>安裝 Docker

您需要 Docker 才能使用 Windows 容器。 Docker 是由 Docker 引擎(dockerd.exe) 及 Docker 用戶端 (docker.exe) 所組成。 若要取得已安裝的所有項目最簡單方式是快速入門指南，協助您取得所有項目設定並執行您的第一個容器中。

- [在 Windows Server 2019 的 Windows 容器](../quick-start/quick-start-windows-server.md)
- [在 Windows 10 上的 Windows 容器](../quick-start/quick-start-windows-10.md)

進行安裝，請參閱[使用指令碼來安裝 Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

您可以使用 Docker 之前，您將需要安裝容器映像。 如需詳細資訊，請參閱[使用映像快速入門指南](../quick-start/quick-start-images.md)。

## <a name="configure-docker-with-a-configuration-file"></a>使用設定檔設定 Docker

建議使用設定檔在 Windows 上設定 Docker 引擎。 設定檔位於 'C:\ProgramData\Docker\config\daemon.json'。 如果尚未存在，您可以建立此檔案。

>[!NOTE]
>並非所有可用的 Docker 設定選項適用於 Windows 上的 Docker。 下列範例示範套用的設定選項。 如需 Docker Engine 設定的詳細資訊，請參閱[Docker 精靈的組態檔](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

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

您只需要將所需的組態變更新增至設定檔。 例如，下列範例會設定 Docker 引擎設為接受連接埠 2375年上的連入連線。 其他所有設定選項將使用預設值。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同樣地，下列範例會設定 Docker 精靈將映像和容器保留在的替代路徑。 如果未指定，預設值是`c:\programdata\docker`。

```json
{    
    "data-root": "d:\\docker"
}
```

下列範例會設定 Docker 精靈將只接受透過連接埠 2376 連接的安全的連線。

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>在 Docker 服務設定 Docker

也可以藉由修改與 Docker 服務設定 Docker 引擎`sc config`。 若使用此方法，會直接在 Docker 服務上設定 Docker 引擎旗標。 在命令提示字元 (cmd.exe 而非 PowerShell) 執行下列命令︰

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>您不需要執行此命令，如果您的 daemon.json 檔案已包含`"hosts": ["tcp://0.0.0.0:2375"]`項目。

## <a name="common-configuration"></a>常見的設定

下列設定檔範例會顯示 Docker 的一般設定。 這些可以合併成單一設定檔。

### <a name="default-network-creation"></a>建立預設網路

若要設定 Docker 引擎，而它不會建立預設 NAT 網路，請使用下列設定。

```json
{
    "bridge" : "none"
}
```

如需詳細資訊，請參閱 [Manage Docker Networks](../container-networking/network-drivers-topologies.md) (管理 Docker 網路)。

### <a name="set-docker-security-group"></a>設定 Docker 安全性群組

當您已登入 Docker 主機並在本機執行 Docker 命令時，會透過具名管道執行這些命令。 依預設，只有系統管理員群組的成員可以透過具名管道存取 Docker 引擎。 若要指定具有此存取權的安全性群組，請使用 `group` 旗標。

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

## <a name="how-to-uninstall-docker"></a>如何解除安裝 Docker

本章節會告訴您如何解除安裝 Docker，並從 Windows 10 或 Windows Server 2016 系統執行 Docker 系統元件完全清除。

>[!NOTE]
>您必須在這些指示從提升權限的 PowerShell 工作階段中執行所有命令。

### <a name="prepare-your-system-for-dockers-removal"></a>準備系統以移除 Docker

解除安裝 Docker 之前，請確定您的系統上未執行任何容器。

執行下列 cmdlet 來檢查有執行中的容器：

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

它也是不錯的做法從系統移除所有容器、 容器映像、 網路及磁碟區，然後移除 Docker。 您可以執行下列 cmdlet:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>解除安裝 Docker

接下來，您將需要實際解除安裝 Docker。

若要解除安裝 Windows 10 上的 Docker

- 移至 [**設定** > Windows 10 電腦上的**應用程式**
- 在**應用程式 & 功能**] 下找到**Docker for Windows**
- 移至**適用於 Windows 的 Docker** > **解除安裝**

若要解除安裝 Windows Server 2016 上的 Docker:

從提升權限的 PowerShell 工作階段中，使用**解除安裝套件**和**解除安裝模組**cmdlet 來從您的系統移除 Docker 模組及其對應的套件管理提供者，如下列範例所示：

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>您可以找到您用來安裝 Docker 的套件提供者 `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>清理 Docker 資料與系統元件。

解除安裝 Docker 之後，您將需要移除 Docker 的預設網路，讓 Docker 消失後，您的系統上將不會維持其設定。 您可以執行下列 cmdlet:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

執行下列 cmdlet，從您的系統移除 Docker 的計畫資料：

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

您也可以移除與 Windows 上的 Docker/容器相關的 Windows 選用功能。

這包含 「 容器 」 功能，此功能會自動啟用任何 Windows 10 或 Windows Server 2016 已安裝 Docker。 也可能包含 "Hyper-V" 功能，在已安裝 Docker 的 Windows 10 上，此功能會自動啟用，但在 Windows Server 2016 上必須明確啟用。

>[!IMPORTANT]
>[HYPER-V 功能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)是一般的虛擬化功能的更多個只會啟用容器。 停用 HYPER-V 功能之前，請確定沒有任何其他虛擬化的元件在您的系統上需要 HYPER-V。

若要移除 Windows 10 上的 Windows 功能：

- 移至 [**控制台]** > **程式** > **程式和功能** > **開啟或關閉 Windows 功能**。
- 尋找的功能或您想要停用的功能名稱--在此案例中，**容器**，以及 （選擇性） **HYPER-V**。
- 取消選取您想要停用功能名稱旁邊的方塊。
- 選取 **「 確定 」**

若要移除 Windows Server 2016 上的 Windows 功能：

從提升權限的 PowerShell 工作階段中，執行下列 cmdlet 將停用**容器**和 （選擇性） 從您的系統的**HYPER-V**功能：

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>您的系統重新開機

若要完成解除安裝並清理，請從已提升權限的 PowerShell 工作階段重新啟動您的系統執行下列 cmdlet:

```powershell
Restart-Computer -Force
```
