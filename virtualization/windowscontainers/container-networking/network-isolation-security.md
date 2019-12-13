---
title: Windows 容器的網路功能
description: Windows 容器中的網路隔離和安全性。
keywords: docker, 容器
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: b39ec17ac04995e8e1ce8795b5721df7a291e31c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910588"
---
# <a name="network-isolation-and-security"></a>網路隔離和安全性

## <a name="isolation-with-network-namespaces"></a>與網路命名空間隔離

每個容器端點會放在自己的__網路命名空間__中。 管理主機 vNIC 和主機網路堆疊位於預設的網路命名空間。 為了在相同主機上的容器之間強制執行網路隔離，會為每個 Windows Server 容器建立網路命名空間，而容器會在 Hyper-v 隔離下執行，並在其中安裝容器的網路介面卡。 Windows Server 容器使用主機 vNIC 連結到虛擬交換器。 Hyper-v 隔離會使用綜合 VM NIC （不會公開至公用程式 VM）來連接至虛擬交換器。

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
  >在 Windows Server 之前，版本1709和 Windows 10 會秋季建立者更新，預設的輸入規則為全部拒絕。 執行這些舊版本的使用者可以使用 ``docker run -p`` （埠轉送）來建立輸入允許規則。

### <a name="hyper-v-isolation"></a>Hyper-V 隔離

在 Hyper-v 隔離中執行的容器有自己的隔離核心，因此會使用下列設定來執行自己的 Windows 防火牆實例：

* 預設在 Windows 防火牆 (在公用程式 VM 中執行) 和 VFP 中允許所有

![文字](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes pod

在[Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)中，會先建立一個已附加端點的基礎結構容器。 屬於相同 pod （包括基礎結構和背景工作角色容器）的容器會共用一個通用的網路命名空間（相同的 IP 和埠空間）。

![文字](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>自訂預設連接埠 ACL

如果您想要修改預設通訊埠 Acl，請先閱讀我們的主機網路服務檔（即將新增連結）。 您必須更新下列元件內的原則：

>[!NOTE]
>針對處於透明和 NAT 模式的 Hyper-v 隔離，您目前無法 mk-reprogram 預設通訊埠 Acl。 這在表格中會表示為「X」。

| 網路驅動程式 | Windows Server 容器 | Hyper-V 隔離  |
| -------------- |-------------------------- | ------------------- |
| 透明 | Windows 防火牆 | X |
| NAT | Windows 防火牆 | X |
| L2Bridge | 兩者 | VFP |
| L2Tunnel | 兩者 | VFP |
| Overlay  | 兩者 | VFP |