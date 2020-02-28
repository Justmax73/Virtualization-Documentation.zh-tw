---
title: 基礎映射服務生命週期
description: Windows 容器基底映射生命週期的相關資訊。
keywords: windows 容器，容器，生命週期，版本資訊，基底映射，容器基底映射
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 27672f6a8e71989bfd47fe9810dfd429eedd9206
ms.sourcegitcommit: 379eb650f6af90f5963e2fcf9de7196238fbe9fa
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/27/2020
ms.locfileid: "77705683"
---
# <a name="base-image-servicing-lifecycles"></a>基礎映射服務生命週期

Windows 容器基底映射是以半年通道版本或 Windows Server 的長期維護通道版本為基礎。 本文將告訴您兩個通道的不同基底映射版本，支援的持續時間。

「半年通道」是每年兩次的功能更新版本，每個版本有十八個月的服務時程表。 這可讓客戶更快速地利用新的作業系統功能，無論是應用程式（特別是在容器和微服務上建立的），還是軟體定義的混合式資料中心。 如需詳細資訊，請參閱[Windows Server 半年通道總覽](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview)。

對於 Server Core 映射，客戶也可以使用長期維護通道，每兩到三年發行 Windows Server 的新主要版本。 長期維護通道版本會收到五年的主要支援，以及五年的延伸支援。 此通道適用于需要較長服務選項和功能穩定性的系統。

下表列出每一種基底映射類型、其服務通道，以及其支援的持續時間。

|Base image                       |維護管道|版本|作業系統組建|可用性|主要支援結束日期|延伸支援日期|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core、Nano Server、Windows|半年      |1909   |18363   |11/12/2019  |2021/05/11                 |N/A                  |
|Server Core、Nano Server、Windows|半年      |1903   |18362   |05/21/2019  |12/08/2020                 |N/A                  |
|Server Core                      |長期        |WIN ENT LTSB 2016 Spanish 64 Bits   |17763   |11/13/2018  |01/09/2024                 |01/09/2029           |
|Server Core、Nano Server、Windows|半年      |1809   |17763   |11/13/2018  |05/12/2020                 |N/A                  |
|Server Core，Nano Server         |半年      |1803   |17134   |04/30/2018  |11/12/2019                 |N/A                  |
|Server Core，Nano Server         |半年      |1709   |16299   |2017 年 10 月 17 日  |2019 年 4 月 9 日                 |N/A                  |
|Server Core                      |長期        |1607   |14393   |10/15/2016  |01/11/2022                 |01/11/2027           |
|Nano 伺服器                      |半年      |1607   |14393   |10/15/2016  |10/09/2018                 |N/A                  |

如需服務需求和其他其他資訊，請參閱 [Windows 生命週期常見問題](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)、 [windows Server 版本資訊](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)和[windows 基礎作業系統映射 Docker hub 存放庫](https://hub.docker.com/_/microsoft-windows-base-os-images)。
