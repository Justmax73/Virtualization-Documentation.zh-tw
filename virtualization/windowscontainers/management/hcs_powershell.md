---
title: HCS PowerSWhell
description: 使用 HCS PowerShell 和 Windows 容器。
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 45144ec5-f76a-4460-abd1-9b60e47506d6
---

# 管理互通性

**這是初版內容，後續可能會變更。** 

## 顯示所有容器

若要傳回容器清單，請使用 `Get-ComputeProcess` 命令。

```none
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## 停止容器

若要停止容器 (無論是以 PowerShell 還是 Docker 建立的)，請使用 `Stop-ComputeProcess` 命令。

> 在寫入時，VMMS 服務將必須重新啟動，使容器能夠在使用 `Get-Container` 命令時顯示為已停止。

```none
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```


<!--HONumber=May16_HO3-->


