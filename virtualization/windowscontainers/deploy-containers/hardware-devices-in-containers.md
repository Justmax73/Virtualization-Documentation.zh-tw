---
title: 在 Windows 上的容器中的裝置
description: 支援何種裝置存在的 Windows 上的容器
keywords: docker，容器，裝置硬體
author: cwilhit
ms.openlocfilehash: f32ba3de347bcf968088d2f3f20f22f82166d652
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621556"
---
# <a name="devices-in-containers-on-windows"></a>在 Windows 上的容器中的裝置

根據預設，Windows 容器會提供給主機裝置： 就像是 Linux 容器的最少的存取權。 有特定的工作負載所在有幫助： 或甚至命令式-才能存取，以及與主機硬體裝置通訊。 本指南涵蓋在容器中支援哪些裝置，以及如何開始使用。

> [!IMPORTANT]
> 此功能需要的支援的 Docker 版本`--device`適用於 Windows 容器的命令列選項。 正式 Docker 支援已排程即將推出的 Docker EE 引擎 19.03 版本。 之前，適用於 Docker 的[上游來源](https://master.dockerproject.org/)會包含所需的位元。

## <a name="requirements"></a>需求

此功能才能運作，您的環境必須符合下列需求：
- 容器主機必須執行 Windows Server 2019 或 Windows 10 版本 1809年或更新版本。
- 您的容器基本映像版本必須是 1809年或更新版本。
- 您的容器都必須是在獨立處理序模式中執行的 Windows 容器。
- 19.03 或更新版本，容器主機必須執行 Docker 引擎。

## <a name="run-a-container-with-a-device"></a>執行容器與裝置

若要啟動容器與裝置，使用下列命令：

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

您必須取代`{interface class guid}`與適當[的裝置介面類別 GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)，可以找到下方的區段中。

若要啟動容器搭配多個裝置，使用下列命令和字串一起多個`--device`引數：

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

在 Windows 中，所有的裝置會宣告它們實作的介面類別的清單。 藉由 docker 傳遞此命令，它可確保所有裝置用來識別為要求的類別的實作，將會都連接到容器。

這表示您**要指派離主機裝置**。 相反地，主機與共用容器。 同樣地，因為您要指定類別的 GUID，將會與容器共用實作該 GUID 的_所有_裝置。

## <a name="what-devices-are-supported"></a>什麼裝置支援

下列的裝置 （和其裝置介面類別 Guid） 現今支援：
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>裝置類型</center></th>
<th><center>介面類別 GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C 匯流排</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM 連接埠</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI 匯流排</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU 加速</center></td>
<td><center>請參閱專用的文件</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> 上面所列的裝置是現今支援在 Windows 容器中的_僅_裝置。 嘗試傳遞任何其他類別 Guid，將會導致無法啟動容器。

## <a name="hyper-v-isolated-windows-container-support"></a>超 V 隔離 Windows 容器支援

不現今支援的裝置指派和 Hyper-v-V 隔離 Windows 容器中的工作負載的共用裝置。

## <a name="hyper-v-isolated-linux-container-support"></a>超 V 隔離的 Linux 容器支援

不現今支援的裝置指派和 Hyper-v-V 隔離的 Linux 容器中的工作負載的共用裝置。
