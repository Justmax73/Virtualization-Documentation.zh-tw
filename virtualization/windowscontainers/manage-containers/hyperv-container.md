---
title: 隔離模式
description: 說明 Hyper-v 隔離與處理常式隔離的容器有何差異。
keywords: Docker, 容器
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161725"
---
# <a name="isolation-modes"></a>隔離模式

Windows 容器提供兩種不同的執行時間隔離`process`模式`Hyper-V` ：隔離。 在這兩種隔離模式下執行的容器，都會完全建立、管理且正常運作。 而且也會產生及使用相同的容器映像。 隔離模式的差異是在容器、主機作業系統及該主機上執行的所有其他容器之間建立的隔離程度。

## <a name="process-isolation"></a>進程隔離

這是容器的「傳統」隔離模式，也就是[Windows 容器概述](../about/index.md)中所述。 透過程式隔離，多個容器實例會在指定的主機上同時執行，並透過命名空間、資源控制和進程隔離技術來提供隔離。 在此模式下執行時，容器會與主機以及彼此共用同一個內核。  這與 Linux 容器的執行方式大致相同。

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V 隔離
這個隔離模式能在主機與容器版本之間提供增強的安全性和更廣泛的相容性。 使用 Hyper-v 隔離，在主機上併發執行多個容器實例;不過，每個容器都在高度優化的虛擬機器內執行，並有效地取得它自己的內核。 虛擬機器的存在會在每個容器以及容器主機之間提供硬體層級隔離。

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>隔離範例

### <a name="create-container"></a>建立容器

管理具有 Docker 的 Hyper-v 隔離容器與管理進程隔離的容器幾乎完全相同。 若要以 Hyper-v 隔離完全 Docker 來建立容器，請使用`--isolation`參數加以設定。 `--isolation=hyperv`

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

若要在進程隔離中建立包含完全 Docker 的容器`--isolation` ，請使用`--isolation=process`參數加以設定。

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

在 Windows Server 上執行的 windows 容器預設為使用程式隔離執行。 在 Windows 10 專業版和企業版上執行的 windows 容器預設為使用 Hyper-v 隔離執行。 從 Windows 10 年10月2018更新開始，執行 Windows 10 專業版或企業版主機的使用者可以使用進程隔離來執行 Windows 容器。 使用者必須使用`--isolation=process`標誌直接要求處理常式隔離。

> [!WARNING]
> 在 Windows 10 專業版和企業版上使用程式隔離來執行，都是為了進行開發/測試。 您的主機必須執行 Windows 10 組建 17763 +，而且您必須擁有具有引擎18.09 或更新版本的 Docker 版本。
> 
> 您應該繼續使用 Windows Server 作為生產部署的主機。 透過在 Windows 10 專業版和企業版上使用此功能，您也必須確定主機和容器版本標籤相符，否則容器可能無法啟動或顯示未定義的行為。

### <a name="isolation-explanation"></a>隔離說明

這個範例示範了 process 與 Hyper-v 隔離之間隔離功能的差異。

在這裡，會部署一個處理常式隔離的容器，並將裝載一個長時間執行的 ping 程式。

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
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

相反地，此範例也會啟動含 ping 程式的 Hyper-v solated 容器。

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

同樣地，`docker top` 可用來從容器傳回執行中處理序。

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

不過，在容器主機上搜尋進程時，找不到 ping 程式，並會引發錯誤。

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
