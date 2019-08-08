---
title: Windows 容器基礎映像歷程記錄
description: 已發行的 Windows 容器映像的清單，附 SHA256 層雜湊
keywords: docker, 容器, 雜湊
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 0ec6eccbcf69532d583c32136f1a0c50c9811a8b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998365"
---
# <a name="windows-container-base-image-history"></a>Windows 容器基本映像歷程記錄

每一個 Windows 容器都是採用 Microsoft 提供的基本 OS 所建置。 如果您不確定容器是以哪個 Windows 版本建置，可以執行 `docker inspect <tag>` 並依照下表比對第一列或前兩列。

例如，其結果將顯示 `docker inspect microsoft/windowsservercore:10.0.14393.447`

```
...
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67",
        "sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15"
    ]
}
```

這是由 Microsoft 於映像中提供的兩個層。 第一層固定不變並且代表原始的 Windows Server 版本，第二層則會根據包含的最新累積更新而變更。

如果您想了解每個版本做了哪些變更，請至 [Windows 10 和 Windows Server 2016 更新記錄](https://support.microsoft.com/help/12387/windows-10-update-history)中查閱各版本的知識庫。


## <a name="tools-to-simplify-this-process"></a>簡化此程序的工具

Stefan Scherer 已建立一個工具，可用來讀取映像資訊清單及判斷版本，而不需下載整個容器。 如需詳細資訊，請參閱這個[部落格](https://stefanscherer.github.io/winspector/)和 [GitHub](https://github.com/StefanScherer/winspector) 存放庫。


## <a name="image-versions"></a>映像版本

<table>
    <tr>
        <th>Windows 版本</th>
        <th>microsoft/windowsservercore</th>
        <th>microsoft/nanoserver</th>
    </tr>
    <tr>
        <td>10.0.14393.206</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063</td>
    </tr>
    <tr>
        <td>10.0.14393.321</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:cc6b0a07c696c3679af48ab4968de1b42d35e568f3d1d72df21f0acb52592e0b</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:2c195a33d84d936c7b8542a8d9890a2a550e7558e6ac73131b130e5730b9a3a5</td>
    </tr>
    <tr>
        <td>10.0.14393.447</td>
        <td>sha256:3fd27ecef6a323f5ea7f3fde1f7b87a2dbfb1afa797f88fd7d20e8dbdc856f67<br/>
        sha256:b9454c3094c68005f07ae8424021ff0e7906dac77a172a874cd5ec372528fc15</td>
        <td>sha256:342d4e407550c52261edd20cd901b5ce438f0b1e940336de3978210612365063<br/>
        sha256:c8606bedb07a714a6724b8f88ce85b71eaf5a1c80b4c226e069aa3ccbbe69154</td>
    </tr>
    <tr>
        <td>10.0.14393.576</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:de57d9086f9a337bb084b78f5f37be4c8f1796f56a1cd3ec8d8d1c9c77eb693c</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:0d812bf7a7032db75770c3d5b92c0ac9390ca4a9efa0d90ba2f55ccb16515381</td>
    </tr>
    <tr>
        <td>10.0.14393.693</td>
        <td>sha256:f358be10862ccbc329638b9e10b3d497dd7cd28b0e8c7931b4a545c88d7f7cd6<br/>
        sha256:c28d44287ce521eac86e0296c7677f5d8ca1e86d1e45e7618ec900da08c95df3</td>
        <td>sha256:6c357baed9f5177e8c8fd1fa35b39266f329535ec8801385134790eb08d8787d<br/>
        sha256:dd33c5d8d8b3c230886132c328a7801547f13de1dac9a629e2739164a285b3ab</td>
    </tr>
</table>

