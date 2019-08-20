---
title: Windows 上的容器中的裝置
description: Windows 上的容器存在哪些裝置支援
keywords: docker、容器、裝置、硬體
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2019
ms.locfileid: "10045028"
---
# <a name="devices-in-containers-on-windows"></a>Windows 上的容器中的裝置

根據預設, Windows 容器會獲得對主機裝置的最低存取權, 就像 Linux 容器一樣。 在特定的工作負載中, 有好處 (甚至是強制性的) 可存取主機硬體裝置並與之進行通訊。 本指南涵蓋容器中支援哪些裝置, 以及如何開始使用。

## <a name="requirements"></a>需求

若要使用此功能, 您的環境必須符合下列需求:
- 容器主機必須執行 Windows Server 2019 或 Windows 10 (版本1809或更新版本)。
- 您的容器基底影像版本必須是1809或更新版本。
- 您的容器必須是在進程隔離模式下執行的 Windows 容器。
- 容器主機必須執行 Docker 引擎19.03 或更新版本。

## <a name="run-a-container-with-a-device"></a>在裝置上執行容器

若要使用裝置啟動容器, 請使用下列命令:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

您必須以適當`{interface class guid}`的[裝置介面類別別 GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)取代, 如下一節中所述。

若要使用多個裝置啟動容器, 請使用下列命令和字串將`--device`多個引數組成:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

在 Windows 中, 所有裝置都會宣告其所執行介面類別別的清單。 透過將此命令傳遞到 Docker, 就能確保識別為執行要求類別的所有裝置都將會查明容器。

這表示您**並未**將裝置指派給主機。 相反地, 主機會與容器共用它。 同樣地, 因為您要指定類別 GUID,_所有_實現該 GUID 的裝置都會與容器共用。

## <a name="what-devices-are-supported"></a>支援哪些裝置

目前支援下列裝置 (及其裝置介面類別別 Guid):
  
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
> 裝置支援是與驅動程式相關的。 嘗試傳遞沒有在上述資料表中定義的類別 Guid, 可能會導致未定義的行為。

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-v-隔離的 Windows 容器支援

目前不支援在 Hyper-v 中針對工作負載進行裝置指派和裝置共用。

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-v-隔離的 Linux 容器支援

目前不支援在 Hyper-v 中針對工作負載進行裝置指派和裝置共用。
