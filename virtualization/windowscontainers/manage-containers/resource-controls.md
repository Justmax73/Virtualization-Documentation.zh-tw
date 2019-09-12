---
title: 實作資源控制項
description: 有關 Windows 容器適用之資源控制項的詳細資料
keywords: Docker, 容器, CPU, 記憶體, 磁碟, 資源
author: taylorb-microsoft
ms.date: 11/21/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: 3e9f7e3208222cd6c0f512c5f892453ac6e6980c
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107872"
---
# <a name="implementing-resource-controls-for-windows-containers"></a>實作 Windows 容器適用的資源控制項
可以針對各個容器和各項資源實作多個資源控制項。  根據預設，容器的執行受限於一般 Windows 資源管理，此管理方式通常是以公平共用為根據，但是經由這些控制項的設定，開發人員或管理員就可以限制或影響資源使用量。  可以控制的資源包含︰CPU/處理器、記憶體/RAM、磁碟/儲存空間和網路功能/輸送量。

Windows 容器會使用[工作物件](https://docs.microsoft.com/windows/desktop/ProcThread/job-objects)來群組和追蹤與每個容器相關聯的處理序。  資源控制項會實作在與容器相關聯的父系工作物件上。 

在 [Hyper-V 隔離](./hyperv-container.md)的情況下，資源控制項會自動同時套用到虛擬機器以及在虛擬機器內執行之容器的工作物件，如此可確保即便在容器中執行的處理序已略過或逸出工作物件控制項，虛擬機器也會確保它無法超出定義的資源控制項。

## <a name="resources"></a>資源
此部分提供了每項資源與 Docker 命令列介面之間的對應，做為如何將資源控制項用於 (可能會由協調器或其他工具設定) 對應的 Windows 主機運算服務 (HCS) API，以及 Windows 通常如何實作資源控制項 (請注意，這是概要描述，基本實作方式可能有所變更) 的示範。

|  | |
| ----- | ------|
| *記憶體* ||
| Docker 介面 | [--memory](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| HCS 介面 | [MemoryMaximumInMB](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共用的核心 | [JOB_OBJECT_LIMIT_JOB_MEMORY](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_basic_limit_information) |
| Hyper-V 隔離 | 虛擬機器記憶體 |
| _關於 Windows Server 2016 Hyper-V 隔離的注意事項：使用記憶體上限時，您會看到容器最初配置記憶體的上限數量，然後開始將它傳回至容器主機。  在較新版本 (1709 或更新版本)，這已最佳化。_ |
| ||
| *CPU (計數)* ||
| Docker 介面 | [--cpus](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 介面 | [ProcessorCount](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共用的核心 | 使用 [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)* 模擬 |
| Hyper-V 隔離 | 公開的虛擬處理器數量 |
| ||
| *CPU (百分比)* ||
| Docker 介面 | [--cpu-percent](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 介面 | [ProcessorMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共用的核心 | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Hyper-V 隔離 | 虛擬處理器上的 Hypervisor 限制 |
| ||
| *CPU (共用)* ||
| Docker 介面 | [--cpu-shares](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| HCS 介面 | [ProcessorWeight](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共用的核心 | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Hyper-V 隔離 | Hypervisor 虛擬處理器權重 |
| ||
| *儲存空間 (映像)* ||
| Docker 介面 | [--io-maxbandwidth/--io-maxiops](https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| HCS 介面 | [StorageIOPSMaximum 和 StorageBandwidthMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共用的核心 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Hyper-V 隔離 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| ||
| *儲存空間 (磁碟區)* ||
| Docker 介面 | [--storage-opt size=](https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| HCS 介面 | [StorageSandboxSize](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| 共用的核心 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Hyper-V 隔離 | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |

## <a name="additional-notes-or-details"></a>其他註解或詳細資料

### <a name="memory"></a>記憶體

Windows 容器會執行每個容器中的某個系統處理序，通常這些容器都會提供每個容器功能，例如使用者管理、網路等等， 這些處理序所需的大多數記憶體會在容器之間共用，因此記憶體上限必須夠高才能完成這些處理序。  [系統需求](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/system-requirements#memory-requirments)文件中有提供每個基本映像類型適用的表格，而且有含/不含 Hyper-V 隔離。

### <a name="cpu-shares-without-hyper-v-isolation"></a>CPU 共用 (不含 Hyper-V 隔離)

當使用 CPU 共用時，基礎實作 (不使用 Hyper-V 隔離時) 會設定 [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)，尤其會將控制項旗標設定為 JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED 並提供適當的權重。  工作物件的有效權重範圍為 1 – 9，預設值為 5，其精確度低於主機運算服務值 1 – 10000。  舉例來說，7500 共用權重會產生 7 的權重，或 2500 共用權重會產生 2 的值。
