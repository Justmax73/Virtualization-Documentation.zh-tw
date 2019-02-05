---
title: 在 Windows 上的容器中的裝置
description: 支援何種裝置存在的 Windows 上的容器
keywords: docker，容器，裝置硬體
author: cwilhit
ms.openlocfilehash: da9785b051826efa4bb2c64542a7c75a12ddd2b4
ms.sourcegitcommit: 4490d384ade48699e7f56dc265185dac75bf9d77
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/05/2019
ms.locfileid: "9058988"
---
**這是目前預覽材質。 請參閱第 4 個項目中的 '需求' 區段下方，如需詳細資訊。**

# <a name="devices-in-containers-on-windows"></a>在 Windows 上的容器中的裝置

根據預設，Windows 容器會提供給主機裝置： 就像是 Linux 容器的最少的存取權。 有特定的工作負載很有幫助： 或甚至命令式-才能存取，以及與主機硬體裝置通訊。 本指南涵蓋在容器中支援哪些裝置，以及如何開始使用。

## <a name="requirements"></a>需求

- 您必須執行 Windows Server 2019 或更新版本或 Windows 10 專業版/企業版與年 10 月 2018年更新
- 您的容器映像版本必須是 1809年或更新版本。
- 您的容器都必須是在獨立處理序模式中執行的 Windows 容器。
- 雖然 Windows 裝置功能存在於 Docker 精靈，它不尚未存在於 Docker 用戶端 （請參閱此[提取要求](https://github.com/docker/cli/pull/1606)來追蹤）。 您必須等到未來版本的 Docker for Windows / 以充分利用這項功能的 Docker EE 與此程式碼。 狀態變更時，將會更新本文件。

## <a name="run-a-container-with-a-device"></a>執行容器與裝置

若要與裝置啟動容器，使用下列命令：

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

您必須取代`{interface class guid}`使用適當的[裝置介面類別 GUID](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/overview-of-device-interface-classes)，可以找到下方的區段中。

若要開始搭配多個裝置的容器，使用下列命令和字串一起多個`--device`引數：

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

在 Windows 中，所有的裝置會宣告它們實作的介面類別的清單。 藉由 docker 傳遞此命令，它可確保所有裝置識別為要求的類別的實作，將會都連接到容器。

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
</tbody>
</table>

> [!TIP]
> 上面所列的裝置是現今支援在 Windows 容器中的_僅_裝置。 嘗試傳遞任何其他類別的 Guid，將會導致無法啟動容器。

## <a name="hyper-v-container-device-support"></a>HYPER-V 容器裝置支援

指派裝置和裝置共用不支援 HYPER-V 隔離容器現今。

## <a name="linux-containers-on-windows-lcow-device-support"></a>Windows (LCOW) 的裝置支援的 Linux 容器

指派裝置和裝置共用不支援在 LCOW 現今。
