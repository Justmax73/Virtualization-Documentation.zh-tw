---
title: 與 Windows 虛擬機器共用裝置
description: 引導您完成與 Hyper-V 虛擬機器共用裝置的步驟 (USB、音訊、麥克風及裝載的磁碟機)
keywords: windows 10, hyper-v
ms.author: scooley
ms.date: 10/20/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d1aeb9cb-b18f-43cb-a568-46b33346a188
ms.openlocfilehash: c891a723d43a9e6e0a0a8bc7bfc2b47a960732d1
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439585"
---
# <a name="share-devices-with-your-virtual-machine"></a>與您的虛擬機器共用裝置

> 僅適用於 Windows 虛擬機器。

加強的工作階段模式可讓 Hyper-V 使用 RDP (遠端桌面通訊協定) 連接到虛擬機器。  透過 RDP 連接不但會改善一般虛擬機器檢視體驗，也可讓虛擬機器與您的電腦共用裝置。  因為它在 Windows 10 中預設為開啟狀態，所以您可能早已使用 RDP 連接到您的 Windows 虛擬機器。  本文會強調 [連線設定] 對話方塊中的一些優點及隱藏選項。

RDP/加強的工作階段模式：

* 讓虛擬機器可調整大小並具備高度 DPI 感知能力。
* 改善虛擬機器整合
  * 共用剪貼簿
  * 透過拖放和複製貼上共用檔案
* 允許裝置共用
  * 麥克風/喇叭
  * USB 裝置
  * 資料磁碟 (包括 C:)
  * 印表機

本文會告訴您如何查看您的工作階段類型、進入加強的工作階段模式及進行您的工作階段設定。

## <a name="share-drives-and-devices"></a>共用磁碟機和裝置

加強的工作階段模式的裝置共用功能會隱藏在這個不起眼的連線視窗中，當您連接到虛擬機器時，此視窗就會跳出來：

![](media/esm-default-view.png)

根據預設，使用加強的工作階段模式的虛擬機器將會共用剪貼簿和印表機。  預設也會設定這些項目，以便將來自虛擬機器的音訊傳回電腦的喇叭。

若要與虛擬機器共用裝置或是變更這些預設設定：

1. 顯示更多選項

  ![](media/esm-show-options.png)

1. 檢視本機資源

  ![](media/esm-local-resources.png)

### <a name="share-storage-and-usb-devices"></a>共用存放裝置和 USB 裝置

根據預設，使用加強的工作階段模式的虛擬機器會共用印表機、剪貼簿、將智慧卡和其他安全性裝置傳遞給虛擬機器，好讓您可以從虛擬機器使用更安全的登入工具。

若要共用其他裝置，像是 USB 裝置或 C: 磁碟機，請選取 [其他...] 功能表：  
![](media/esm-more-devices.png)

您可從該處選取您想要與虛擬機器共用的裝置。  系統磁碟機 (Windows C:) 對於檔案共用特別有用。  
![](media/esm-drives-usb.png)

### <a name="share-audio-devices-speakers-and-microphones"></a>共用音訊裝置 (喇叭和麥克風)

根據預設，使用加強的工作階段模式的虛擬機器會傳遞音訊，好讓您可以從虛擬機器聽到音訊。  虛擬機器將會使用目前在主機電腦上選取的音訊裝置。

若要變更這些設定或新增麥克風傳遞 (好讓您可以在虛擬機器中錄製音訊)：

選取用於進行遠端音訊設定的 [設定...] 功能表  
![](media/esm-audio.png)

立即進行音訊和麥克風設定  
![](media/esm-audio-settings.png)

因為您的虛擬機器可能已在本機執行，所以 [在這部電腦上播放] 和 [在遠端電腦上播放] 選項將會產生相同的結果。

## <a name="re-launching-the-connection-settings"></a>重新啟動連線設定

如果您並未獲得解析度和裝置共用對話方塊，請嘗試從 Windows 功能表或是以系統管理員身分從命令列獨立啟動 VMConnect。  

``` Powershell
vmconnect.exe
```

## <a name="check-session-type"></a>檢查工作階段類型

您可以使用虛擬機器連線工具 (VMConnect) 最上方的「加強的工作階段模式」圖示，查看您擁有哪一種連線類型。  這個按鈕也可讓您在基本工作階段與加強的工作階段模式之間切換。

![](media/esm-button-location.png)

| 圖示 | 連線狀態 |
|:-----|:---------|
|![](media/esm-basic.png)| 您目前正在加強的工作階段模式中執行。  按一下此圖示會在基本模式中重新連接到虛擬機器。 |
|![](media/esm-connect.png)| 您目前正在基本工作階段模式中執行，但是有加強的工作階段模式可用。  按一下此圖示會在加強的工作階段模式中重新連接到虛擬機器。  |
|![](media/esm-stop.png)| 您目前正在基本模式中執行。  加強的工作階段模式不適用於此部虛擬機器。 |