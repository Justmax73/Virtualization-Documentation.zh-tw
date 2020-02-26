---
title: 建立自訂虛擬機器資源庫
description: 在 Windows 10 Creators Update 及更新版本的虛擬機器資源庫中，建置您自己的項目。
keywords: windows 10, hyper-v, 快速建立, 虛擬機器, 資源庫
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: 1348b9923d9de1314818f13414abdacee2cb9735
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439708"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>建立自訂虛擬機器資源庫

> Windows 10 Fall Creators Update 及更新版本。

在 Fall Creators Update 中，[快速建立] 擴充為包含虛擬機器資源庫。

![快速建立 VM 資源庫，含自訂映像](media/vmgallery.png)

資源庫中有一組 Microsoft 和 Microsoft 協力廠商提供的映像，同時也會列出您自己的映像。

本文詳細說明：

* 建置與資源庫相容的虛擬機器。
* 建立新的資源庫來源。
* 新增您的自訂資源庫來源至資源庫。

## <a name="gallery-architecture"></a>資源庫架構

虛擬機器資源庫是一組定義在 Windows 登錄中之虛擬機器來源的圖形檢視。  每個虛擬機器來源都是一個 JSON 檔案的路徑 (本機路徑或 URI)，其中包含虛擬機器的清單項目。

您在資源庫中看到的虛擬機器清單是第一個來源的完整內容，後面接著第二個來源的內容，以此類推，直到所有可用的虛擬機器都已列出為止。  每當您啟動資源庫時，就會動態建立此清單。

![資源庫架構](media/vmgallery-architecture.png)

登錄機碼： `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

值名稱： `GalleryLocations`

類型： `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>建立資源庫相容的虛擬機器

資源庫中的虛擬機器可以是磁碟映像 (.iso) 或虛擬硬碟 (.vhdx)。

從虛擬硬碟製作的虛擬機器電腦有幾個組態需求：

1. 建置為支援 UEFI 韌體。 如果使用 Hyper-V 建立，則是第 2 代 VM。
1. 虛擬硬碟至少應有 20GB - 請記住，這是最大大小。  Hyper-V 不會佔用 VM 未使用的空間。

### <a name="testing-a-new-vm-image"></a>測試新的 VM 映像

虛擬機器資源庫會使用從本機安裝來源進行安裝的相同機制，來建立虛擬機器。

若要驗證虛擬機器映像是否能啟動並執行：

1. 開啟 VM 資源庫 (Hyper-V 快速建立) 並選取 **\[本機安裝來源\]** 。
  ![ 按鈕，以使用本機安裝來源](media/use-local-source.png)
1. 選取 **\[變更安裝來源\]** 。
  ![ 按鈕，以使用本機安裝來源](media/change-source.png)
1. 選擇要在資源庫中使用的 .iso 或 .vhdx。
1. 如果映像是 Linux 映像，請取消選取 \[安全開機\] 選項。
  ![ 按鈕，以使用本機安裝來源](media/toggle-secure-boot.png)
1. 建立虛擬機器。  如果虛擬機器可以正確開機，就已為資源庫做好準備。

## <a name="build-a-new-gallery-source"></a>建置新的資源庫來源

下一個步驟是建立新的資源庫來源。  這是 JSON 檔案，其中列出您的虛擬機器並加入您在資源庫中看到的所有額外資訊。

文字資訊：

![有標記的資源庫文字位置](media/gallery-text.png)

* **name** - 必要 - 這是顯示在虛擬機器檢視的左欄和頂端的名稱。
* **publisher** - 必要
* **description** - 必要 - 描述 VM 的字串清單。
* **version** - 必要
* lastUpdated - 預設為 0001 年 1 月 1 日星期一。

  格式應為：yyyy-mm-ddThh:mm:ssZ

  下列 PowerShell 命令將提供適當格式的今天日期，並將它放在剪貼簿中：

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* locale - 預設為空白。

圖片：

![有標記的資源庫圖片位置](media/gallery-pictures.png)

* **logo** - 必要
* symbol
* thumbnail

以及，當然，您的虛擬機器 (.iso 或.vhdx)。

若要產生雜湊，您可以使用下列 powershell 命令：

  ``` PowerShell
  Get-FileHash -Path .\TMLogo.jpg -Algorithm SHA256
  ```

下列 JSON 範本具有入門項目和資源庫的結構描述。  如果您在 VSCode 中編輯，它會自動提供 IntelliSense。

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>將您的資源庫連接至 VM 資源庫 UI

新增自訂資源庫來源至 VM 資源庫的最簡單方法，就是在 regedit 中將其加入。

1. 開啟 **regedit.exe**
1. 流覽至 `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`
1. 尋找 `GalleryLocations` 項目。

    如果項目已存在，請移至 **\[編輯\]** 功能表和 **\[修改\]** 。

    如果項目不存在，請移至 **\[編輯\]** 功能表中，從 **\[新增\]** 瀏覽到 **\[多字串值\]**

1. 將您的資源庫新增到 `GalleryLocations` 登錄機碼。

    ![包含新項目的資源庫登錄機碼](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>疑難排解

### <a name="check-for-errors-loading-gallery"></a>檢查載入資源庫時是否發生錯誤

虛擬機器資源庫會在 Windows 事件檢視器中提供錯誤報告。  若要檢查是否有錯誤：

1. 開啟事件檢視器
1. 瀏覽至 **\[Windows 記錄\]**  ->  **\[應用程式\]**
1. 從來源 VMCreate 尋找事件。

## <a name="resources"></a>資源

GitHub [連結](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery)提供幾個資源庫指令碼和協助程式。

請[到此處](https://go.microsoft.com/fwlink/?linkid=851584)查看範例資源庫項目。  這是定義隨附資源庫的 JSON 檔案。
