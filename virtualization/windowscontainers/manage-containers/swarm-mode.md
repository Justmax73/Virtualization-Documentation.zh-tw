---
title: "開始使用群集模式"
description: "初始化群集叢集、建立覆疊的網路，並附加網路的服務。"
keywords: "Docker, 容器, 群集, 協調流程"
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
translationtype: Human Translation
ms.sourcegitcommit: f615c6dd268932a2ff99ac12c4e9ffdcf2cc217e
ms.openlocfilehash: ee6053003b31f226d2cfba8566f274ccc19d97ec
ms.lasthandoff: 03/02/2017

---

# 開始使用群集模式 

**重要事項︰***目前群集模式和覆疊網路功能支援僅提供給 [Windows 測試人員](https://insider.windows.com/)，作為即將推出之 Windows 10 Creators Update 的一部分。即將推出對其他 Windows 平台的支援。*

## 什麼是「群集模式」？
群集模式是一項 Docker 功能，其提供內建容器協調流程功能，包括 Docker 主機的原生叢集和容器工作負載的排程。 一組 Docker 主機在其 Docker 引擎於「群集模式」中一起執行時，即構成「群集」叢集。 如需群集模式的其他內容，請參閱 [Docker 的主要文件網站](https://docs.docker.com/engine/swarm/)。

## 管理員節點與背景工作節點
群集由兩種容器主機所組成︰*管理員節點*與*背景工作節點*。 每個群集皆透過管理員節點初始化，而所有用來控制及監視群集的 Docker CLI 命令皆必須從自身的其中一個管理員節點執行。 管理員節點可視作群集狀態的「看守員」，兩者共同構成共識群組，可維持對群集上執行之服務的狀態感知，且其負責確保群集的實際狀態一律符合開發人員或系統管理員所定義的預定狀態。 

>    **注意︰**任何給定的群集都可以有多個管理員節點，但必須一律*至少有一個*。 

背景工作節點由 Docker 群集透過管理員節點協調。 若要加入群集，背景工作節點必須使用群集初始化時由管理員節點產生的「加入權杖」。 背景工作節點僅需從管理員節點接受工作並予以執行，因此不需要 (也不擁有) 對群集狀態的感知。

## 群集模式系統需求

至少一個執行 **Windows 10 Creators Update** (適用於 [Windows 測試人員](https://insider.windows.com/)計畫的成員) 的實體或虛擬電腦系統 (若要使用群集的完整功能，建議使用至少兩個節點)，且設定為容器主機 (如需如何開始在 Windows 10 上使用 Docker 容器的詳細資料，請參閱[在 Windows 10 上的 Windows 容器](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10)主題)

**Docker 引擎 v1.13.0 或以上版本**

開啟連接埠︰每部主機皆必須可以使用下列連接埠。 在某些系統上，這些連接埠預設為開啟。
- 用於叢集管理通訊的 TCP 連接埠 2377
- 用於節點間通訊的 TCP 與 UDP 連接埠 7946
- 用於覆疊網路流量的 TCP 與 UDP 連接埠 4789

## 初始化群集叢集
若要初始化群集，只需從其中一部容器主機執行下列命令 (使用您主機電腦的本機 IPv4 位址來取代 \<HOSTIPADDRESS\>)：

```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
從給定的容器主機執行這個命令時，該主機上的 Docker 引擎會開始作為管理員節點以群集模式執行。

## 將節點新增至群集

> **注意︰**多個節點*不*需要利用群集模式及覆疊網路功能模式功能。 所有群集/覆疊功能都可與以群集模式執行 (即管理員節點，其使用 `docker swarm init` 命令置入群集模式) 的單一主機搭配使用。

### 將背景工作新增至群集
群集自管理員節點初始化後，即可使用另一項簡易命令，將其他主機作為背景工作新增至群集︰

```none
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

在此，\<MANAGERIPADDRESS\> 是群集管理員節點的本機 IP 位址，而 \<WORKERJOINTOKEN\> 是由自管理員節點執行的 `docker swarm init` 命令所提供作為輸出的背景工作加入權杖。 加入權杖也可使用另一方式取得，即在群集初始化後自管理員節點執行其中一項下列命令︰

```none
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### 將管理員新增至群集
其他管理員節點可使用下列命令新增至群集叢集︰

```none
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

同樣地，\<MANAGERIPADDRESS\> 是群集管理員節點的本機 IP 位址。 加入權杖 \<MANAGERJOINTOKEN\> 是群集的*管理員*加入權杖，可透過自現有管理員節點執行其中一項下列命令取得︰

```none
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## 建立覆疊網路

設定群集叢集後，即可在群集上建立覆疊網路。 覆疊網路可透過自群集管理員節點執行下列命令來建立︰

```none
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

在此，\<NETWORKNAME\> 是您想要提供給網路的名稱。

## 將服務部署到群集
覆疊網路建立後，即可建立服務並將其附加到網路。 網路使用下列語法建立︰

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

在此，\<SERVICENAME\> 是您想要提供給服務的名稱，您將使用該名稱透過服務探索 (其會使用 Docker 的原生 DNS 伺服器) 參考服務。 \<NETWORKNAME\> 是您想作為此服務連接目標的網路名稱 (例如，"myOverlayNet")。 \<CONTAINERIMAGE\> 是將會定義服務的容器影像名稱。

> **注意︰**需要此命令的第二個引數 `--endpoint-mode dnsrr`，才能指定至將會使用 DNS 循環配置資源原則 (以平衡服務容器端點間網路流量) 的 Docker 引擎。 DNS 循環配置資源是目前 Windows 唯一支援的平衡策略。尚不支援 Windows Docker 主機的[路由網格](https://docs.docker.com/engine/swarm/ingress/)，但很快就會推出。 現今，要尋找替代負載平衡策略的使用者可以設定外部負載平衡器 (例如 NGINX)，並使用群集的[發行連接埠模式](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)公開要進行負載平衡所透過的容器主機連接埠

## 縮放服務
服務部署到群集叢集後，組成該服務的容器執行個體會在叢集間部署。 根據預設，支援服務的容器執行個體數目 (即服務的「複本」或「工作」數目) 為一個。 不過，您可以使用 `docker service create` 命令的 `--replicas` 選項，或透過建立服務後加以縮放來透過多個工作建立服務。

服務延展性是 Docker 群集提供的主要益處，其也可透過單一 Docker 命令加以利用︰

```none
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

在此，\<SERVICENAME\> 是進行縮放的服務名稱，而 \<REPLICAS\> 是進行縮放之服務的工作或容器執行個體數目。

## 檢視群集狀態

有多個實用的命令可用來檢視群集及其上執行之服務的狀態。

### 列出群集節點
使用下列命令查看目前加入群集的節點清單，包括每個節點狀態的資訊。 必須自**管理員節點**執行這個命令。

```none
C:\ docker node ls
```

在這個命令的輸出中，您會發現其中一個節點以星號 (*) 標示；該星號僅用來表示目前節點，即 `docker node ls` 命令的執行來源節點。

### 列出網路
使用下列命令查看給定節點上存在的網路清單。 若要查看覆疊網路，必須自以群集模式執行的**管理員節點**執行此命令。

```none
C:\ docker network ls
```

### 列出服務
使用下列命令查看目前在群集上執行的服務清單，包括其狀態的資訊。

```none
C:\ docker service ls
```

### 列出定義服務的容器執行個體
使用下列命令，查看針對給定服務執行的容器執行個體詳細資料。 此命令的輸出包括每個容器的執行依據識別碼及節點，以及容器的狀態資訊。  

```none
C:\ docker service ps <SERVICENAME>
```

## 限制
目前，Windows 上的群集模式有以下限制︰
- 已知問題︰覆疊及群集模式目前僅受已連線至乙太網路的容器主機支援；**其不適用於連線至 Wi-Fi 的主機。** 我們正致力於修正這個問題。
- 不支援資料平面加密 (意即使用 `--opt encrypted` 選項的容器至容器流量)
- 尚不支援 Windows Docker 主機的[路由網格](https://docs.docker.com/engine/swarm/ingress/)，但很快就會推出。 現今，要尋找替代負載平衡策略的使用者可以設定外部負載平衡器 (例如 NGINX)，並使用群集的[發行連接埠模式](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)公開要進行負載平衡所透過的容器主機連接埠。  



