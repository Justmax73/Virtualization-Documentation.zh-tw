---
title: Windows Server 容器儲存體
description: Windows Server 容器如何使用主機和其他儲存體類型
keywords: 容器, 磁碟區, 儲存體, 裝載, 繫結裝載
author: patricklang
ms.openlocfilehash: 5f8ff4b25ad4a4c34ed2e28683607cfc02891e1e
ms.sourcegitcommit: 62fff5436770151a28b6fea2be3a8818564f3867
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2019
ms.locfileid: "10147221"
---
# <a name="overview"></a>概觀

<!-- Great diagram would be great! -->


## <a name="layer-storage"></a>分層儲存體

這是所有內建在容器中的檔案。 每次 `docker pull` 然後 `docker run` 該容器，兩者相同。


### <a name="where-layers-are-stored-and-how-to-change-it"></a>分層儲存的位置，以及如何變更

在預設安裝中，分層儲存在 `C:\ProgramData\docker` 中並分散至 "image" 和 "windowsfilter" 目錄中。 您可以使用 `docker-root` 組態來變更分層儲存的位置，如 [Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)文件所示範。

> [!NOTE]
> 分層儲存體僅支援 NTFS， 不支援 ReFS。

您不應該修改分層目錄中的任何檔案 - 使用以下命令謹慎管理檔案：

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker load](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>分層儲存體支援的作業

執行中的容器可使用除交易外的大部分 NTFS 作業， 包括設定 ACL，而且所有 ACL 都是在容器內檢查。 如果您想要在容器中以多個使用者執行處理序，您可以在 `Dockerfile` 中使用 `RUN net user /create ...` 建立使用者，設定檔案 ACL，然後使用 [Dockerfile USER 指示詞](https://docs.docker.com/engine/reference/builder/#user)，設定處理序以該使用者執行。


## <a name="image-size"></a>映像大小
Windows 應用程式的通用模式是先查詢可用磁碟空間量，再安裝或建立新的檔案，或做為清除暫存檔案的觸發程序。  以最大化應用程式相容性為目標，Windows 容器中的 C: 磁碟機代表 20GB 的虛擬可用大小。  某些使用者可能會想要覆寫此預設值，將可用空間設定為較小或較大的值，這可以透過 “storage-opt” 組態中的 “size” 選項完成。

### <a name="examples"></a>範例
命令列： `docker run --storage-opt "size=50GB" microsoft/windowsservercore:1709 cmd`

Docker 組態檔
```
"storage-opts": [
    "size=50GB"
  ]
```
> 請注意，此方法適用於 Docker 組建。
如需修改 Docker 組態檔的詳細資訊，請參閱[設定 Docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) 文件。


## <a name="persistent-volumes"></a>永續性磁碟區

永續性儲存體可透過一些方式提供給容器：

- 繫結裝載
- 具名磁碟區

Docker 有如何[使用磁碟區](https://docs.docker.com/engine/admin/volumes/volumes/)的優良概觀，最好先閱讀該文件。 此頁面的其餘部分著重在 Linux 與 Windows 之間的差異，並提供 Windows 範例。


### <a name="bind-mounts"></a>繫結裝載

[繫結裝載](https://docs.docker.com/engine/admin/volumes/bind-mounts/)可讓容器與主機共用目錄。 如果您需要一個位置來儲存容器重新啟動時本機電腦上可用的檔案，或想要與多個容器分享位置，這項功能非常有用。 如果您想要容器在可存取相同檔案的多部電腦上執行，則請改用具名磁碟區或 SMB 裝載。

#### <a name="permissions"></a>權限

繫結裝載所使用的權限模型，根據容器的隔離層級而有所不同。

使用 **Hyper-V 隔離**的容器，包括 Windows Server (版本 1709) 上的 Linux 容器，使用簡單的唯讀或讀寫權限模型。
使用 `LocalSystem` 帳戶，存取主機上的檔案。 如果您在容器中存取遭拒，請確定 `LocalSystem` 有主機上該目錄的存取權。
使用唯讀旗標時，對容器內磁碟區所做的變更，對主機上的目錄而言是不可見或不保存的。

使用**處理序隔離**的 Windows Server 容器有些許不同，因為它們使用容器中的處理序身分識別來存取資料，這表示檔案 ACL 會生效。
容器內執行中處理序的身分識別 (根據預設，在 Windows Server Core 上為 "ContainerAdministrator"，在 Nano Server 容器上則為 "ContainerUser") 將用來存取裝載磁碟機中的檔案和目錄 (而不使用 `LocalSystem`)，並將需要授與存取權以使用資料。
因為這些身分識別只存在於容器內容中，而不在儲存檔案的主機，所以當您設定 ACL 以授與容器權限時，必須使用已知安全性群組例如 `Authenticated Users`。

> [!WARNING]
> 請勿將敏感的目錄 (例如 `C:\`) 繫結裝載至未受信任的容器。 這會讓後者變更主機上通常無法存取的檔案，可能會造成安全性缺口。

範例用法： 

- `docker run -v c:\ContainerData:c:\data:RO` 用於唯讀存取
- `docker run -v c:\ContainerData:c:\data:RW` 用於讀寫存取
- `docker run -v c:\ContainerData:c:\data` 用於讀寫存取 (預設)

#### <a name="symlinks"></a>符號連結

符號連結在容器中解析。 如果您將本身是符號連結或包含符號連結的主機路徑繫結裝載至容器，容器將無法存取它們。

#### <a name="smb-mounts"></a>SMB 裝載

在 Windows Server (版本 1709)，稱為「SMB 全域對應」的新功能可在主機上裝載 SMB 公用，然後將該共用上的目錄傳入容器。 容器不需要以特定伺服器、共用、使用者名稱或密碼設定 - 這一切都是在主機上處理。 容器就像擁有本機儲存體一樣運作。

##### <a name="configuration-steps"></a>設定步驟

1. 在容器主機上，全域對應遠端 SMB 共用：
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    這個命令會使用認證來透過遠端 SMB 伺服器進行驗證。 然後，將遠端共用路徑對應至 G: 磁碟機代號 (可以是任何其他可用的磁碟機代號)。 在此容器主機上建立的容器，現在可以將其讓資料磁碟區對應至 G: 磁碟機上的路徑。

    > [!NOTE]
    > 針對容器使用 SMB 全域對應時，容器主機上的所有使用者都可以存取遠端共用。 容器主機上執行的任何應用程式也都可以存取對應的遠端共用。

2. 建立容器並將其資料磁碟區對應至全域裝載的 SMB 共用  docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    在容器內，G:\AppData1 就會對應至遠端共用的 "ContainerData" 目錄。 容器內的應用程式可使用全域對應遠端共用上儲存的任何資料。 以相同的命令，多個容器可以取得此共用資料的讀取/寫入權。

這項 SMB 全域對應支援是 SMB 用戶端功能，可在包含以下的任何相容 SMB 伺服器上運作：

- 儲存空間直接存取 (S2D) 或傳統 SAN 上的向外延展檔案伺服器
- Azure 檔案服務 (SMB 共用)
- 傳統檔案伺服器
- SMB 通訊協定的協力廠商實作 (例如：NAS 設備)

> [!NOTE]
> 在 Windows Server (版本 1709)，SMB 全域對應不支援 DFS、DFSN、DFSR 共用。

### <a name="named-volumes"></a>具名磁碟區

具名磁碟區可讓您依名稱建立磁碟區、將它指派給容器，以及依相同名稱稍後重複使用它。 您不需要追蹤建立時的實際路徑，只要記住名稱。 Windows 上的 Docker 引擎有內建的具名磁碟區外掛程式，可以在本機電腦上建立磁碟區。 如果您想要在多部電腦上使用具名磁碟區，需要其他外掛程式。

範例步驟：

1. `docker volume create unwound` - 建立名稱為 'unwound' 的磁碟區
2. `docker run -v unwound:c:\data microsoft/windowsservercore` - 啟動容器並將磁碟區對應至 c:\data
3. 在容器中將一些檔案寫入至 c:\data，然後停止容器
4. `docker run -v unwound:c:\data microsoft/windowsservercore` - 啟動新容器
5. 在新容器中執行 `dir c:\data` - 檔案仍在那裡

> [!NOTE]
> Windows Server 會將目標路徑名（容器內的路徑）轉換成小寫;`-v unwound:c:\MyData`i. `-v unwound:/app/MyData`或在 linux 容器中，會導致在已映射（且已建立，如果不`c:\mydata`存在） `/app/mydata`的容器或 linux 容器內的目錄。
