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
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129258"
---
# <a name="docker-engine-on-windows"></a>Windows 上的 Docker 引擎

Docker 引擎和用戶端不會包含在 Windows 中，且需要個別安裝與設定。 此外，Docker 引擎可接受許多自訂設定。 部分範例包括設定精靈接受連入要求的方式、預設網路功能選項，以及偵錯/記錄設定。 在 Windows 中，這些設定可以在設定檔中指定，或使用 Windows 服務控制管理員指定。 這份檔詳細說明如何安裝及設定 Docker 引擎，也提供一些常用設定的範例。

## <a name="install-docker"></a>安裝 Docker

您需要 Docker 才能使用 Windows 容器。 Docker 是由 Docker 引擎(dockerd.exe) 及 Docker 用戶端 (docker.exe) 所組成。 您最簡單的方法就是在快速入門手冊中找到，這將協助您設定並執行您的第一個容器。

- [安裝 Docker](../quick-start/set-up-environment.md)

如需腳本安裝，請參閱[使用腳本來安裝 DOCKER EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)。

在您可以使用 Docker 之前，您必須安裝容器影像。 如需詳細資訊，請參閱[容器基本影像的](../manage-containers/container-base-images.md)檔。

## <a name="configure-docker-with-a-configuration-file"></a>使用設定檔設定 Docker

建議使用設定檔在 Windows 上設定 Docker 引擎。 設定檔位於 'C:\ProgramData\Docker\config\daemon.json'。 如果檔案尚不存在，您可以建立此檔案。

>[!NOTE]
>並非每個可用的 Docker 配置選項都適用于 Windows 上的 Docker。 下列範例顯示適用的設定選項。 如需 Docker 引擎設定的詳細資訊，請參閱[docker 守護程式設定檔](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)。

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

您只需要將所需的設定變更新增至設定檔。 例如，下列範例會將 Docker 引擎設定為接受埠2375上的傳入連線。 其他所有設定選項將使用預設值。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同樣地，下列範例會設定 Docker 守護程式，將影像和容器保持在替換路徑中。 如果沒有指定，則預設為`c:\programdata\docker`。

```json
{    
    "data-root": "d:\\docker"
}
```

下列範例將 Docker 守護程式設定為只接受經由埠2376的安全連線。

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

您也可以透過修改 Docker 服務的方式`sc config`來設定 docker 引擎。 若使用此方法，會直接在 Docker 服務上設定 Docker 引擎旗標。 在命令提示字元 (cmd.exe 而非 PowerShell) 執行下列命令︰

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>如果您的守護程式. json 檔案已包含該`"hosts": ["tcp://0.0.0.0:2375"]`專案，則不需要執行此命令。

## <a name="common-configuration"></a>常見配置

下列設定檔範例會顯示 Docker 的一般設定。 這些可以合併成單一設定檔。

### <a name="default-network-creation"></a>預設網路建立

若要設定 Docker 引擎，使其不會建立預設的 NAT 網路，請使用下列配置。

```json
{
    "bridge" : "none"
}
```

如需詳細資訊，請參閱 [Manage Docker Networks](../container-networking/network-drivers-topologies.md) (管理 Docker 網路)。

### <a name="set-docker-security-group"></a>設定 Docker 安全性群組

如果您已登入 Docker 主機，且正在本機執行 Docker 命令，這些命令會透過具名管道執行。 依預設，只有系統管理員群組的成員可以透過具名管道存取 Docker 引擎。 若要指定具有此存取權的安全性群組，請使用 `group` 旗標。

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

本節將說明如何從 Windows 10 或 Windows Server 2016 系統卸載 Docker 及執行 Docker 系統元件的完整清理。

>[!NOTE]
>您必須從提升的 PowerShell 會話中執行這些指令中的所有命令。

### <a name="prepare-your-system-for-dockers-removal"></a>針對 Docker 移除準備您的系統

在您卸載 Docker 前，請確定您的系統上沒有執行任何容器。

請執行下列 Cmdlet 來檢查執行中的容器：

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

在移除 Docker 之前，請先從您的系統中移除所有容器、容器影像、網路及卷，這也是一種很好的做法。 您可以執行下列 Cmdlet 來執行此動作：

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>解除安裝 Docker

接下來，您將需要實際卸載 Docker。

在 Windows 10 上卸載 Docker

- 移至您的 Windows 10 電腦上的 [**設定** > ]**應用程式**
- 在 [**應用程式 & 功能**] 底下，尋找**Windows 的 Docker**
- 移至**Windows 版視窗** > **卸載**的 Docker

若要在 Windows Server 2016 上卸載 Docker：

從提升許可權的 PowerShell 會話中，使用**卸載套件**和**卸載模組**Cmdlet，從您的系統中移除 Docker 模組及其對應的套件管理提供者，如下列範例所示：

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>您可以找到您用來安裝 Docker 的套件提供者 `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>清理 Docker 資料和系統元件

卸載 Docker 之後，您必須移除 Docker 的預設網路，以便在離開 Docker 之後，不會在系統上保留其設定。 您可以執行下列 Cmdlet 來執行此動作：

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

執行下列 Cmdlet 以從您的系統中移除 Docker 的程式資料：

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

您也可以移除與 Windows 上的 Docker/容器相關的 Windows 選用功能。

這包括「容器」功能，在安裝 Docker 時會自動在任何 Windows 10 或 Windows Server 2016 上啟用。 也可能包含 "Hyper-V" 功能，在已安裝 Docker 的 Windows 10 上，此功能會自動啟用，但在 Windows Server 2016 上必須明確啟用。

>[!IMPORTANT]
>[Hyper-v 功能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)是一般的虛擬化功能，可讓您的工作不僅僅是容器。 在停用 Hyper-v 功能前，請確定您的系統上沒有任何需要 Hyper-v 的虛擬化元件。

若要移除 Windows 10 上的 Windows 功能：

- 移至 [**控制台** > **程式** > ] 程式**和功能** > [**開啟或關閉 Windows 功能**]。
- 找出您想要停用的功能或功能名稱（在此案例中為 [**容器**] 和（選擇） [ **hyper-v**]）。
- 取消核取您要停用之功能之名稱旁的方塊。
- 選取 **[確定]**

若要移除 Windows Server 2016 上的 Windows 功能：

從提升許可權的 PowerShell 會話中，執行下列 Cmdlet 來停用您系統中的**容器**和（選擇性） **hyper-v**功能：

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>重新開機您的系統

若要完成卸載並清理，請從提升許可權的 PowerShell 會話中執行下列 Cmdlet，以重新開機您的系統：

```powershell
Restart-Computer -Force
```
