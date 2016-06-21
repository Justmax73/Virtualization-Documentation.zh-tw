---
title: Hyper-V 容器
description: 部署 Hyper-V 容器。
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
---

# Hyper-V 容器

**這是初版內容，後續可能會變更。** 

Windows 容器技術包含兩種不同類型的容器，即 Windows Server 容器和 Hyper-V 容器。 這兩種容器的建立、管理和運作方式都相同。 而且也會產生及使用相同的容器映像。 兩者的差異在於，在容器、主機作業系統，以及所有在該主機上執行的其他容器之間建立的隔離層級。

**Windows Server 容器** – 主機上可以同時執行多個容器執行個體，但要透過命名空間、資源控制以及處理序隔離技術加以隔離。  Windows Server 容器與主機共用相同的核心，而且彼此共用。

**Hyper-V 容器** – 主機上可同時執行多個容器執行個體，但每個容器只在特殊的虛擬機器內執行。 如此可為每個 Hyper-V 容器與容器主機之間提供核心層級的隔離。

## Hyper-V 容器

### 建立容器

使用 Docker 管理 Hyper-V 容器的方式，與管理 Windows Server 容器幾乎完全相同。 以 Docker 建立 Hyper-V 容器時，會使用 `--isolation=hyperv` 參數。

```none
docker run -it --isolation=hyperv nanoserver cmd
```

### 隔離說明

本範例會區分 Windows Server 和 Hyper-V 容器之間的隔離功能。 

這裡部署了 Windows Server 容器，並將主控長時間執行的偵測處理序。

```none
docker run -d windowsservercore ping localhost -t
```

使用 `docker top` 命令時，會如同容器內所示傳回偵測處理序。 在此範例中處理序的識別碼為 3964。

```none
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

在容器主機上，`get-process` 命令可以用來從主機傳回任何正在執行的偵測處理序。 此範例中就有一個，而且處理序識別碼符合容器中的偵測處理序。 它是容器和主機顯示的相同處理序。

```none
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

作為對比，此範例同時會啟動具有偵測處理序的 Hyper-V 容器。 

```none
docker run -d --isolation=hyperv nanoserver ping -t localhost
```

同樣地，`docker top` 可用來從容器傳回執行中處理序。

```none
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

不過，搜尋容器主機上的處理序時，找不到偵測處理序，而且擲回錯誤。

```none
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

最後，在主機上會顯示 `vmwp` 處理序，這是執行中虛擬機器，它會封裝執行的容器並保護主機作業系統中執行的處理序。

```none
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```


<!--HONumber=Jun16_HO2-->


