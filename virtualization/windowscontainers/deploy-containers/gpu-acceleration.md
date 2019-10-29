---
title: Windows 容器中的 GPU 加速
description: Windows 容器中存在的 GPU 加速度層級
keywords: docker、容器、裝置、硬體
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: da762ce138467e50dce22d5086ad407138b38e48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/29/2019
ms.locfileid: "10261857"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 容器中的 GPU 加速

針對許多容器化的工作，CPU 計算資源提供足夠的效能。 不過，針對特定的工作負荷類別，Gpu （圖形處理單元）所提供的整體平行計算能力可以利用數量級來加速作業，增加成本並提升輸送量。

Gpu 已是許多常見工作負載的常見工具，從傳統轉譯和模擬到機器學習訓練與推斷。 Windows 容器支援 DirectX 的 GPU 加速度，以及所有以它為基礎的框架。

> [!NOTE]
> 此功能可在 Docker 桌面、版本2.1 和 Docker 引擎-企業版、版本19.03 或更新版本中使用。

## <a name="requirements"></a>需求

若要使用此功能，您的環境必須符合下列需求：

- 容器主機必須執行 Windows Server 2019 或 Windows 10 （版本1809或更新版本）。
- 容器基本影像必須是[mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windows)或更新版本。 目前不支援 Windows Server Core 和 Nano Server 容器影像。
- 容器主機必須執行 Docker 引擎19.03 或更新版本。
- 容器主機必須有一個 GPU 執行顯示驅動程式版本 WDDM 2.5 或更新版本。

若要檢查您的顯示驅動程式的 WDDM 版本，請在您的容器主機上執行 DirectX 診斷工具（dxdiag）。 在工具的 [顯示] 索引標籤中，查看如下所示的「驅動程式」一節。

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>使用 GPU 加速執行容器

若要使用 GPU 加速啟動容器，請執行下列命令：

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX （以及所有以它為基礎的框架）是唯一可以利用 GPU 來加速的 Api。 不支援協力廠商架構。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-隔離的 Windows 容器支援

Hyper-v 中工作負荷的 GPU 加速功能目前不支援在 Hyper-v 中隔離的 Windows 容器。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-隔離的 Linux 容器支援

Hyper-v 中工作負荷的 GPU 加速功能目前不支援在 Hyper-v 中隔離的 Linux 容器。

## <a name="more-information"></a>更多資訊

如需利用 GPU 加速的容器範例 DirectX app 的完整範例，請參閱[DirectX 容器範例](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx)。
