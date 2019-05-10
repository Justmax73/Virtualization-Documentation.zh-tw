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
ms.openlocfilehash: b39ec17ac04995e8e1ce8795b5721df7a291e31c
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2019
ms.locfileid: "9620816"
---
# <a name="network-isolation-and-security"></a>網路隔離和安全性

## <a name="isolation-with-network-namespaces"></a>以網路命名空間隔離

每個容器端點會放在自己的__網路命名空間__中。 管理主機 vNIC 和主機網路堆疊位於預設的網路命名空間。 為了強制執行網路隔離在相同主機上的容器之間，為每個 Windows Server 容器建立網路命名空間，並在其中已安裝適用於容器的網路介面卡的 HYPER-V 隔離下執行的容器。 Windows Server 容器使用主機 vNIC 連結到虛擬交換器。 HYPER-V 隔離會使用綜合 VM NIC （不向公用程式 VM 公開） 連結到虛擬交換器。

![文字](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>網路安全性

根據所使用的容器和網路驅動程式，Windows 防火牆和 [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/) 的組合會強制執行連接埠 ACL。

### <a name="windows-server-containers"></a>Windows Server 容器

這些項目使用 Windows 主機的防火牆 (由網路命名空間啟用) 以及 VFP

* 預設輸出：全部允許
* 預設輸入：允許全部 (TCP、UDP、ICMP、IGMP) 未經要求的的網路流量
  * 拒絕所有其他不是來自這些通訊協定的網路流量

  >[!NOTE]
  >在 Windows Server 版本 1709年和 Windows 10 Fall Creators Update 之前, 預設輸入的規則是全部拒絕。 執行這些較舊版本的使用者可以建立輸入的允許規則與``docker run -p``（連接埠轉送）。

### <a name="hyper-v-isolation"></a>Hyper-V 隔離

在 HYPER-V 隔離執行的容器有自己的隔離的核心，因此使用下列設定執行它們自己的 Windows 防火牆執行個體：

* 預設在 Windows 防火牆 (在公用程式 VM 中執行) 和 VFP 中允許所有

![文字](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes pod

[Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)的基礎結構容器首次建立連接端點時。 屬於相同 pod，包括基礎結構和工作者容器，容器會共用通用網路命名空間 （相同的 IP 和連接埠空間）。

![文字](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>自訂預設連接埠 ACL

如果您想要修改預設連接埠 Acl，請先閱讀我們主機網路服務的文件 （連結即將推出）。 您將需要更新下列元件的原則：

>[!NOTE]
>對於透明和 NAT 模式中的 HYPER-V 隔離，您目前無法重新程式設計預設連接埠 Acl。 這在表格中會表示為「X」。

| 網路驅動程式 | WindowsServer 容器 | Hyper-V 隔離  |
| -------------- |-------------------------- | ------------------- |
| 透明 | Windows 防火牆 | X |
| NAT | Windows 防火牆 | X |
| L2Bridge | 兩者 | VFP |
| L2Tunnel | 兩者 | VFP |
| 重疊  | 兩者 | VFP |