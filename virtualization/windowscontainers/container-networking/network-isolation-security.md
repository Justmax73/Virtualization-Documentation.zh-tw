---
title: Windows 容器的網路功能
description: Windows 容器中的網路隔離和安全性。
keywords: Docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 7203989483cb07423b70ff8cc644f715ba4be274
ms.sourcegitcommit: ec186664e76d413d3bf75f2056d5acb556f4205d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "1876035"
---
# <a name="network-isolation-and-security"></a>網路隔離和安全性

## <a name="isolation-with-network-namespaces"></a>以網路命名空間進行隔離
每個容器端點會放在自己的__網路命名空間__中。 管理主機 vNIC 和主機網路堆疊位於預設的網路命名空間。 為了在相同主機上的容器之間強制執行網路隔離，系統會為每個 Windows Server 和 Hyper-V 容器 (其中已針對容器安裝網路介面卡) 建立網路命名空間。 Windows Server 容器使用主機 vNIC 連結到虛擬交換器。 Hyper-V 容器使用綜合 VM NIC (不向公用程式 VM 公開) 連結到虛擬交換器。


![文字](media/network-compartment-visual.png)


```powershell 
Get-NetCompartment
```

## <a name="network-security"></a>網路安全性
根據所使用的容器和網路驅動程式，Windows 防火牆和 [VFP](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/) 的組合會強制執行連接埠 ACL。

### <a name="windows-server-containers"></a>Windows Server 容器
這些項目使用 Windows 主機的防火牆 (由網路命名空間啟用) 以及 VFP
  * 預設輸出：全部允許
  * 預設輸入：允許全部 (TCP、UDP、ICMP、IGMP) 未經要求的的網路流量
    * 拒絕所有其他不是來自這些通訊協定的網路流量

  > 注意：在 Windows Server 1709 版本與 Windows 10 Fall Creators Update 之前，預設的*輸入*規則是全部拒絕。 執行這些較舊版本的使用者可以使用 ``docker run -p`` (連接埠轉送) 建立輸入允許規則


### <a name="hyper-v-containers"></a>Hyper-V 容器
Hyper-V 容器有自己的隔離核心，因此使用下列設定執行它們自己的 Windows 防火牆執行個體：
  * 預設在 Windows 防火牆 (在公用程式 VM 中執行) 和 VFP 中允許所有


![文字](media/windows-firewall-containers.png)


### <a name="kubernetes-pods"></a>Kubernetes Pod
在 [Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) 中，在連接端點時首次建立基礎結構容器。 屬於相同 pod 的容器 (包括基礎結構和工作者容器) 共用通用網路命名空間 (相同的 IP 和連接埠空間)。


![文字](media/pod-network-compartment.png)


### <a name="customizing-default-port-acls"></a>自訂預設連接埠 ACL
如果您想要修改預設連接埠 ACL，請參考我們的 HNS 文件 (連結即將推出)。 您將需要更新下列元件中的原則：

> 注意：對於透明和 NAT 模式中的 Hyper-V 容器，您目前無法重新程式設計預設連接埠 ACL。 這在表格中會表示為「X」。

| 網路驅動程式 | Windows Server 容器 | Hyper-V 容器  |
| -------------- |-------------------------- | ------------------- |
| 透明 | Windows 防火牆 | X |
| NAT | Windows 防火牆 | X |
| L2Bridge | 兩者 | VFP |
| L2Tunnel | 兩者 | VFP |
| 重疊  | 兩者 | VFP |