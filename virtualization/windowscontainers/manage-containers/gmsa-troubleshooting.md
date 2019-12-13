---
title: 針對 Windows 容器的 Gmsa 進行疑難排解
description: 如何針對 Windows 容器的群組受管理的服務帳戶（Gmsa）進行疑難排解。
keywords: docker，容器，active directory，gmsa，群組受管理的服務帳戶，群組受管理的服務帳戶，疑難排解，疑難排解
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910238"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>針對 Windows 容器的 Gmsa 進行疑難排解

## <a name="known-issues"></a>已知問題

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主機名稱必須符合 Windows Server 2016 和 Windows 10 版本1709和1803的 gMSA 名稱

如果您執行的是 Windows Server 2016 1709 或1803版，容器的主機名稱必須符合您的 gMSA SAM 帳戶名稱。

當主機名稱不符合 gMSA 名稱時，輸入 NTLM 驗證要求和名稱/SID 轉譯（由許多程式庫使用，例如 ASP.NET 成員資格角色提供者）將會失敗。 即使主機名稱和 gMSA 名稱不相符，Kerberos 仍會繼續正常運作。

這項限制已在 Windows Server 2019 中修正，其中容器現在會在網路上一律使用其 gMSA 名稱，而不論指派的主機名稱為何。

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>使用具有多個容器的 gMSA 同時導致 Windows Server 2016 和 Windows 10 版本1709與1803發生間歇性失敗

由於所有容器都需要使用相同的主機名稱，因此第二個問題會影響 windows Server 2019 和 Windows 10 版本1809之前的 Windows 版本。 當有多個容器指派相同的身分識別和主機名稱時，當兩個容器同時與相同的網域控制站交談時，可能會發生競爭情況。 當另一個容器與相同的網域控制站交談時，它會取消與任何使用相同身分識別的先前容器的通訊。 這可能會導致間歇驗證失敗，有時可能會在您于容器內執行 `nltest /sc_verify:contoso.com` 時觀察為信任失敗。

我們已變更 Windows Server 2019 中的行為，以分隔容器身分識別與電腦名稱稱，讓多個容器同時使用相同的 gMSA。

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>您無法在 Windows 10 版本1703、1709和1803上搭配使用 Gmsa 與 Hyper-v 隔離的容器

當您嘗試在 Windows 10 和 Windows Server 1703、1709和1803版上使用具有 Hyper-v 隔離容器的 gMSA 時，容器初始化將會停止回應或失敗。

這個 bug 已在 Windows Server 2019 和 Windows 10 版本1809中修正。 您也可以在 Windows Server 2016 和 Windows 10 1607 版的 Gmsa 上執行 Hyper-v 隔離的容器。

## <a name="general-troubleshooting-guidance"></a>一般疑難排解指引

如果您在執行具有 gMSA 的容器時遇到錯誤，下列指示可協助您找出根本原因。

### <a name="make-sure-the-host-can-use-the-gmsa"></a>請確定主機可以使用 gMSA

1. 確認主機已加入網域，且可以連線到網域控制站。
2. 從 RSAT 安裝 AD PowerShell 工具並執行[測試 uninstall-adserviceaccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) ，以查看電腦是否有權取得 gMSA。 如果 Cmdlet 傳回**False**，則表示電腦沒有 gMSA 密碼的存取權。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果**uninstall-adserviceaccount**傳回**False**，請確認主機屬於可存取 gMSA 密碼的安全性群組。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果您的主機屬於授權用來抓取 gMSA 密碼，但仍失敗**測試 uninstall-adserviceaccount**的安全性群組，您可能需要重新開機電腦，以取得反映其目前群組成員資格的新票證。

#### <a name="check-the-credential-spec-file"></a>檢查認證規格檔案

