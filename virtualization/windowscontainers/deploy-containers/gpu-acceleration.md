---
title: Windows 容器中的 GPU 加速
description: Windows 容器中存在哪個層級的 GPU 加速
keywords: docker，容器，裝置，硬體
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909908"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Windows 容器中的 GPU 加速

針對許多容器化的工作負載，CPU 計算資源會提供足夠的效能。 不過，針對特定的工作負載類別，Gpu 所提供的大量平行計算能力（圖形處理單位）可以加速作業，依程度排序、降低成本並大幅改善輸送量。

Gpu 已經是許多熱門工作負載的常用工具，從傳統的轉譯和模擬，到機器學習訓練和推斷。 Windows 容器支援 DirectX 的 GPU 加速和以其為基礎的所有架構。

> [!NOTE]
> 這項功能適用于 Docker Desktop、版本2.1 和 Docker 引擎-Enterprise、19.03 或更新版本。

## <a name="requirements"></a>需求

若要讓這項功能正常執行，您的環境必須符合下列需求：

- 容器主機必須執行 Windows Server 2019 或 Windows 10 1809 版或更新版本。
- 容器基底映射必須是[mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windows)或更新版本。 目前不支援 Windows Server Core 和 Nano Server 容器映射。
- 容器主機必須執行 Docker Engine 19.03 或更新版本。
- 容器主機必須有執行顯示驅動程式 WDDM 2.5 或更新版本的 GPU。

若要檢查您的顯示器驅動程式 WDDM 版本，請在容器主機上執行 DirectX 診斷工具（dxdiag）。 在工具的 [顯示] 索引標籤中，查看 [驅動程式] 區段，如下所示。

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>使用 GPU 加速執行容器

若要使用 GPU 加速來啟動容器，請執行下列命令：

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX （以及在其之上建立的所有架構）是唯一可以使用 GPU 加速的 Api。 不支援協力廠商架構。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-隔離的 Windows 容器支援

目前不支援 Hyper-v 中的工作負載 GPU 加速-隔離的 Windows 容器。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-隔離的 Linux 容器支援

目前不支援 Hyper-v 中的工作負載 GPU 加速-隔離的 Linux 容器。

## <a name="more-information"></a>詳細資訊

如需利用 GPU 加速之容器化 DirectX 應用程式的完整範例，請參閱[DirectX 容器範例](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx)。
