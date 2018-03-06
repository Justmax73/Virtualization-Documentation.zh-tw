# <a name="linux-containers"></a>Linux 容器

這項功能會使用 [Hyper-V 隔離](../manage-containers/hyperv-container.md)來執行 Linux 核心，使其具備足夠的作業系統來支援容器。 建置此功能的 Windows 和 Hyper-V 變更開始於 _Windows 10 Fall Creators Update_ 和 _Windows Server (版本 1709)_，但是將此功能整合在一起也需要使用 Docker 技術建置所在的開放原始碼 [Moby project](https://www.github.com/moby/moby) 及 Linux 核心。 

![Linux 容器預覽影片](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

若要試用看看，您需要：

- Windows 10 或 Windows Server Insider Preview 組建 16267 或更新版本
- 根據 Moby 主要分支 (使用 `--experimental` 旗標執行) 的 Docker 精靈組建
- 您所選擇的相容 Linux 映像

我們有提供此預覽版的入門指南：

- [Docker Enterprise Edition 預覽版](https://blog.docker.com/2017/09/docker-windows-server-1709/)包含 LinuxKit 系統以及可執行 Linux 容器的 Docker EE 預覽版。 如需更多背景資訊，也請參閱[預覽版：Windows 上的 Linux 容器 (使用 LinuxKit)](https://go.microsoft.com/fwlink/?linkid=857061)
- [在 Windows 10 和 Windows Server 上執行具有 Hyper-V 隔離功能的 Ubuntu 容器](https://go.microsoft.com/fwlink/?linkid=857067)


## <a name="work-in-progress"></a>進行中的工作

您可以在 [GitHub](https://github.com/moby/moby/issues/33850) 上追蹤 Moby project 中的持續進度


### <a name="known-app-issues"></a>已知應用程式問題

所有應用程式都需要磁碟區對應，此對應具有[繫結裝載](#Bind-mounts)底下所涵蓋的一些限制。 它們將無法正確啟動或執行。

- MySQL
- PostgreSQL
- WordPress
- Jenkins
- MariaDB
- RabbitMQ


### <a name="bind-mounts"></a>繫結裝載

使用 `docker run -v ...` 的繫結裝載磁碟區會將檔案儲存在 Windows NTFS 檔案系統上，所以 POSIX 作業需要一些轉譯。 某些檔案系統作業目前只有一部分或是尚未實作，這可能會導致某些應用程式不相容。

這些作業目前並未針對繫結裝載磁碟區工作：

- MkNod
- XAttrWalk
- XAttrCreate
- Lock
- Getlock
- Auth
- Flush
- INotify

也有一些作業並未完整實作：

- GetAttr – Nlink 計數永遠回報為 2
- Open – 只會實作 ReadWrite、WriteOnly 和 ReadOnly 旗標
