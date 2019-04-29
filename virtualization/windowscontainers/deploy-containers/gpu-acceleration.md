---
title: 在 Windows 容器中的 GPU 加速
description: GPU 加速的程度存在於 Windows 容器
keywords: docker，容器，裝置硬體
author: cwilhit
ms.openlocfilehash: 281241e07e4bc184e73c4e74a117b44253a775be
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578652"
---
# <a name="gpu-acceleration-in-windows-containers"></a>在 Windows 容器中的 GPU 加速

對於許多容器化工作負載，CPU 計算資源提供足夠的效能。 不過，某些類別的工作負載，Gpu （圖形處理單元） 所提供的大型平行運算電源可以加快作業大幅度，停機成本，以及區域性改進輸送量。

Gpu 已為許多常見的工作負載，從傳統的轉譯與機器學習訓練和推斷來模擬的常見工具。 Windows 容器支援 DirectX 和為基礎所建置它的所有架構的 GPU 加速。

> [!IMPORTANT]
> 此功能需要的支援的 Docker 版本`--device`適用於 Windows 容器的命令列選項。 正式 Docker 支援已排程即將推出的 Docker EE 引擎 19.03 版本。 之前，適用於 Docker 的[上游來源](https://master.dockerproject.org/)會包含所需的位元。

## <a name="requirements"></a>需求

此功能才能運作，您的環境必須符合下列需求：

- 容器主機必須執行 Windows Server 2019 或 Windows 10 版本 1809年或更新版本。
- 容器基本映像必須是[mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows)或更新版本。 目前不支援 Windows Server Core 與 Nano Server 容器映像。
- 19.03 或更新版本，容器主機必須執行 Docker 引擎。
- 容器主機必須有的 GPU 執行顯示器驅動程式版本 WDDM 2.5 或更新版本。

若要檢查您的顯示器驅動程式的 WDDM 版本，請在您的容器主機上執行 DirectX 診斷工具 (dxdiag.exe)。 在此工具的 「 顯示 」 索引標籤中，查看 「 驅動程式 」 區段如下所示。

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>搭配 GPU 加速執行容器

若要啟動容器搭配 GPU 加速，執行下列命令：

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX （和所有架構為基礎所建置它） 是現今的唯一 Api 可以加速 GPU 與。 不支援第 3 個廠商架構。

## <a name="hyper-v-isolated-windows-container-support"></a>超 V 隔離 Windows 容器支援

現今不支援 GPU 加速為超 V 隔離 Windows 容器的工作負載。

## <a name="hyper-v-isolated-linux-container-support"></a>超 V 隔離的 Linux 容器支援

現今不支援 GPU 加速為超 V 隔離的 Linux 容器的工作負載。
