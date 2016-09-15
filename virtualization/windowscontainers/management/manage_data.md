---
title: "容器的資料磁碟區"
description: "使用 Windows 容器建立和管理資料磁碟區。"
keywords: "docker, 容器"
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
redirect_url: https://docs.docker.com/engine/tutorials/dockervolumes/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 60dc25d879b5c8de6515899d9de9e920a08b7e62

---

# 容器的資料磁碟區

**這是初版內容，後續可能會變更。** 

在建立容器時，您可能需要建立新的資料目錄，或將現有的目錄加入容器中。 這可以透過新增資料磁碟區來完成。 資料磁碟區會在容器及容器主機上顯示，而且可以在兩者之間共用資料。 資料磁碟區也可以在相同容器主機上的多個容器之間共用。 本文將詳細說明如何建立、檢查和移除資料磁碟區。

## 資料磁碟區

### 建立新的資料磁碟區

使用 `docker run` 命令的 `-v` 參數建立新的資料磁碟區。 根據預設，新的資料磁碟區會儲存在 'c:\ProgramData\Docker\volumes' 下的主機。

這個範例會建立名為 'new-data-volume' 的資料磁碟區。 此資料磁碟區可以在 'c:\new-data-volume' 的執行中容器存取。

```none
docker run -it -v c:\new-data-volume windowsservercore cmd
```

如需有關建立磁碟區的詳細資訊，請參閱 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#data-volumes) (docker.com 上管理容器中的資料)。

### 掛接現有的目錄

除了建立新的資料磁碟區外，您可能會想要將主機的現有目錄傳遞至容器。 這也可以使用 `docker run` 命令的 `-v` 參數完成。 主機目錄內的所有檔案也都可以在容器中使用。 在掛接磁碟區中容器所建立的任何檔案也可在主機上使用。 可將相同的目錄掛接至許多容器。 在此設定中，容器之間可以共用資料。

在此範例中，來源目錄 'c:\source' 已作為 'c:\destination' 掛接至容器。

```none
docker run -it -v c:\source:c:\destination windowsservercore cmd
```

如需有關掛接主機目錄的詳細資訊，請參閱 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume) (docker.com 上管理容器中的資料)。

### 掛接單一檔案

單一檔案無法掛接至 Windows 容器。 執行下列命令並不會失敗，但是產生的容器不會包含檔案。 

```none
docker run -it -v c:\config\config.ini microsoft/windowsservercore cmd
```

因應措施：任何要掛接至容器的檔案，都需要從目錄進行掛接。

```none
docker run -it -v c:\config:c:\config microsoft/windowsservercore cmd
```

### 裝載完整的磁碟機

您可以使用類似下列所示的命令裝載完整的磁碟機。

```none
docker run -it -v d:\:d: windowsservercore cmd
```

此處無法執行裝載第二部磁碟機的步驟。 例如無法執行下列作業。

```none
docker run -it -v d:\source:d:\destination windowsservercore cmd
```

### 資料磁碟區容器

可以使用 `docker run` 命令的 `--volumes-from` 參數從其他正在執行的容器繼承資料磁碟區。 使用此繼承便可建立一個容器，其明確目的為主控容器化應用程式的資料磁碟區。 

本範例會從容器 ‘cocky_bell` 將資料磁碟區掛接至新的容器。 一旦啟動新的容器，此磁碟區中找到的資料將可供在容器中執行的應用程式使用。  

```none
docker run -it --volumes-from cocky_bell windowsservercore cmd
```

如需有關資料容器的詳細資訊，請參閱 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-file-as-a-data-volume) (docker.com 上管理容器中的資料)。

### 檢查共用的資料磁碟區

您可以使用 `docker inspect` 命令檢視掛接的磁碟區。

```none
docker inspect backstabbing_kowalevski
```

這會傳回容器的相關資訊，包括名為 ‘Mounts’ 的區段，其中包含掛接磁碟區的相關資料，例如來源和目的地目錄。

```none
"Mounts": [
    {
        "Source": "c:\\container-share",
        "Destination": "c:\\data",
        "Mode": "",
        "RW": true,
        "Propagation": ""
}
```

如需有關檢查磁碟區的詳細資訊，請參閱 [Manage data in containers on docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#locating-a-volume) (docker.com 上管理容器中的資料)。




<!--HONumber=Sep16_HO2-->