1. 從[CredentialSpec PowerShell 模組](https://aka.ms/credspec)執行**CredentialSpec** ，以找出電腦上的所有認證規格。 認證規格必須儲存在 Docker 根目錄底下的 "CredentialSpecs" 目錄中。 您可以藉由執行**docker info-f "{{來尋找 docker 根目錄。DockerRootDir}} "** 。
2. 開啟 CredentialSpec 檔案，並確認已正確填寫下欄欄位：
    - **Sid**：您 gMSA 帳戶的 sid
    - **MachineAccountName**： GMSA SAM 帳戶名稱（不包含完整功能變數名稱或貨幣符號）
    - **DnsTreeName**： Active Directory 樹系的 FQDN
    - **DnsName**： gMSA 所屬網域的 FQDN
    - **NetBiosName**： gMSA 所屬網域的 NETBIOS 名稱
    - **GroupManagedServiceAccounts/Name**： gMSA SAM 帳戶名稱（不包含完整功能變數名稱或貨幣符號）
    - **GroupManagedServiceAccounts/Scope**：網域 FQDN 的一個專案，另一個用於 NETBIOS

    您的輸入看起來應該像下列完整認證規格的範例：

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

3. 請確認您的協調流程解決方案的認證規格檔案路徑是否正確。 如果您使用的是 Docker，請確定容器執行命令包含 `--security-opt="credentialspec=file://NAME.json"`，其中 "NAME. json" 會取代為**CredentialSpec**的名稱輸出。 名稱是一般檔案名，相對於 Docker 根目錄下的 CredentialSpecs 資料夾。

### <a name="check-the-firewall-configuration"></a>檢查防火牆設定

如果您在容器或主機網路上使用嚴格的防火牆原則，它可能會封鎖 Active Directory 網網域控制站或 DNS 伺服器所需的連線。

| 通訊協定及連接埠 | 用途 |
|-------------------|---------|
| TCP 和 UDP 53 | DNS |
| TCP 和 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 和 UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

根據您的容器傳送至網域控制站的流量類型而定，您可能需要允許存取其他埠。
如需 Active Directory 所使用之埠的完整清單，請參閱[Active Directory 和 Active Directory Domain Services 埠需求](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)。

### <a name="check-the-container"></a>檢查容器

1. 如果您在 Windows Server 2019 或 Windows 10 版本1809之前執行 Windows 版本，您的容器主機名稱必須符合 gMSA 名稱。 請確定 `--hostname` 參數符合 gMSA 的簡短名稱（沒有網域元件，例如 "webapp01"，而不是 "webapp01.contoso.com"）。

2. 檢查容器網路設定，以確認容器可以解析和存取 gMSA 網域的網域控制站。 在容器中設定錯誤的 DNS 伺服器，是身分識別問題的常見原因。

3. 在容器中執行下列 Cmdlet （使用 `docker exec` 或對等），以檢查容器是否具有有效的網域連線：

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    如果 gMSA 可供使用，且網路連線允許容器與網域通訊，則信任驗證應該會傳回 `NERR_SUCCESS`。 如果失敗，請確認主機和容器的網路設定。 兩者都必須能夠與網域控制站通訊。

4. 檢查容器是否可以取得有效的 Kerberos 票證授權票證（TGT）：

    ```powershell
    klist get krbtgt
    ```

    此命令應該會傳回「已成功取得 krbtgt 的票證」，並列出用來抓取票證的網域控制站。 如果您可以取得 TGT，但從上一個步驟 `nltest` 失敗，這可能表示 gMSA 帳戶設定不正確。 如需詳細資訊，請參閱[檢查 gMSA 帳戶](#check-the-gmsa-account)。

    如果您無法取得容器內的 TGT，這可能表示 DNS 或網路連線問題。 請確定容器可以使用網域 DNS 名稱解析網域控制站，且網域控制站可從容器路由傳送。

5. 請確定您的應用程式已[設定為使用 gMSA](gmsa-configure-app.md)。 當您使用 gMSA 時，容器內的使用者帳戶不會變更。 相反地，當系統帳戶與其他網路資源交談時，會使用 gMSA。 這表示您的應用程式將需要以網路服務或本機系統的身分執行，才能利用 gMSA 身分識別。

    > [!TIP]
    > 如果您執行 `whoami` 或使用其他工具來識別容器中目前的使用者內容，您就不會看到 gMSA 名稱本身。 這是因為您一律以本機使用者身分登入容器，而不是網域身分識別。 每當電腦帳戶與網路資源交談時，就會使用 gMSA，這就是為什麼您的應用程式需要以網路服務或本機系統的身分執行。

### <a name="check-the-gmsa-account"></a>檢查 gMSA 帳戶

1. 如果您的容器似乎設定正確，但使用者或其他服務無法自動向您的容器化應用程式進行驗證，請檢查 gMSA 帳戶上的 Spn。 用戶端會依其到達應用程式的名稱來尋找 gMSA 帳戶。 例如，如果用戶端透過負載平衡器或不同的 DNS 名稱連線到您的應用程式，則這可能表示您需要 gMSA 的額外 `host` Spn。

2. 請確定 gMSA 和容器主機屬於相同的 Active Directory 網域。 如果 gMSA 屬於不同的網域，容器主機將無法取得 gMSA 密碼。

3. 請確定您的網域中只有一個帳戶與您的 gMSA 同名。 gMSA 物件會將貨幣符號（$）附加至其 SAM 帳戶名稱，因此，gMSA 可能會命名為 "我的帳戶 $"，而不相關的使用者帳戶在相同網域中命名為 "我的帳戶"。 如果網域控制站或應用程式必須依名稱查閱 gMSA，這可能會造成問題。 您可以使用下列命令，在 AD 中搜尋類似名稱的物件：

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. 如果您已在 gMSA 帳戶上啟用非限制委派，請確定[UserAccountControl 屬性](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties)仍已啟用 `WORKSTATION_TRUST_ACCOUNT` 旗標。 容器中的 NETLOGON 必須要有此旗標，才能與網域控制站通訊，這是因為應用程式必須將名稱解析成 SID，反之亦然。 您可以使用下列命令來檢查是否已正確設定旗標：

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    如果上述命令傳回 `False`，請使用下列程式將 `WORKSTATION_TRUST_ACCOUNT` 旗標新增至 gMSA 帳戶的 UserAccountControl 屬性。 此命令也會清除 UserAccountControl 屬性中的 `NORMAL_ACCOUNT`、`INTERDOMAIN_TRUST_ACCOUNT`和 `SERVER_TRUST_ACCOUNT` 旗標。

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
