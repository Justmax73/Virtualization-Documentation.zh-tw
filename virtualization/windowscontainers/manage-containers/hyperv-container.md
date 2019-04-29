---
title: Hyper-V 隔離
description: Explaination 的 HYPER-V 隔離有何不同處理序隔離容器。
keywords: Docker, 容器
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 2ff2d1204e1f973d49af5e1d4441e4eacd946101
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576899"
---
# <a name="hyper-v-isolation"></a>Hyper-V 隔離

Windows 容器技術包含兩個不同的層級的隔離容器、 處理程序和 HYPER-V 隔離。 同時類型建立、 管理和運作方式都相同。 而且也會產生及使用相同的容器映像。 兩者的差異在於，在容器、主機作業系統及該主機上執行的所有其他容器之間建立的隔離層級。

**處理序隔離**– 執行個體的主機，使用隔離可同時執行多個容器提供透過命名空間、 資源控制以及處理序隔離技術。  容器與主機，以及彼此共用相同的核心。  這是大約相同方式容器在 Linux 上執行。

**HYPER-V 隔離**– 主機上可以同時執行多個容器執行個體，但每個容器執行在特殊的虛擬機器。 這會提供核心層級的隔離，每個容器，以及在容器主機之間。

## <a name="hyper-v-isolation-examples"></a>HYPER-V 隔離範例

### <a name="create-container"></a>建立容器

管理 HYPER-V 隔離容器與 Docker 是管理 Windows Server 容器幾乎完全相同。 若要建立容器使用 HYPER-V 隔離完整 Docker，使用`--isolation`參數來設定`--isolation=hyperv`。

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>隔離說明

此範例示範隔離功能在 Windows Server 和 HYPER-V 隔離之間的差異。

這裡的處理程序隔離的容器部署，並將主控長時間執行的偵測處理序。

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
```

使用 `docker top` 命令時，會如同容器內所示傳回偵測處理序。 在此範例中處理序的識別碼為 3964。

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

在容器主機上，`get-process` 命令可以用來從主機傳回任何正在執行的偵測處理序。 此範例中就有一個，而且處理序識別碼符合容器中的偵測處理序。 它是容器和主機顯示的相同處理序。

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

作為對比，此範例會啟動 HYPER-V 隔離的容器具有偵測處理序。

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 ping -t localhost
```

同樣地，`docker top` 可用來從容器傳回執行中處理序。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

不過，搜尋容器主機上的處理序時，找不到偵測處理序，而且擲回錯誤。

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

最後，在主機上會顯示 `vmwp` 處理序，這是執行中虛擬機器，它會封裝執行的容器並保護主機作業系統中執行的處理序。

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
