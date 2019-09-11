---
title: Windows 樹枝 gMSAs 的疑難排解
description: 如何疑難排解 Windows 容器的群組 Managed Services 帳戶（gMSAs）。
keywords: docker、容器、active directory、gmsa、群組受管理的服務帳戶、群組受管理的服務帳戶、疑難排解、疑難排解
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 00a0d9b1367da55b7669fc26a3eca303272967ab
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079722"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Windows 樹枝 gMSAs 的疑難排解

## <a name="known-issues"></a>已知問題

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主機名稱必須符合 Windows Server 2016 和 Windows 10 （版本1709和1803）的 gMSA 名稱。

如果您執行的是 Windows Server 2016、版本1709或1803，則您的容器主機名稱必須符合您的 gMSA SAM 帳戶名稱。

當主機名稱與 gMSA 名稱不相符、入站 NTLM 驗證要求和名稱/SID 轉換（由許多文件庫（例如 ASP.NET 成員資格角色提供者）使用時，將會失敗。 即使主機名稱與 gMSA 名稱不相符，Kerberos 仍能正常運作。

這個限制是在 Windows Server 2019 中修正，而容器現在總是在網路上使用它的 gMSA 名稱（無論指派的主機名稱為何）。

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>在多個容器中同時使用 gMSA 會導致 Windows Server 2016 和 Windows 10 （版本1709和1803）的間歇性失敗。

因為所有容器都必須使用相同的主機名稱，所以第二個問題會影響 Windows Server 2019 和 Windows 10 （版本1809）之前的 Windows 版本。 將多個容器指派成相同的身分識別和主機名稱時，當兩個容器同時與同一個網網域控制站交談時，可能會發生爭用情況。 當另一個容器與同一個網網域控制站交談時，它會取消與任何先前使用相同身分識別的容器的通訊。 這可能會導致不穩定的驗證失敗，有時在容器內執行`nltest /sc_verify:contoso.com`時，可能會被視為信任失敗。

我們變更了 Windows Server 2019 中的行為，以將容器身分識別與電腦名稱稱分開，允許多個容器同時使用相同的 gMSA。

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>在 Windows 10 版本1703、1709和1803上，您無法搭配 Hyper-v 獨立容器使用 gMSAs

當您嘗試在 Windows 10 和 Windows Server 版本1703、1709和1803上使用具有 Hyper-v 隔離容器的 gMSA 時，容器初始化將會掛起或失敗。

此錯誤已在 Windows Server 2019 和 Windows 10 （版本1809）中修正。 您也可以在 Windows Server 2016 和 Windows 10 （版本1607）上執行 Hyper-v 隔離容器與 gMSAs。

## <a name="general-troubleshooting-guidance"></a>一般疑難排解指南

如果您在使用 gMSA 執行容器時遇到錯誤，下列指示可能會協助您找出根本原因。

### <a name="make-sure-the-host-can-use-the-gmsa"></a>請確定主機可以使用 gMSA

1. 確認主機已加入網域且可以存取網網域控制站。
2. 從 RSAT 安裝 AD PowerShell 工具並執行[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) ，以查看電腦是否有存取權來取得 gMSA。 如果 Cmdlet 傳回**False**，電腦就沒有 gMSA 密碼的存取權。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果**ADServiceAccount**返回**False**，請確認主機屬於可存取 gMSA 密碼的安全性群組。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果您的主機屬於已獲授權取得 gMSA 密碼的安全性群組，但仍未通過**測試-ADServiceAccount**，您可能需要重新開機電腦，以取得反映其目前群組成員資格的新票證。

#### <a name="check-the-credential-spec-file"></a>檢查認證規格檔案

