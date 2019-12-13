---
title: Windows 容器中的裝置
description: Windows 上的容器有哪些裝置支援
keywords: docker，容器，裝置，硬體
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910598"
---
# <a name="devices-in-containers-on-windows"></a>Windows 容器中的裝置

根據預設，Windows 容器會獲得最少的主機裝置存取權，就像 Linux 容器一樣。 有些工作負載有其優點，甚至是命令式，可以存取主機硬體裝置並與之通訊。 本指南涵蓋容器中支援的裝置，以及如何開始使用。

## <a name="requirements"></a>需求

若要讓這項功能正常執行，您的環境必須符合下列需求：
- 容器主機必須執行 Windows Server 2019 或 Windows 10 1809 版或更新版本。
- 您的容器基底映射版本必須是1809或更新版本。
- 您的容器必須是在進程隔離模式中執行的 Windows 容器。
- 容器主機必須執行 Docker Engine 19.03 或更新版本。

## <a name="run-a-container-with-a-device"></a>使用裝置執行容器

若要啟動具有裝置的容器，請使用下列命令：

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

您必須以適當的[裝置介面類別別 GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)取代 `{interface class guid}`，其可在下一節中找到。

若要啟動具有多個裝置的容器，請使用下列命令和字串搭配多個 `--device` 引數：

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

在 Windows 中，所有裝置都會宣告其所執行的介面類別別清單。 藉由將此命令傳遞至 Docker，將可確保識別為執行所要求類別的所有裝置都將被檢測到容器中。

這表示您**不**會將裝置指派給主機以外的位置。 而是由主機與容器共用。 同樣地，因為您要指定類別 GUID，所以_所有_執行該 GUID 的裝置都會與容器共用。

## <a name="what-devices-are-supported"></a>支援哪些裝置

目前支援下列裝置（及其裝置介面類別別 Guid）：
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>裝置類型</center></th>
<th><center>介面類別別 GUID</center></th>
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
<td><center>COM 埠</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI 匯流排</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU 加速</center></td>
<td><center>請參閱<a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU 加速</a>檔</center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> 裝置支援與驅動程式相關。 嘗試傳遞未在上表中定義的類別 Guid 可能會導致未定義的行為。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-隔離的 Windows 容器支援

目前不支援在 Hyper-v 隔離的 Windows 容器中，裝置指派和裝置共用工作負載。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-隔離的 Linux 容器支援

目前不支援 Hyper-v 中的工作負載的裝置指派和裝置共用（隔離的 Linux 容器）。
