---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
---

# Windows Server 容器管理

<g id="1" ctype="x-strong">這是初版內容，後續可能會變更。</g>

容器的生命週期包含啟動、停止和移除容器等動作。 在執行這些動作時，您可能還必須擷取容器映像清單、管理容器網路功能，以及限制容器資源。 本文件將詳細說明使用 Docker 的基本容器管理工作，並也會連結至深入的說明文章。

## 管理容器

### 建立容器

使用 <g id="2" ctype="x-code">docker run</g>，以使用 Docker 建立容器。

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

如需 Docker <g id="2" ctype="x-code">run</g> 命令的詳細資訊，請參閱 <g id="4CapsExtId1" ctype="x-link"><g id="4CapsExtId2" ctype="x-linkText">Docker run reference</g><g id="4CapsExtId3" ctype="x-title"></g></g> (Docker run 參考)。

### 停止容器

使用 <g id="2" ctype="x-code">docker stop</g> 命令，以使用 Docker 停止容器。

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

若要使用 Docker 移除容器，請使用 <g id="2" ctype="x-code">docker rm</g> 命令。

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

如需 Docker rm 命令的詳細資訊，請參閱 <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Docker rm 參考</g><g id="2CapsExtId3" ctype="x-title"></g></g>。






<!--HONumber=Apr16_HO5-->


