---
title: 容器儲存體總覽
description: Windows Server 容器如何使用主機和其他儲存體類型
keywords: 容器, 磁碟區, 儲存體, 裝載, 繫結裝載
author: cwilhit
ms.openlocfilehash: f758877f1131813fe4637a01c03b49d7a18a83c4
ms.sourcegitcommit: db085db8a54664184a2f7cfa01d00598a1c66992
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/04/2020
ms.locfileid: "78288670"
---
# <a name="container-storage-overview"></a>容器儲存體總覽

<!-- Great diagram would be great! -->

本主題提供容器在 Windows 上使用存放裝置的不同方式概述。 容器在儲存體時的行為與虛擬機器不同。 根據本質，容器的建立是為了防止在其內執行的應用程式在主機的檔案系統上寫入狀態。 容器預設會使用「臨時」空間，但 Windows 也提供保存儲存體的方法。

## <a name="scratch-space"></a>臨時空間

Windows 容器預設會使用暫時儲存體。 所有容器 i/o 都會出現在「臨時空間」中，且每個容器都有自己的臨時。 檔案建立和檔案寫入會在臨時空間中捕捉，而不會對主機進行 escape。 當容器實例停止時，會擲回臨時區域中發生的所有變更。 啟動新的容器實例時，就會為實例提供新的臨時空間。

## <a name="layer-storage"></a>分層儲存體

如[容器總覽](../about/index.md)中所述，容器映射是以一系列圖層表示的檔案組合。 圖層儲存體是內建于容器中的所有檔案。 每次 `docker pull` 然後 `docker run` 該容器，兩者相同。

### <a name="where-layers-are-stored-and-how-to-change-it"></a>分層儲存的位置，以及如何變更

在預設安裝中，分層儲存在 `C:\ProgramData\docker` 中並分散至 "image" 和 "windowsfilter" 目錄中。 您可以使用 `docker-root` 組態來變更分層儲存的位置，如 [Windows 上的 Docker 引擎](../manage-docker/configure-docker-daemon.md)文件所示範。

> [!NOTE]
> 分層儲存體僅支援 NTFS， 不支援 ReFS。

您不應該修改分層目錄中的任何檔案 - 使用以下命令謹慎管理檔案：

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker 載入](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>分層儲存體支援的作業

執行中的容器可使用除交易外的大部分 NTFS 作業， 包括設定 ACL，而且所有 ACL 都是在容器內檢查。 如果您想要在容器中以多個使用者執行處理序，您可以在 `Dockerfile` 中使用 `RUN net user /create ...` 建立使用者，設定檔案 ACL，然後使用 [Dockerfile USER 指示詞](https://docs.docker.com/engine/reference/builder/#user)，設定處理序以該使用者執行。

## <a name="persistent-storage"></a>持續性儲存體

Windows 容器支援透過「系結裝載」和「磁片區」來提供持續性儲存體的機制。 若要深入瞭解，請參閱[容器中的持續性儲存體](./persistent-storage.md)。

## <a name="storage-limits"></a>儲存體限制

Windows 應用程式的通用模式是先查詢可用磁碟空間量，再安裝或建立新的檔案，或做為清除暫存檔案的觸發程序。  以最大化應用程式相容性的目標而言，Windows 容器中的 C：磁片磁碟機代表一個虛擬可用大小 20 gb。

某些使用者可能會想要覆寫此預設值，並將可用空間設定為較小或較大的值。 這可以透過「儲存體-opt」設定內的「大小」選項來完成。

### <a name="examples"></a>範例

命令列： `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

或者，您可以直接變更 docker 設定檔：

```Docker Configuration File
"storage-opt": [
    "size=50GB"
  ]
```

> [!TIP]
> 此方法也適用于 docker build。 如需修改 Docker 組態檔的詳細資訊，請參閱[設定 Docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) 文件。
