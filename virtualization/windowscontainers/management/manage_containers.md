---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
translationtype: Human Translation
ms.sourcegitcommit: 2b85875eae1dcf1e50162e69c53dbf1ac7463450
ms.openlocfilehash: 8921cbd910bf657ddc4998e4214c1e9f9c3a01e9

---

# Windows Server 容器管理

**這是初版內容，後續可能會變更。** 

容器的生命週期包含啟動、停止和移除容器等動作。 在執行這些動作時，您可能還必須擷取容器映像清單、管理容器網路功能，以及限制容器資源。 本文件將詳細說明使用 Docker 的基本容器管理工作，並也會連結至深入的說明文章。 

## 管理容器

### 建立容器

使用 `docker run`，用 Docker 建立容器。

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

如需 Docker `run` 命令的詳細資訊，請參閱 [Docker run reference]( https://docs.docker.com/engine/reference/run/) (Docker run 參考)。

### 停止容器

使用 `docker stop` 命令，用 Docker 停止容器。

```none
PS C:\> docker stop tender_panini

tender_panini
```

此範例會使用 Docker 停止所有執行中的容器。

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### 移除容器

若要用 Docker 移除容器，請使用 `docker rm` 命令。

```none
PS C:\> docker rm prickly_pike

prickly_pike
``` 

若要使用 Docker 移除所有容器。

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

如需 Docker rm 命令的詳細資訊，請參閱 [Docker rm reference](https://docs.docker.com/engine/reference/commandline/rm/)。



<!--HONumber=Jun16_HO4-->


