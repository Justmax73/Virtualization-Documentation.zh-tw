---
title: 開始使用群集模式
description: 初始化群集叢集、建立覆疊的網路，並附加網路的服務。
keywords: Docker, 容器, 群集, 協調流程
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
ms.openlocfilehash: 560e9ffc92728628268d7d557b8fa8428316c8ec
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909678"
---
# <a name="getting-started-with-swarm-mode"></a>開始使用群集模式 

## <a name="what-is-swarm-mode"></a>什麼是「群集模式」？
群集模式是一項 Docker 功能，其提供內建容器協調流程功能，包括 Docker 主機的原生叢集和容器工作負載的排程。 一組 Docker 主機在其 Docker 引擎於「群集模式」中一起執行時，即構成「群集」叢集。 如需群集模式的其他內容，請參閱 [Docker 的主要文件網站](https://docs.docker.com/engine/swarm/)。

## <a name="manager-nodes-and-worker-nodes"></a>管理員節點與背景工作節點
群集由兩種容器主機所組成︰*管理員節點*與*背景工作節點*。 每個群集皆透過管理員節點初始化，而所有用來控制及監視群集的 Docker CLI 命令皆必須從自身的其中一個管理員節點執行。 管理員節點可視作群集狀態的「看守員」，兩者共同構成共識群組，可維持對群集上執行之服務的狀態感知，且其負責確保群集的實際狀態一律符合開發人員或系統管理員所定義的預定狀態。 

>[!NOTE]
>任何指定的 swarm 都可以有多個 manager 節點，但一定要有*至少一個*。 

背景工作節點由 Docker 群集透過管理員節點協調。 若要加入群集，背景工作節點必須使用群集初始化時由管理員節點產生的「加入權杖」。 背景工作節點僅需從管理員節點接受工作並予以執行，因此不需要 (也不擁有) 對群集狀態的感知。

## <a name="swarm-mode-system-requirements"></a>群集模式系統需求

至少要有一部實體或虛擬電腦系統（若要使用至少兩個節點的 swarm 完整功能），執行**windows 10 建立者更新**或**windows server 2016** *與所有最新的更新\** 、安裝為容器主機[（如需](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-server)如何在[windows 10 上](https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10)開始使用 Docker 容器的詳細資訊，請參閱主題，

\***注意**： Windows Server 2016 上的 Docker Swarm 需要[KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)

**Docker Engine v 1.13.0 或更新版本**

開啟連接埠︰每部主機皆必須可以使用下列連接埠。 在某些系統上，這些連接埠預設為開啟。
- 用於叢集管理通訊的 TCP 連接埠 2377
- 用於節點間通訊的 TCP 與 UDP 連接埠 7946
- 用於覆疊網路流量的 UDP 連接埠 4789

## <a name="initializing-a-swarm-cluster"></a>初始化群集叢集

若要初始化群集，只需從其中一部容器主機執行下列命令 (使用您主機電腦的本機 IPv4 位址來取代 \<HOSTIPADDRESS\>)：

```
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
從給定的容器主機執行這個命令時，該主機上的 Docker 引擎會開始做為管理員節點以群集模式執行。

## <a name="adding-nodes-to-a-swarm"></a>將節點新增至群集

您*不*需要多個節點來運用 swarm 模式和重迭網路模式功能。 所有群集/覆疊功能都可與以群集模式執行 (即管理員節點，其使用 `docker swarm init` 命令置入群集模式) 的單一主機搭配使用。

### <a name="adding-workers-to-a-swarm"></a>將背景工作新增至群集

群集自管理員節點初始化後，即可使用另一項簡易命令，將其他主機做為背景工作新增至群集︰

```
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

在此，\<MANAGERIPADDRESS\> 是群集管理員節點的本機 IP 位址，而 \<WORKERJOINTOKEN\> 是由自管理員節點執行的 `docker swarm init` 命令所提供作為輸出的背景工作加入權杖。 加入權杖也可使用另一方式取得，即在群集初始化後自管理員節點執行其中一項下列命令︰

```
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### <a name="adding-managers-to-a-swarm"></a>將管理員新增至群集
若要將其他管理員節點新增至群集叢集，可使用下列命令︰

```
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

同樣地，\<MANAGERIPADDRESS\> 是群集管理員節點的本機 IP 位址。 加入權杖 \<MANAGERJOINTOKEN\> 是群集的*管理員*加入權杖，可透過自現有管理員節點執行其中一項下列命令取得︰

```
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## <a name="creating-an-overlay-network"></a>建立覆疊網路

設定群集叢集後，即可在群集上建立覆疊網路。 若要建立覆疊網路，可以從群集管理員節點執行下列命令︰

```
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

在此，\<NETWORKNAME\> 是您想要提供給網路的名稱。

## <a name="deploying-services-to-a-swarm"></a>將服務部署到群集
覆疊網路建立後，即可建立服務並將其連接至網路。 建立服務的語法如下︰

```
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

在此，\<SERVICENAME\> 是您想要提供給服務的名稱，您將使用該名稱透過服務探索 (其會使用 Docker 的原生 DNS 伺服器) 參考服務。 \<NETWORKNAME\> 是您想要用來連接此服務的網路名稱（例如，"myOverlayNet"）。 \<INSTALL-CONTAINERIMAGE\> 是將定義服務的容器映射名稱。

>[!NOTE]
>此命令的第二個引數（`--endpoint-mode dnsrr`）是指定給 Docker 引擎的必要條件，DNS 迴圈配置資源原則將用來平衡服務容器端點間的網路流量。 目前，DNS 迴圈配置資源是 Windows Server 2016 上唯一支援的負載平衡策略。Windows Server 2019 （和更新版本）支援 Windows docker 主機的[路由網格](https://docs.docker.com/engine/swarm/ingress/)，但 windows server 2016 上則不支援。 目前在 Windows Server 2016 上尋求替代負載平衡策略的使用者可以設定外部負載平衡器（例如 NGINX），並使用 Swarm 的[發佈埠模式](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)來公開要平衡流量的容器主機埠。

## <a name="scaling-a-service"></a>調整服務
服務部署到群集叢集後，組成該服務的容器執行個體會在叢集間部署。 根據預設，支援服務的容器執行個體數目 (即服務的「複本」或「工作」數目) 為一個。 不過，您可以使用 `docker service create` 命令的 `--replicas` 選項建立具有多個工作的服務，或是在建立服務之後再來調整服務。

服務延展性是 Docker 群集提供的主要益處，其也可透過單一 Docker 命令加以利用︰

```
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

在此，\<SERVICENAME\> 是進行縮放的服務名稱，而 \<REPLICAS\> 是進行縮放之服務的工作或容器執行個體數目。


## <a name="viewing-the-swarm-state"></a>檢視群集狀態

有多個實用的命令可用來檢視群集及其上執行之服務的狀態。

### <a name="list-swarm-nodes"></a>列出群集節點
使用下列命令查看目前加入群集的節點清單，包括每個節點狀態的資訊。 必須自**管理員節點**執行這個命令。

```
C:\> docker node ls
```

在這個命令的輸出中，您會發現其中一個節點以星號 (*) 標示；該星號僅用來表示目前節點，即 `docker node ls` 命令的執行來源節點。

### <a name="list-networks"></a>列出網路
使用下列命令查看給定節點上存在的網路清單。 若要查看覆疊網路，必須自以群集模式執行的**管理員節點**執行此命令。

```
C:\> docker network ls
```

### <a name="list-services"></a>列出服務
使用下列命令查看目前在群集上執行的服務清單，包括其狀態的資訊。

```
C:\> docker service ls
```

### <a name="list-the-container-instances-that-define-a-service"></a>列出定義服務的容器執行個體
使用下列命令，查看針對給定服務執行的容器執行個體詳細資料。 此命令的輸出包括每個容器的執行依據識別碼及節點，以及容器的狀態資訊。  

```
C:\> docker service ps <SERVICENAME>
```
## <a name="linuxwindows-mixed-os-clusters"></a>Linux + Windows 混合作業系統叢集

最近我們的小組成員張貼了一個簡短的三部分示範，說明如何使用 Docker 群集來設定 Windows+Linux 混合作業系統應用程式。 如果您是 Docker 群集的新手，或是要用它來執行混合作業系統應用程式，這都是很棒的起始點。 快來一探究竟：
- [使用 Docker Swarm 執行 Windows + Linux 容器化應用程式（第1/3 部分）](https://www.youtube.com/watch?v=ZfMV5JmkWCY&t=170s)
- [使用 Docker Swarm 執行 Windows + Linux 容器化應用程式（第2/3 部分）](https://www.youtube.com/watch?v=VbzwKbcC_Mg&t=406s)
- [使用 Docker Swarm 執行 Windows + Linux 容器化應用程式（第3/3 部分）](https://www.youtube.com/watch?v=I9oDD78E_1E&t=354s)

### <a name="initializing-a-linuxwindows-mixed-os-cluster"></a>初始化 Linux + Windows 混合作業系統叢集
初始化混合作業系統叢集很簡單，只要您的防火牆規則設定得當，而且您的主機可以互相存取，就只需要使用標準 `docker swarm join` 命令，即可將 Linux 主機新增至群集：
```
C:\> docker swarm join --token <JOINTOKEN> <MANAGERIPADDRESS>
```
您也可以使用從 Windows 主機初始化群集時所執行的相同命令，從 Linux 主機初始化群集：
```
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```

### <a name="adding-labels-to-swarm-nodes"></a>新增標籤至群集節點
若要將 Docker 服務啟動至混合作業系統群集叢集，必須要有方法可以區分哪些群集節點是執行該服務設計所針對的作業系統，而哪些不是。 [Docker 物件標籤](https://docs.docker.com/engine/userguide/labels-custom-metadata/)提供很好用的方式來標示節點，以方便建立及設定服務，使其只在符合其作業系統的節點上執行。 

>[!NOTE]
>[Docker 物件標籤](https://docs.docker.com/engine/userguide/labels-custom-metadata/)可以用來將中繼資料套用至各種 docker 物件（包括容器映射、容器、磁片區和網路）和適用于各種用途（例如標籤可以用來分隔應用程式的「前端」和「後端」元件，方法是允許前端微服務只能在「前端」標示的節點和後端 mircoservices 上 secheduled，只在「後端」標記的節點上排程）。 在此案例中，我們在節點上使用標籤來區分 Windows 作業系統節點和 Linux 作業系統節點。

若要標示您現有的群集節點，請使用下列的語法：

```
C:\> docker node update --label-add <LABELNAME>=<LABELVALUE> <NODENAME>
```

在這裡，`<LABELNAME>` 是您要建立的標籤名稱，例如，在此案例中，我們是依作業系統來區分節點，因此標籤的邏輯名稱可以用 "os"。 `<LABELVALUE>` 是標籤的值，在此案例中，您可以選擇使用 "windows" 和 "linux" 的值。 (當然，您可以為您的標籤和標籤值選擇任何名稱，只要保持一致即可)。 `<NODENAME>` 是您要加上標籤的節點名稱;您可以藉由執行 `docker node ls`，提醒自己的節點名稱。 

**例如**，如果您的叢集中有四個群集節點，包括兩個 Windows 節點和兩個 Linux 節點，您的標籤更新命令看起來會像這樣：

```
# Example -- labeling 2 Windows nodes and 2 Linux nodes in a cluster...
C:\> docker node update --label-add os=windows Windows-SwarmMaster
C:\> docker node update --label-add os=windows Windows-SwarmWorker1
C:\> docker node update --label-add os=linux Linux-SwarmNode1
C:\> docker node update --label-add os=linux Linux-SwarmNode2
```

### <a name="deploying-services-to-a-mixed-os-swarm"></a>將服務部署至混合作業系統群集
群集節點有了標籤之後，要將服務部署至叢集就很簡單了；只要在 [`docker service create`](https://docs.docker.com/engine/reference/commandline/service_create/) 命令中使用 `--constraint` 選項即可：

```
# Deploy a service with swarm node constraint
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> --constraint node.labels.<LABELNAME>=<LABELVALUE> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

例如，使用上述範例的標籤和標籤值命名方式時，用來建立服務的一組命令 (一個用於 Windows 的服務，一個用於 Linux 的服務) 看起來會像這樣：

```
# Example -- using the 'os' label and 'windows'/'linux' label values, service creation commands might look like these...

# A Windows service
C:\> docker service create --name=win_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==windows' microsoft/nanoserver:latest powershell -command { sleep 3600 }

# A Linux service
C:\> docker service create --name=linux_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==linux' redis
```

## <a name="limitations"></a>限制
目前，Windows 上的群集模式有以下限制︰
- 不支援資料層加密 (亦即使用 `--opt encrypted` 選項的容器至容器流量)
- Windows Server 2016 不支援 Windows docker 主機的[路由網狀](https://docs.docker.com/engine/swarm/ingress/)，而是只從 windows server 2019 開始。 現今，要尋找替代負載平衡策略的使用者可以設定外部負載平衡器 (例如 NGINX)，並使用群集的[發行連接埠模式](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)公開要進行負載平衡所透過的容器主機連接埠。 以下提供更多詳細資料。

 >[!NOTE]
>如需有關如何設定 Docker Swarm 路由網格的詳細資訊，請參閱這[篇 blog 文章](https://docs.microsoft.com/en-us/virtualization/community/team-blog/2017/20170926-docker-s-routing-mesh-available-with-windows-server-version-1709)

## <a name="publish-ports-for-service-endpoints"></a>為服務端點發佈連接埠
 想要為其服務端點發佈埠的使用者，可以使用發佈埠模式或 Docker Swarm 的[路由網格](https://docs.docker.com/engine/swarm/ingress/)功能來立即執行此動作。 

若要為定義服務的每個工作/容器端點發佈主機連接埠，請在 `docker service create` 命令中使用 `--publish mode=host,target=<CONTAINERPORT>` 引數：

```
# Create a service for which tasks are exposed via host port
C:\ > docker service create --name=<SERVICENAME> --publish mode=host,target=<CONTAINERPORT> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

例如，下列命令會建立服務 's1'，其每個工作將會透過容器連接埠 80 和隨機選取的主機連接埠來公開。

```
C:\ > docker service create --name=s1 --publish mode=host,target=80 --endpoint-mode dnsrr web_1 powershell -command {echo sleep; sleep 360000;}
```

使用發佈連接埠模式來建立服務之後，即可查詢此服務來檢視每個服務工作的連接埠對應：

```
C:\ > docker service ps <SERVICENAME>
```
上述命令將會傳回針對服務執行之每個容器執行個體的詳細資料 (在所有群集主機上)。 輸出的其中一個資料行（[埠] 資料行）將會包含每個表單主機的通訊埠資訊 \<HOSTPORT\>->\<CONTAINERPORT\>/tcp。 每個容器實例的 \<HOSTPORT\> 值都會不同，因為每個容器都是在自己的主機埠上發行。


## <a name="tips--insights"></a>提示與深入解析 

#### <a name="existing-transparent-network-can-block-swarm-initializationoverlay-network-creation"></a>*現有的透明網路可以封鎖 swarm 初始化/重迭網路建立* 
在 Windows 中，overlay 和 transparent 網路驅動程式需要有外部 vSwitch 才能連結至 (虛擬) 主機網路介面卡。 建立覆疊網路之後，會建立新的交換器，然後將其連接到開放式網路介面卡。 透明網路模式也會使用主機網路介面卡。 同時，任何特定網路介面卡一次都只能連結至一個交換器；如果主機只有一個網路介面卡，則它一次只能連接至一個外部 vSwitch，無論該 vSwitch 是要用於覆疊網路還是透明網路。 

因此，如果容器主機只有一個網路介面卡，很可能會遇到透明網路阻止建立覆疊網路 (反之亦然) 的問題，因為透明網路目前正在占用主機唯一的虛擬網路介面。

有兩種方法可以避開這個問題：
- *選項 1 - 刪除現有的透明網路：* 在初始化群集之前，確定您的容器主機上沒有現存的透明網路。 刪除透明網路，以確定您的主機上有可用的虛擬網路介面卡可以用來建立覆疊網路。
- *選項 2 – 在您的主機上建立另一個 (虛擬) 網路介面卡：* 與其移除主機上的任何透明網路，您可以在主機上建立另一個網路介面卡，以用來建立覆疊網路。 若要這樣做，只要建立新的外部網路介面卡 (使用 PowerShell 或 Hyper-V 管理員) 即可；有了新的介面之後，當您的群集初始化時，主機網路服務 (HNS) 就會自動在您的主機上辨識該介面，並使用它來連結外部 vSwitch，以供建立覆疊網路。



