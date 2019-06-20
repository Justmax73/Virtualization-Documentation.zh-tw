---
title: 基本影像服務週期
description: Windows 容器基本映射生命週期的相關資訊。
keywords: windows 容器、容器、生命週期、發行資訊、基礎圖像、容器基底影像
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: c26f4b225287fbc25566e36376eb8cd604d45a68
ms.sourcegitcommit: 9cd1aa792a417e71192c7aa39e409ae6ca0bc710
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2019
ms.locfileid: "9788545"
---
# <a name="base-image-servicing-lifecycles"></a>基本影像服務週期

Windows 容器基礎映射是以半年通道版本或長期服務通道版本 (Windows Server) 為基礎。 本文將說明從兩個通道的不同版本的基本映射支援的持續時間。

半年通道是每年兩次的功能更新版本, 每個版本都有為期十八個月的服務時間軸。 這可讓客戶以更快的速度來利用新的作業系統功能 (無論是在應用程式中 (尤其是在容器與微上建立的元件), 還是軟體定義的混合式資料中心。 如需詳細資訊, 請參閱[Windows Server 半年通道概覽](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)。

針對伺服器核心影像, 客戶也可以使用長期服務通道, 每兩到三年發行新的主要 Windows Server 主要版本。 長期服務通道發行的期限是五年的主流支援, 以及五年的延長支援。 此通道可與需要較長服務選項和功能性穩定性的系統搭配使用。

下表列出每個類型的基底影像、其服務通道, 以及支援的持續時間。

|基底影像                       |維護管道|版本|OS 組建|可用性|主要支援結束日期|延長支援日期|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|伺服器核心版、Nano Server、Windows|半年      |1903   |18362   |05/21/2019  |12/08/2020                 |無                  |
|Server Core                      |長期        |1809   |17763   |11/13/2018  |01/09/2024                 |01/09/2029           |
|伺服器核心版、Nano Server、Windows|半年      |1809   |17763   |11/13/2018  |05/12/2020                 |無                  |
|Server Core、Nano Server         |半年      |1803   |17134   |2018/04/30  |2019/11/12                 |無                  |
|Server Core、Nano Server         |半年      |1709   |16299   |2017 年 10 月 17 日  |2019 年 4 月 9 日                 |N/A                  |
|Server Core                      |長期        |1607   |14393   |10/15/2016  |2022 年 1 月 11 日                 |01/11/2027           |
|Nano 伺服器                      |半年      |1607   |14393   |10/15/2016  |10/09/2018                 |無                  |

如需服務需求及其他其他資訊, 請參閱 [Windows 週期常見問題](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)、 [windows Server 發行資訊](https://docs.microsoft.com/en-us/windows-server/get-started/windows-server-release-info), 以及[windows 基本 OS 影像 Docker 中樞存放庫](https://hub.docker.com/_/microsoft-windows-base-os-images)。
