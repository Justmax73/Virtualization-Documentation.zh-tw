# 容器共用資料夾

**這是初版內容，後續可能會變更。**

共用資料夾可讓資料在容器主機與容器之間共用。 共用資料夾建立後，即可在容器內使用該共用資料夾。 任何從主機放入共用資料夾的資料，都將可在容器內使用。 任何從容器之中放入共用資料夾的資料，都將可在主機上使用。 主機上的單一資料夾可與多個容器共用，在此設定中，資料夾將可在執行中的容器之間共用。

## 管理資料 - PowerShell

### 建立共用資料夾

若要建立共用資料夾，請使用 `Add-ContainerSharedFolder` 命令。 下列範例會在容器中建立一個目錄 `c:\shared_data`，與主機上的目錄 `c:\data_source` 相對應。

> 容器在新增共用資料夾時，必須處於停止狀態。

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\data_source -DestinationPath c:\shared_data

ContainerName SourcePath       DestinationPath AccessMode
------------- ----------       --------------- ----------
DEMO          c:\data_source   c:\shared_data  ReadWrite
```

### 唯讀共用資料夾

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\sf1 -DestinationPath c:\sf2 -AccessMode ReadOnly

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\sf1     c:\sf2          ReadOnly
```

### 列出共用資料夾

若要檢視特定容器的共用資料夾清單，請使用 `Get-ContainerSharedFolder` 命令。

```powershell
PS C:\> Get-ContainerSharedFolder -ContainerName DEMO2

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\source  c:\source       ReadWrite
```

### 修改共用資料夾

若要修改現有的共用資料夾設定，請使用 `Set-ContainerSharedFolder` 命令。

```powershell
PS C:\> Set-ContainerSharedFolder -ContainerName SFRO -SourcePath c:\sf1 -DestinationPath c:\sf1
```

### 移除共用資料夾

若要移除共用資料夾，請使用 `Remove-ContainerSharedFolder` 命令。

> 在移除共用資料夾時，容器必須處於停止狀態

```powershell
PS C:\> Remove-ContainerSharedFolder -ContainerName DEMO2 -SourcePath c:\source -DestinationPath c:\source
```
## 管理資料 - Docker

### 裝載磁碟區

在使用 Docker 來管理 Windows 容器時，可使用 `-v` 選項來裝載磁碟區。

在下列範例中，來源資料夾為 c:\source，目的地資料夾為 c:\destination。

```powershell
PS C:\> docker run -it -v c:\source:c:\destination 1f62aaf73140 cmd
```

如需關於使用 Docker 管理容器資料的詳細資訊，請參閱 [Docker.com 上的 Docker 磁碟區](https://docs.docker.com/userguide/dockervolumes/)。





