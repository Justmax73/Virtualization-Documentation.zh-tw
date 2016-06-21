---
title: 容器資源管理
description: 使用 Windows 容器管理容器資源。
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b2192e64-9d74-474e-8af0-2d8b3ad1deee
---

# 容器資源管理

**這是初版內容，後續可能會變更。** 

Windows 容器能夠管理可使用的 CPU、磁碟 IO、網路和記憶體資源數量。 限制容器資源耗用量，可讓主機資源有效使用，並防止過度使用。 本文件將詳細說明如何使用 Docker 來管理容器資源。

## 使用 Docker 管理資源 

我們提供透過 Docker 來管理容器資源子集的功能。 具體來說，我們讓使用者能夠指定 CPU 在容器間的共用方式。 

### CPU

容器間的 CPU 共用可在執行階段透過 --cpu-shares 旗標來管理。 根據預設，所有容器會均分相等的 CPU 時間。 若要變更容器使用的相對 CPU 數量，請執行值從 1 到 10000 的 --cpu-shares 旗標。 根據預設，所有容器的權數皆為 5000。 如需 CPU 共用限制的詳細資訊，請參閱 [Docker Run Reference]( https://docs.docker.com/engine/reference/run/#cpu-share-constraint) (Docker Run 參考)。 

```none 
docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## 已知問題

- Hyper-V 容器目前尚不支援 CPU 和 IO 資源控制。
- 容器資料磁碟區目前尚不支援 IO 資源控制。

<!--HONumber=May16_HO3-->


