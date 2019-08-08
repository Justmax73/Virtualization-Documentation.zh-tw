---
title: Hyper-V 隔離
description: Explaination Hyper-v 隔離與處理常式隔離的容器有何不同。
keywords: Docker, 容器
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 092312848173102bec5a791f2c48fe8166e70d5f
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998325"
---
# <a name="hyper-v-isolation"></a>Hyper-V 隔離

Windows 容器技術包括容器、process 與 Hyper-v 隔離的兩個不同隔離層級。 這兩種類型都是以相同的方式建立、管理和功能。 而且也會產生及使用相同的容器映像。 兩者的差異在於，在容器、主機作業系統及該主機上執行的所有其他容器之間建立的隔離層級。

程式**隔離**–多個容器實例可以同時在主機上執行, 並透過命名空間、資源控制和進程隔離技術來提供隔離。  容器與主機共用同一個內核, 以及彼此共用。  這與在 Linux 上執行容器的方式大致相同。

**Hyper-v 隔離**-可在主機上併發執行多個容器實例, 不過, 每個容器都會在特殊的虛擬機器內執行。 這會在每個容器以及容器主機之間提供內核層級隔離。

## <a name="hyper-v-isolation-examples"></a>Hyper-v 隔離範例

### <a name="create-container"></a>建立容器

使用 Docker 管理 Hyper-v 隔離容器與管理 Windows Server 容器幾乎完全相同。 若要以 Hyper-v 隔離完全 Docker 來建立容器, 請使用`--isolation`參數加以設定。 `--isolation=hyperv`

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>隔離說明

這個範例示範 Windows Server 與 Hyper-v 隔離之間隔離功能的差異。

在這裡, 將會部署程式隔離的容器, 並將裝載長時間執行的 ping 程式。

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

相反地, 此範例也會啟動具有 ping 程式的 Hyper-v 獨立容器。

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
