---
title: "適用於 Windows 容器的 Active Directory 服務帳戶"
description: "適用於 Windows 容器的 Active Directory 服務帳戶"
keywords: "docker, 容器, active directory"
author: PatrickLang
ms.date: 11/04/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
translationtype: Human Translation
ms.sourcegitcommit: 804008c172b80a4f354a92cd4d12a4e23e1d4328
ms.openlocfilehash: 00a43f8d4d27327c61e318f3a915047106ad2aca

---

# 適用於 Windows 容器的 Active Directory 服務帳戶

使用者及其他服務可能必須對您的應用程式與服務建立經過驗證的連線，以便您讓資料保持安全，並防止未獲授權的使用。 Windows Active Directory (AD) 網域本身即同時支援密碼與憑證驗證。 當您在 Windows 網域加入主機上建置應用程式或服務時，如果其以本機系統或網路服務身分執行，即預設使用主機的識別。 否則，您可以改為設定其他 AD 帳戶進行驗證。

雖然 Windows 容器無法加入網域，但也可以利用 Active Directory 網域識別，方式與裝置加入領域時雷同。 透過 Windows Server 2012 R2 網域控制站，我們引進了新的網域帳戶，名為群組受管理的服務帳戶 (gMSA)，其用意是供服務共用。 藉由使用群組受管理的服務帳戶 (gMSA)，就可以將 Windows 容器本身及其裝載的服務設定為使用特定 gMSA 作為網域識別。 任何以本機系統或網路服務身分執行的服務都會使用 Windows 容器的識別，就如同現在使用網域加入主機的識別一般。 容器映像中沒有儲存任何可能被部份公開的密碼或憑證私密金鑰，容器也不需要經過重建以變更儲存的密碼或憑證，就能重新部署至部署、測試及生產環境。 


# 詞彙和參考
- [Active Directory](http://social.technet.microsoft.com/wiki/contents/articles/1026.active-directory-services-overview.aspx) 是用於在 Windows 探索、搜尋及複寫使用者、電腦與服務帳戶資訊的服務。 
  - [Active Directory Domain Services](https://technet.microsoft.com/en-us/library/dd448614.aspx) 提供用以驗證電腦與使用者的 Windows Active Directory 網域。 
  - 當裝置成為 Active Directory 網域成員時，即_加入網域_。 網域加入是一種裝置狀態，不僅為裝置提供網域電腦識別，也凸顯了不同的網域加入服務。
  - [群組受管理的服務帳戶](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)通常縮寫為 gMSA，是一種 Active Directory 帳戶，讓使用者不需要共用密碼即可輕鬆使用 Active Directory 保護服務。 多部機器或容器可以視需要共用同一個 gMSA，以驗證服務之間的連線。
- _CredentialSpec_ PowerShell 模組 - 此模組的用途是將群組受管理的服務帳戶設定為搭配容器使用。 指令碼模組及範例步驟於 [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools) 提供，請參閱 ServiceAccount

# 運作方式

現在，群組受管理的服務帳戶通常用來保護一部電腦與一項服務之間的連線。 使用帳戶的一般步驟如下：

1. 建立群組 gMSA
2. 將服務設定為以 gMSA 身分執行
3. 對執行服務的網域加入主機授與 Active Directory 中的 gMSA 祕密存取權
4. 允許在其他服務存取 gMSA，例如資料庫或檔案共用

當服務啟動時，網域加入主機會自動從 Active Directory 取得 gMSA 祕密，然後使用該帳戶執行服務。 因為該服務以 gMSA 身分執行，所以可以存取 gMSA 獲得允許的任何資源。


Windows 容器遵循類似的程序：

1. 建立群組 gMSA。 根據預設，網域系統管理員或帳戶操作員必須執行此動作。 否則可以將建立及管理 gMSA 的權限委派給管理使用這些帳戶之服務的管理員。 請參閱 [gMSA 使用者入門](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)
2. 授與 gMSA 網域加入容器主機的存取權
3. 允許在其他服務存取 gMSA，例如資料庫或檔案共用
4. 從 [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools)使用 CredentialSpec PowerShell 模組，以儲存使用 gMSA 所需設定
5. 使用額外選項啟動容器 `--security-opt "credentialspec=..."`

當容器啟動時，以本機系統或網路服務身分執行的已安裝服務就會顯示為以 gMSA 身分執行。 這類似帳戶在網域加入主機上運作的方式，不同之處在於使用了 gMSA，而非電腦帳戶。 

![圖表 - 服務帳戶](media/serviceaccount_diagram.png)


# 範例使用


## SQL 連接字串
當服務以本機系統或網路服務身分在容器中執行時，服務可以使用 Windows 整合式驗證連線至 Microsoft SQL Server。

範例：

```none
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

在 Microsoft SQL Server 上，建立使用網域及 gMSA 名稱的登入，後面加上 $。 登入一經建立，即可新增至資料庫的使用者並獲得適當存取權限。

範例： 

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```



<!--HONumber=Nov16_HO1-->


