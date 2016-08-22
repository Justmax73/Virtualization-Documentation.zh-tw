---
title: "在 Windows 中設定 Docker"
description: "在 Windows 中設定 Docker"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 08/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: fac57150de3ffd6c7d957dd628b937d5c41c1b35
ms.openlocfilehash: 7ba03dbcedbe42d54c955ff321e9f3f180a5a674

---

# Windows 上的 Docker 引擎

Docker 引擎和代理程式並未隨附於 Windows，且需要個別安裝及設定。 此外，Docker 引擎可接受許多自訂設定。 部分範例包括設定精靈接受連入要求的方式、預設網路功能選項，以及偵錯/記錄設定。 在 Windows 中，這些設定可以在設定檔中指定，或使用 Windows 服務控制管理員指定。 本文詳細說明如何安裝及設定 Docker 引擎，且提供常用設定的範例。

## 安裝 Docker

需要先安裝 Docker，才能搭配使用 Windows 容器。 Docker 是由 Docker 引擎及 Docker 用戶端所組成。 此演練將會安裝這兩者。

建立 Docker 可執行檔的資料夾。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

下載 Docker 引擎。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

下載 Docker 用戶端。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

將 Docker 目錄新增至系統路徑。 完成時，重新啟動 PowerShell 工作階段，以便識別修改過的路徑。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

若要將 Docker 安裝為 Windows 服務，請執行下列命令。

```none
dockerd --register-service
```

安裝之後，就可以啟動服務。

```none
Start-Service Docker
```

必須先安裝容器映像，才能使用 Docker。 如需詳細資訊，請參閱＜[Manage Container Images](../management/manage_images.md)＞　(管理容器映像)。

## Docker 設定檔

建議使用設定檔在 Windows 上設定 Docker 引擎。 設定檔位於 'c:\ProgramData\docker\config\daemon.json'。 若尚未有此檔案，請加以建立。

請注意，並非所有可用的 Docker 設定選項都適用於 Windows 上的 Docker。 下列所示範例為適用的選項。 如需 Docker 引擎設定的完整文件 (包含適用於 Linux 的文件)，請參閱 [Docker Daemon]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/) (Docker 精靈)。

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

## 服務控制管理員

也可以透過使用 `sc config` 修改 Docker 服務來設定 Docker 引擎。 若使用此方法，會直接在 Docker 服務上設定 Docker 引擎旗標。


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## 一般設定

下列設定檔範例會顯示 Docker 的一般設定。 這些可以合併成單一設定檔。

### 建立預設網路 

若要設定 Docker 引擎而不建立預設 NAT 網路，請使用下列項目。 如需詳細資訊，請參閱 [Manage Docker Networks](../management/container_networking.md) (管理 Docker 網路)。

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

```none
[Environment]::SetEnvironmentVariable("HTTP_PROXY”, “http://username:password@proxy:port/”, [EnvironmentVariableTarget]::Machine)
```

完成變數設定之後，請重新啟動 Docker 服務。

```none
restart-service docker
```

如需詳細資訊，請參閱 [Docker.com 上的精靈通訊端選項](https://docs.docker.com/v1.10/engine/reference/commandline/daemon/#daemon-socket-option)。

## 收集記錄檔
Docker 引擎會將事件記錄至 Windows 應用程式事件記錄檔，而不是記錄至檔案。 您可以使用 Windows PowerShell，輕鬆讀取、排序和篩選這些記錄檔

比方說，這會顯示 Docker 引擎前 5 分鐘的記錄檔，並從最舊的開始排序。
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

您也可以輕鬆透過管道將記錄檔傳送至 CSV 檔案，以供其他工具或試算表讀取。
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.csv ```



<!--HONumber=Aug16_HO3-->


