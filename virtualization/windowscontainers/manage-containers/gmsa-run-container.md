---
title: 使用 gMSA 執行容器
description: 如何使用群組管理的服務帳戶（gMSA）執行 Windows 容器。
keywords: docker、容器、active directory、gmsa、群組受管理的服務帳戶、群組受管理的服務帳戶
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b9c0406b5fe9527d88365dabf0cfd10114c34c74
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079715"
---
# <a name="run-a-container-with-a-gmsa"></a>使用 gMSA 執行容器

若要使用群組受管理的服務帳戶（gMSA）執行容器，請將 credential 規格檔案提供`--security-opt`給[docker](https://docs.docker.com/engine/reference/run)的參數：

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>在 Windows Server 2016 版本1709和1803上，容器的主機名稱必須符合 gMSA 的短名稱。

在上一個範例中，gMSA SAM 帳戶名稱是「webapp01」，因此容器主機名稱也稱為「webapp01」。

在 Windows Server 2019 及更新版本中，不需要主機名稱欄位，但容器仍會依據 gMSA 名稱（而不是主機名稱）來識別自己，即使您明確提供不同的名稱。

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

如果信任的 DC 線上狀態與信任驗證狀態不`NERR_Success`是，請遵循[疑難排解指示](gmsa-troubleshooting.md#check-the-container)來調試問題。

您可以執行下列命令並檢查用戶端名稱，在容器中驗證 gMSA 身分識別：

```powershell
PS C:\> klist get krbtgt

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

若要將 PowerShell 或其他主控台 app 開啟為 gMSA 帳戶，您可以要求容器在網路服務帳戶下執行，而不是在 [標準] ContainerAdministrator （或 NanoServer 的 ContainerUser）帳戶中執行：

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

當您以網路服務執行時，您可以嘗試在網網域控制站上連線到 SYSVOL，以測試網路驗證作為 gMSA：

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>後續步驟

除了執行容器之外，您也可以使用 gMSAs 進行下列作業：

- [設定應用程式](gmsa-configure-app.md)
- [協調容器](gmsa-orchestrate-containers.md)

如果您在安裝期間遇到任何問題，請參閱我們的[疑難排解指南](gmsa-troubleshooting.md)，以取得可能的解決方案。
