---
title: 容器中的持續性儲存體
description: Windows 容器如何能夠持續儲存
keywords: 容器, 磁碟區, 儲存體, 裝載, 繫結裝載
author: cwilhit
ms.openlocfilehash: 945a78d4ecb9c96da4de8f7246f84b6b444dd5b5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909668"
---
# <a name="persistent-storage-in-containers"></a>容器中的持續性儲存體

<!-- Great diagram would be great! -->

在某些情況下，應用程式必須能夠保存容器中的資料，或者您想要將檔案顯示在容器組建時間未包含的容器中。 持續性儲存體可透過幾種方式提供給容器：

- 繫結裝載
- 具名磁碟區

Docker 有如何[使用磁碟區](https://docs.docker.com/engine/admin/volumes/volumes/)的優良概觀，最好先閱讀該文件。 此頁面的其餘部分著重在 Linux 與 Windows 之間的差異，並提供 Windows 範例。

## <a name="bind-mounts"></a>繫結裝載

[繫結裝載](https://docs.docker.com/engine/admin/volumes/bind-mounts/)可讓容器與主機共用目錄。 如果您需要一個位置來儲存容器重新啟動時本機電腦上可用的檔案，或想要與多個容器分享位置，這項功能非常有用。 如果您想要容器在可存取相同檔案的多部電腦上執行，則請改用具名磁碟區或 SMB 裝載。

### <a name="permissions"></a>權限

繫結裝載所使用的權限模型，根據容器的隔離層級而有所不同。

使用**hyper-v 隔離**的容器會使用簡單的唯讀或讀寫許可權模型。 使用 `LocalSystem` 帳戶，存取主機上的檔案。 如果您在容器中存取遭拒，請確定 `LocalSystem` 有主機上該目錄的存取權。 使用唯讀旗標時，對容器內磁碟區所做的變更，對主機上的目錄而言是不可見或不保存的。

使用**進程隔離**的 Windows 容器會有些微不同，因為它們會使用容器內的進程識別來存取資料，這表示會接受檔案 acl。 容器內執行中處理序的身分識別 (根據預設，在 Windows Server Core 上為 "ContainerAdministrator"，在 Nano Server 容器上則為 "ContainerUser") 將用來存取裝載磁碟機中的檔案和目錄 (而不使用 `LocalSystem`)，並將需要授與存取權以使用資料。

由於這些身分識別只存在於容器的內容中（而不是儲存檔案的主機上），因此在設定 Acl 以授與容器的存取權時，您應該使用知名的安全性群組，例如 `Authenticated Users`。

> [!WARNING]
> 請勿將敏感的目錄 (例如 `C:\`) 繫結裝載至未受信任的容器。 這會讓後者變更主機上通常無法存取的檔案，可能會造成安全性缺口。

範例用法：

- 唯讀存取的 `docker run -v c:\ContainerData:c:\data:RO`
- 讀寫存取的 `docker run -v c:\ContainerData:c:\data:RW`
- 讀寫存取的 `docker run -v c:\ContainerData:c:\data` （預設值）

### <a name="symlinks"></a>符號連結

符號連結在容器中解析。 如果您將本身是符號連結或包含符號連結的主機路徑繫結裝載至容器，容器將無法存取它們。

### <a name="smb-mounts"></a>SMB 裝載

在 Windows Server 1709 版和更新版本上，稱為「SMB 全域對應」的功能可讓您在主機上掛接 SMB 共用，然後將該共用上的目錄傳遞至容器。 容器不需要以特定伺服器、共用、使用者名稱或密碼設定 - 這一切都是在主機上處理。 容器就像擁有本機儲存體一樣運作。

#### <a name="configuration-steps"></a>設定步驟

1. 在容器主機上，全域對應遠端 SMB 共用：
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    此命令將使用認證來向遠端 SMB 伺服器進行驗證。 然後，將遠端共用路徑對應至 G: 磁碟機代號 (可以是任何其他可用的磁碟機代號)。 在此容器主機上建立的容器，現在可以將其讓資料磁碟區對應至 G: 磁碟機上的路徑。

    > [!NOTE]
    > 使用容器的 SMB 全域對應時，容器主機上的所有使用者都可以存取遠端共用。 容器主機上執行的任何應用程式也都可以存取對應的遠端共用。

2. 建立容器並將其資料磁碟區對應至全域裝載的 SMB 共用  docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    在容器內，G:\AppData1 就會對應至遠端共用的 "ContainerData" 目錄。 容器內的應用程式可使用全域對應遠端共用上儲存的任何資料。 以相同的命令，多個容器可以取得此共用資料的讀取/寫入權。

這項 SMB 全域對應支援是 SMB 用戶端功能，可在包含以下的任何相容 SMB 伺服器上運作：

- 儲存空間直接存取 (S2D) 或傳統 SAN 上的向外延展檔案伺服器
- Azure 檔案服務 (SMB 共用)
- 傳統檔案伺服器
- SMB 通訊協定的協力廠商實作 (例如：NAS 設備)

> [!NOTE]
> 在 Windows Server (版本 1709)，SMB 全域對應不支援 DFS、DFSN、DFSR 共用。

## <a name="named-volumes"></a>具名磁碟區

具名磁碟區可讓您依名稱建立磁碟區、將它指派給容器，以及依相同名稱稍後重複使用它。 您不需要追蹤建立時的實際路徑，只要記住名稱。 Windows 上的 Docker 引擎有內建的具名磁碟區外掛程式，可以在本機電腦上建立磁碟區。 如果您想要在多部電腦上使用具名磁碟區，需要其他外掛程式。

範例步驟：

1. `docker volume create unwound`-建立名為「已展開」的磁片區
2. `docker run -v unwound:c:\data microsoft/windowsservercore`-啟動容器，並將磁片區對應至 c:\data
3. 在容器中將一些檔案寫入至 c:\data，然後停止容器
4. `docker run -v unwound:c:\data microsoft/windowsservercore`-啟動新的容器
5. 在新容器中執行 `dir c:\data` - 檔案仍在那裡

> [!NOTE]
> Windows Server 會將目標路徑（容器內的路徑）轉換成小寫;`-v unwound:c:\MyData`或 `-v unwound:/app/MyData` 在 Linux 容器中，會導致 `c:\mydata`容器內的目錄，或在 Linux 容器中 `/app/mydata`，並加以對應（如果不存在，則會建立）。