1. 從[CredentialSpec PowerShell 模組](https://aka.ms/credspec)中執行**CredentialSpec** ，以找出電腦上的所有認證規格。 認證規格必須儲存在 Docker 根目錄下的「CredentialSpecs」目錄中。 您可以透過執行**docker 資訊-f "{{]，找到 docker 根目錄。DockerRootDir}} "**。
2. 開啟 CredentialSpec 檔案，並確認以下欄位已正確填入：
    - **Sid**： gMSA 帳戶的 sid
    - **MachineAccountName**： GMSA SAM 帳戶名稱（不包括完整的網功能變數名稱稱或貨幣符號）
    - **DnsTreeName**： Active Directory 林的 FQDN
    - **DnsName**： gMSA 所屬之網域的 FQDN
    - **NetBiosName**： gMSA 所屬網域的 NETBIOS 名稱
    - **GroupManagedServiceAccounts/Name**： gMSA SAM 帳戶名稱（不包括完整的網功能變數名稱稱或貨幣符號）
    - **GroupManagedServiceAccounts/作用域**：一個針對網域 FQDN 的專案，另一個用於 NETBIOS

    您的輸入應該看起來就像是完整認證規格的下列範例：

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. 確認認證規格檔案的路徑對於您的 orchestration 方案而言是正確的。 如果您使用的是 Docker，請確認容器 [執行] `--security-opt="credentialspec=file://NAME.json"`命令包括，其中 "name. json" 會以**CredentialSpec**的名稱輸出取代。 這個名稱是一個一般檔案名，相對於 Docker 根目錄下的 CredentialSpecs 資料夾。

### <a name="check-the-firewall-configuration"></a>檢查防火牆設定

如果您使用的是容器或主機網路上的嚴格防火牆原則，可能會封鎖 Active Directory 網網域控制站或 DNS 伺服器所需的連線。

| 通訊協定和埠 | 用途 |
|-------------------|---------|
| TCP 和 UDP 53 | DNS |
| TCP 和 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 和 UDP 389 | 適用 |
| TCP 636 | LDAP SSL |

根據您的容器傳送至網網域控制站的流量類型，您可能需要允許存取其他埠。
請參閱[Active directory 及 Active Directory 網域服務埠需求](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)，以取得 Active directory 使用的完整埠清單。

### <a name="check-the-container"></a>檢查容器

1. 如果您執行的是 Windows Server 2019 或 Windows 10 （版本1809）之前的 Windows 版本，您的容器主機名稱必須符合 gMSA 的名稱。 確保`--hostname`參數符合 gMSA 短名稱（沒有網域元件，例如，"webapp01"，而不是 "webapp01.contoso.com"）。

2. 檢查容器網路設定，以確認容器可以解析並存取 gMSA 網域的網網域控制站。 錯誤配置容器中的 DNS 伺服器是識別問題的常見原因。

3. 在容器中執行下列 Cmdlet （使用`docker exec`或對等），檢查容器是否有有效的連線至網域：

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    `NERR_SUCCESS`如果有可用的 gMSA，信任驗證應該會傳回，而且網路連線允許容器與網域交談。 如果失敗，請確認主機和容器的網路設定。 兩者都需要能夠與網網域控制站進行通訊。

4. 確定您的 app 已[設定為使用 gMSA](gmsa-configure-app.md)。 當您使用 gMSA 時，容器內的使用者帳戶不會變更。 相反地，系統帳戶會在與其他網路資源交談時使用 gMSA。 這表示您的 app 必須以網路服務或本機系統的身分執行，才能利用 gMSA 身分識別。

    > [!TIP]
    > 如果您在`whoami`容器中執行或使用其他工具來識別您目前的使用者內容，就不會看到 gMSA 名稱本身。 這是因為您永遠都是以本機使用者身分登入容器，而不是網域身分識別。 GMSA 會在與網路資源交談時由電腦帳戶使用，這就是為什麼您的應用程式必須以網路服務或本機系統的方式執行。

5. 最後，如果您的容器配置正確，但是使用者或其他服務無法自動驗證到您的容器化 app，請檢查您的 gMSA 帳戶上的 Spn。 用戶端會根據其到達您應用程式的名稱來找出 gMSA 帳戶。 這可能表示您需要 gMSA 的其他`host` spn （例如，用戶端透過負載平衡器或不同的 DNS 名稱連線至您的 app）。
