---
title: "使用 Hyper-V 建立虛擬機器"
description: "在 Windows 10 Creators Update 上使用 Hyper-V 建立虛擬機器"
keywords: windows 10, hyper-v
author: aoatkinson
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 1b2b778e882b413d29f52adf3e46e12e8aceede1
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/21/2017
---
# 使用 Hyper-V 建立虛擬機器

建立虛擬機器，並安裝其作業系統。  

您需要所要執行之作業系統的 .iso 檔案。 如果您沒有該檔案，請從 [TechNet 評估中心](http://www.microsoft.com/en-us/evalcenter/)擷取 Windows 的試用版。


> Windows 10 Creators Update 導入新的**快速建立**工具，可簡化建置新虛擬機器的程序。  
  如果您不是執行 Windows 10 Creators Update 或更新版本，請遵循下列指示，改用 \[新增虛擬機器精靈\]：  
  [建立新的虛擬機器](create-virtual-machine.md)  
  [建立虛擬網路](connect-to-network.md)

那就開始吧。

![](media/quickcreatesteps_inked.jpg)

1. **開啟 \[Hyper-V 管理員\]**  
  按 Windows 鍵，然後輸入「Hyper-V 管理員」，以搜尋 Hyper-V 管理員的應用程式，或是在 \[開始\] 功能表中捲動應用程式，直到您找到 Hyper-V 管理員。

2. **開啟 \[快速建立\]**  
  在 Hyper-V 管理員右邊的 **\[動作\]** 功能表中尋找 **\[快速建立\]**。

3. **自訂您的虛擬機器**
  * (選用) 為虛擬機器命名。  
    這是 Hyper-V 用於虛擬機器的名稱，而不是指定給要部署在虛擬機器內之客體作業系統的電腦名稱。
  * 選取虛擬機器的安裝媒體。 您可以從 .iso 或 .vhdx 檔案來安裝。  
    如果您是在虛擬機器中安裝 Windows，則可啟用 \[Windows 安全開機\]。 否則，請保持不選取。
  * 設定網路。  
    如果您有現存的虛擬交換器，可以在 \[網路\] 下拉式清單中選取。 如果沒有現存的交換器，您會看到用來設定自動網路的按鈕，可自動設定外部交換器。

4. **連線到虛擬機器**  
  選取 **\[連接\]** 將會啟動 \[虛擬機器連線\]，並啟動您的虛擬機器。     
  不用擔心是否要編輯設定，您隨時都可以回來變更設定。  
  
    系統可能會提示您「按任意鍵從 CD 或 DVD 光碟開機」。 請按照提示執行這項操作。  據它所知，您是要從 CD 安裝。

恭喜您，您有了新的虛擬機器。  現在您可以準備安裝作業系統。  

您的虛擬機器看起來應該像這樣︰  
![](media/OSDeploy_upd.png) 

> **注意︰**除非您是執行大量授權版本的 Windows，否則在虛擬機器內執行的 Windows 需要個別授權。 虛擬機器的作業系統與主機作業系統無關。
