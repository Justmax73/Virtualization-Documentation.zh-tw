---
title: 容器儲存概覽
description: Windows Server 容器如何使用主機和其他儲存體類型
keywords: 容器, 磁碟區, 儲存體, 裝載, 繫結裝載
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209748"
---
# <a name="container-storage-overview"></a>容器儲存概覽

<!-- Great diagram would be great! -->

本主題概要說明容器在 Windows 上使用儲存空間的不同方式。 在儲存空間時，容器的行為與虛擬機器不同。 根據本質，容器的建立是為了防止在其中執行的應用程式從主機的 filesystem 中，將狀態全部寫入。 容器預設會使用「暫存」空間，但 Windows 也提供保留儲存空間的方法。

## <a name="scratch-space"></a>暫存空間

Windows 容器預設會使用暫時儲存空間。 所有容器 i/o 都會在「暫存空間」中發生，而每個容器都會取得自己的劃痕。 檔案的建立和檔案寫操作會在暫存空間中捕獲，而且不會轉義給主機。 當容器實例停止時，會放棄所有在暫存空間中發生的變更。 啟動新的容器實例時，會為實例提供新的暫存空間。

## <a name="layer-storage"></a>分層儲存體

如[容器概述](../about/index.md)中所述，容器影像是以一系列的圖層表示的檔案套件。 [圖層儲存空間] 是所有內嵌在容器中的檔案。 每次 `docker pull` 然後 `docker run` 該容器，兩者相同。

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

## <a name="persistent-storage"></a>持久性儲存

Windows 容器支援透過 bind 裝載和卷來提供持久性儲存區的機制。 若要深入瞭解，請參閱[容器中的永久儲存空間](./persistent-storage.md)。

## <a name="storage-limits"></a>儲存空間限制

Windows 應用程式的通用模式是先查詢可用磁碟空間量，再安裝或建立新的檔案，或做為清除暫存檔案的觸發程序。  使用最大化應用程式相容性的目標，Windows 容器中的 C：磁片磁碟機代表的是 20 gb 的虛擬自由大小。

部分使用者可能會想要覆寫此預設值，並將自由空間設定為較小或較大的值。 這可以透過「儲存選擇」設定中的「大小」選項來完成。

### <a name="examples"></a>範例

命令列： `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

或者，您可以直接變更 docker 設定檔：

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> 這個方法也適用于 docker 建立。 如需修改 Docker 組態檔的詳細資訊，請參閱[設定 Docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) 文件。
