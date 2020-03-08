---
title: 使用 gMSA 執行容器
description: 如何使用群組受管理的服務帳戶（gMSA）來執行 Windows 容器。
keywords: docker，容器，active directory，gmsa，群組受管理的服務帳戶，群組受管理的服務帳戶
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b997cf79cdf7f1782b6299198859714563c45f8c
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853932"
---
# <a name="run-a-container-with-a-gmsa"></a>使用 gMSA 執行容器

若要使用群組受管理的服務帳戶（gMSA）執行容器，請將認證規格檔案提供給[docker run](https://docs.docker.com/engine/reference/run)的 `--security-opt` 參數：

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>在 Windows Server 2016 1709 和1803版上，容器的主機名稱必須符合 gMSA 的簡短名稱。

在上述範例中，gMSA SAM 帳戶名稱為 "webapp01"，因此容器主機名稱也會命名為 "webapp01"。

在 Windows Server 2019 和更新版本上，不需要 hostname 欄位，但是即使您明確提供不同的主機名稱，容器仍然會依照 gMSA 名稱（而不是 hostname）來識別自己。

若要檢查 gMSA 是否正常運作，請在容器中執行下列 Cmdlet：

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

如果未 `NERR_Success`信任的 DC 線上狀態和信任驗證狀態，請遵循[疑難排解指示](gmsa-troubleshooting.md#check-the-container)來進行問題的檢查。

您可以執行下列命令並檢查用戶端名稱，以確認容器內的 gMSA 身分識別：

```powershell
PS C:\> klist get webapp01

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

若要開啟 PowerShell 或另一個主控台應用程式做為 gMSA 帳戶，您可以要求容器在 Network Service 帳戶下執行，而不是一般的 ContainerAdministrator （或 ContainerUser for NanoServer）帳戶：

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

當您以網路服務的身分執行時，您可以嘗試連線到網域控制站上的 SYSVOL，以測試網路驗證作為 gMSA：

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>後續步驟

除了執行容器，您也可以使用 Gmsa 來：

- [設定應用程式](gmsa-configure-app.md)
- [協調容器](gmsa-orchestrate-containers.md)

如果您在安裝期間遇到任何問題，請參閱我們的[疑難排解指南](gmsa-troubleshooting.md)以取得可能的解決方案。
