---
title: 群組受管理的 Windows 容器服務帳戶
description: 群組受管理的 Windows 容器服務帳戶
keywords: docker、容器、active directory、gmsa
author: rpsqrd
ms.date: 06/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b908a35f63b2f25da3fb19c0f96b55fe3e513350
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883171"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>群組受管理的 Windows 容器服務帳戶

Windows 的網路通常使用 Active Directory (AD) 來協助使用者、電腦與其他網路資源之間進行驗證和授權。 企業應用程式開發人員通常會設計其應用程式, 以進行 AD 整合, 並在加入網域的伺服器上執行, 以利用集成的 Windows 驗證, 讓使用者與其他服務能夠自動、透明地登入應用程式與其身分識別。

雖然 Windows 容器不能加入網域, 但仍可使用 Active Directory 網域標識來支援各種驗證案例。

若要達到這個目的, 您可以將 Windows 容器設定為使用[群組管理的服務帳戶](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)(gMSA) 執行, 這是 Windows Server 2012 中引入的一種特殊類型的服務帳戶, 可讓多部電腦共用身分識別, 而不需要以知道其密碼。

當您使用 gMSA 執行容器時, 容器主機會從 Active Directory 網網域控制站中檢索 gMSA 密碼, 並將它提供給容器實例。 只要其電腦帳戶 (系統) 需要存取網路資源, 容器就會使用 gMSA 認證。

本文說明如何開始使用 Active Directory 群組受管理的服務帳戶與 Windows 容器。

## <a name="prerequisites"></a>必要條件

若要使用群組管理的服務帳戶執行 Windows 容器, 您將需要下列各項:

- 至少有一個執行 Windows Server 2012 或更新版本之網網域控制站的 Active Directory 網域。 您不需要使用 gMSAs 的林或網域功能層級需求, 但 gMSA 密碼只能由執行 Windows Server 2012 或更新版本的網網域控制站來發佈。 如需詳細資訊, 請參閱[gMSAs 的 Active Directory 需求](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)。
- 建立 gMSA 帳戶的許可權。 若要建立 gMSA 帳戶, 您必須是網域系統管理員或使用已委派 [*建立 GroupManagedServiceAccount 物件*] 許可權的帳戶。
- 存取網際網路以下載 CredentialSpec PowerShell 模組。 如果您是在斷開連接的環境中工作, 您可以[將模組儲存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)在具備網際網路存取權的電腦上, 然後將它複製到您的開發電腦或容器主機。

## <a name="one-time-preparation-of-active-directory"></a>一次性準備 Active Directory

如果您在網域中尚未建立 gMSA, 您將需要產生金鑰發佈服務 (KDS) 根金鑰。 KDS 負責建立、旋轉 gMSA 密碼, 以及將其釋放至授權的主機。 當容器主機需要使用 gMSA 來執行容器時, 它會與 KDS 取得聯繫, 以取得目前的密碼。

若要檢查是否已建立 KDS 根金鑰, 請在安裝了 AD PowerShell 工具的網網域控制站或網域成員上, 以網域管理員身分執行下列 PowerShell Cmdlet:

```powershell
Get-KdsRootKey
```

如果命令傳回金鑰識別碼, 您就全都是已設定好, 而且可以跳到 [[建立群組管理的服務帳戶](#create-a-group-managed-service-account)] 區段。 否則, 請繼續建立 KDS 根鍵。

在含多個網網域控制站的生產環境或測試環境中, 請以網域管理員身分在 PowerShell 中執行下列 Cmdlet, 以建立 KDS 根金鑰。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

雖然命令暗示金鑰會立即生效, 但您必須先等待10小時, 才能複製 KDS 根目錄, 並將它提供給所有網網域控制站。

如果您的網域只有一個網網域控制站, 您可以將此金鑰設定為有效的10小時前, 以加速處理常式。

>[!IMPORTANT]
>請勿在生產環境中使用這項技術。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>建立群組受管理的服務帳戶

每個使用整合 Windows 驗證的容器, 都至少需要一個 gMSA。 當以系統或網路服務方式執行的 app 在網路上存取資源時, 就會使用主要 gMSA。 不論指派給容器的主機名稱為何, gMSA 的名稱都會成為網路上容器的名稱。 您也可以使用額外的 gMSAs 來設定容器, 以防您想要在容器中以不同于容器電腦帳戶的身分識別的方式執行服務或應用程式。

當您建立 gMSA 時, 您也會建立可在許多不同電腦上同時使用的共用身分識別。 GMSA 密碼的存取權受 Active Directory 存取控制清單的保護。 我們建議您為每個 gMSA 帳戶建立一個安全性群組, 並將相關的容器主機新增到安全性群組, 以限制對密碼的存取權。

最後, 因為樹枝不會自動註冊任何服務主體名稱 (SPN), 所以您必須為 gMSA 帳戶手動建立至少一個主機 SPN。

一般來說, 主機或 HTTP SPN 是使用與 gMSA 帳戶相同的名稱登錄, 但如果用戶端從負載平衡器, 或從 gMSA 名稱以外的 DNS 名稱存取已建立的應用程式, 則您可能需要使用不同的服務名稱。

例如, 如果 gMSA 帳戶名為「WebApp01」, 但您的使用者存取該網站的`mysite.contoso.com`位置, 您應該在`http/mysite.contoso.com` gMSA 帳戶上註冊 SPN。

某些應用程式可能需要專用的 Spn, 才能擁有其獨特的通訊協定。 例如, SQL Server 需要`MSSQLSvc/hostname` SPN。

下表列出建立 gMSA 時所需的屬性。

|gMSA 屬性 | 必要值 | 範例 |
|--------------|----------------|--------|
|姓名 | 任何有效的帳戶名稱。 | `WebApp01` |
|DnsHostName | 附加到帳戶名稱的功能變數名稱。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 至少設定主機 SPN, 視需要新增其他通訊協定。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 包含您容器主機的安全性群組。 | `WebApp01Hosts` |

在您決定 gMSA 的名稱之後, 請在 PowerShell 中執行下列 Cmdlet, 以建立安全性群組和 gMSA。

> [!TIP]
> 您需要使用屬於**網域系統管理員**安全群組的帳戶, 或已被委派 [**建立 GroupManagedServiceAccount 物件**] 許可權來執行下列命令。
> [新的 ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) Cmdlet 是來自[遠端伺服器管理工具](https://aka.ms/rsat)的 AD PowerShell 工具的一部分。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

我們建議您為您的開發人員、測試及生產環境建立個別的 gMSA 帳戶。

## <a name="prepare-your-container-host"></a>準備您的容器主機

使用 gMSA 執行 Windows 容器的每個容器主機都必須已加入網域, 且具有存取權以取得 gMSA 密碼。

1. 將您的電腦加入您的 Active Directory 網域。
2. 確保您的主機屬於安全群組, 控制對 gMSA 密碼的存取權。
3. 重新開機電腦, 讓它取得新的群組成員資格。
4. 針對[適用于 Windows Server 的](https://docs.docker.com/install/windows/docker-ee/)windows 10 或 Docker 設定[docker 桌面](https://docs.docker.com/docker-for-windows/install/)。
5. 採用透過執行[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount), 確認主機可以使用 gMSA 帳戶。 如果命令傳回**False**, 請參閱[疑難排解](#troubleshooting)一節中的診斷步驟。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>建立認證規格

認證規格檔案是一個 JSON 檔, 其中包含您想要容器使用之 gMSA 帳戶的中繼資料。 只要將身分識別配置與容器影像分開, 您就可以變更容器所使用的 gMSA, 只需交換認證規格檔案, 無需變更任何程式碼。

認證規格檔案是使用網域加入的容器主機上的[CredentialSpec PowerShell 模組](https://aka.ms/credspec)建立。
一旦您建立好檔案, 您就可以將它複製到其他容器主機或容器 orchestrator。
認證規格檔案不包含任何機密 (例如 gMSA 密碼), 因為容器主機代表容器檢索 gMSA。

Docker 預期會在 Docker 資料目錄中的**CredentialSpecs**目錄下尋找認證規格檔案。 在預設安裝中, 您會在`C:\ProgramData\Docker\CredentialSpecs`中找到此資料夾。

若要在您的容器主機上建立認證規格檔案:

1. 安裝 RSAT AD PowerShell 工具
    - 若是 Windows Server, 請執行**安裝-add-windowsfeature 的 RSAT-AD-PowerShell**。
    - 如果您使用的是 Windows 10 版本1809或更新版本, 請執行**安裝-WindowsCapability-Online [Rsat. ActiveDirectory. 0.0.1.0**]。
    - 如果是舊版 Windows 10, 請參閱<https://aka.ms/rsat>。
2. 執行下列 Cmdlet 以安裝最新版本的[CredentialSpec PowerShell 模組](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    如果您在容器主機上沒有網際網路存取權, 請`Save-Module CredentialSpec`在連線至網際網路的電腦上執行, 並將模組`C:\Program Files\WindowsPowerShell\Modules`資料夾複製到容器`$env:PSModulePath`主機上的另一個位置。

3. 執行下列 Cmdlet 以建立新的認證規格檔案:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    根據預設, 此 Cmdlet 會使用提供的 gMSA 名稱作為容器的電腦帳戶, 以建立身分身分規格。 檔案將會使用檔案名的 gMSA 網域和帳戶名稱, 儲存在 Docker CredentialSpecs 目錄中。

    如果您是在容器中執行服務或處理做為次要 gMSA, 您可以建立包含其他 gMSA 帳戶的認證規範。 若要這樣做, 請`-AdditionalAccounts`使用參數:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    如需支援參數的完整清單, 請`Get-Help New-CredentialSpec`執行。

4. 您可以使用下列 Cmdlet 顯示所有認證規格及其完整路徑的清單:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configure-your-application-to-use-the-gmsa"></a>將您的應用程式設定為使用 gMSA

在一般設定中, 只有當容器電腦帳戶嘗試向網路資源進行驗證時, 才會使用容器中的一個 gMSA 帳戶。 這表示如果您的應用程式需要使用 gMSA 身分識別, 您的 app 必須以**本機系統**或**網路服務**的方式執行。

### <a name="run-an-iis-app-pool-as-network-service"></a>以網路服務的形式執行 IIS 應用程式池

如果您是在容器中託管 IIS 網站, 您只需要執行才能利用 gMSA, 就可以將您的應用程式池身分識別設定為**Network Service**。 您可以在 Dockerfile 中新增下列命令來執行這項作業:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

如果您先前曾針對您的 IIS 應用程式池使用靜態使用者認證, 請考慮 gMSA 做為那些認證的替換專案。 您可以在開發人員、測試及生產環境之間變更 gMSA, 而 IIS 會自動挑選目前的身分識別, 而不需要變更容器影像。

### <a name="run-a-windows-service-as-network-service"></a>以網路服務的形式執行 Windows 服務

如果您的已設定您的 app 是以 Windows 服務的方式執行, 您可以將服務設定為在您的 Dockerfile 中以**網路服務**執行:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>以網路服務的方式執行任意主控台 app

對於不是在 IIS 或 Service Manager 中託管的一般主控台應用程式, 通常最簡單的做法是以**網路服務**的方式執行容器, 讓 app 自動繼承 gMSA 內容。 此功能可從 Windows Server 版本1709中取得。

將下列行新增到您的 Dockerfile, 讓它在預設情況下以網路服務的方式執行:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

您也可以使用`docker exec`單一登出的方式, 連線到容器做為「網路服務」。 如果您要在容器不是以網路服務的方式執行時, 對正在執行的容器中的連線問題進行疑難排解, 這項功能特別實用。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>使用 gMSA 執行容器

若要使用 gMSA 執行容器, 請將認證規格檔案提供給`--security-opt` [docker](https://docs.docker.com/engine/reference/run)的參數:

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>在 Windows Server 2016 版本1709和1803上, 容器的主機名稱必須符合 gMSA 的短名稱。

在上一個範例中, gMSA SAM 帳戶名稱是「webapp01」, 因此容器主機名稱也稱為「webapp01」。

在 Windows Server 2019 及更新版本中, 不需要主機名稱欄位, 但容器仍會依據 gMSA 名稱 (而不是主機名稱) 來識別自己, 即使您明確提供不同的名稱。

若要檢查 gMSA 是否正常運作, 請在容器中執行下列 Cmdlet:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

如果信任的 DC 線上狀態與信任驗證狀態不`NERR_Success`是, 請查看[疑難排解](#troubleshooting)一節, 以瞭解如何調試問題的秘訣。

您可以執行下列命令並檢查用戶端名稱, 在容器中驗證 gMSA 身分識別:

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

若要將 PowerShell 或其他主控台 app 開啟為 gMSA 帳戶, 您可以要求容器在網路服務帳戶下執行, 而不是在 [標準] ContainerAdministrator (或 NanoServer 的 ContainerUser) 帳戶中執行:

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

當您以網路服務執行時, 您可以嘗試在網網域控制站上連線到 SYSVOL, 以測試網路驗證作為 gMSA:

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrate-containers-with-gmsa"></a>使用 gMSA 的協調容器

在生產環境中, 您通常會使用容器 orchestrator 來部署及管理您的應用程式和服務。 每個控制器都有自己的管理範例, 並負責接受認證規格, 以提供給 Windows 容器平臺。

當您使用 gMSAs 協調容器時, 請確定:

> [!div class="checklist"]
> * 所有可安排在 gMSAs 上執行容器的容器主機都會加入網域
> * 容器主機擁有存取權, 以取得容器所使用之所有 gMSAs 的密碼
> * 認證規格檔案會建立並上傳到 orchestrator, 或複製到每個容器主機 (視 orchestrator 對處理它們的方式而定)。
> * 容器網路可讓容器與 Active Directory 網網域控制站通訊, 以取得 gMSA 票證

### <a name="how-to-use-gmsa-with-service-fabric"></a>如何在 Service Fabric 中使用 gMSA

當您在應用程式資訊清單中指定認證規格位置時, Service Fabric 支援以 gMSA 執行 Windows 容器。 您需要建立認證規格檔案, 並放置在每個主機上 Docker 資料目錄的**CredentialSpecs**子目錄中, 以便讓服務結構能找到它。 您可以執行**CredentialSpec** Cmdlet ( [CredentialSpec PowerShell 模組](https://aka.ms/credspec)的一部分), 以驗證您的認證規格是否位於正確的位置。

如需如何設定您的應用程式的詳細資訊, 請參閱[快速入門: 將 windows 容器部署到 Service fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)和[設定 gMSA, 以取得在 service fabric 上執行的 windows 容器](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)。

### <a name="how-to-use-gmsa-with-docker-swarm"></a>如何搭配 Docker 使用 gMSA Swarm

若要在由 Docker 所管理的容器中使用 gMSA Swarm, 請使用`--credential-spec`參數執行[Docker 服務 create](https://docs.docker.com/engine/reference/commandline/service_create/)命令:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

如需如何將認證規格搭配 Docker 服務使用的詳細資訊, 請參閱[Docker Swarm 範例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)。

### <a name="how-to-use-gmsa-with-kubernetes"></a>如何搭配 Kubernetes 使用 gMSA

支援在 Kubernetes 中使用 gMSAs 進行排程的 Windows 容器, 可做為 Kubernetes 1.14 中的 Alpha 功能。 如需有關此功能的最新資訊, 以及如何在您的 Kubernetes 發佈中進行測試, 請參閱[設定適用于 Windows 盒和容器的 gMSA](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) 。

## <a name="example-uses"></a>範例使用

### <a name="sql-connection-strings"></a>SQL 連接字串

當服務以本機系統或網路服務身分在容器中執行時，服務可以使用 Windows 整合式驗證連線至 Microsoft SQL Server。

以下是使用容器身分驗證 SQL Server 的連接字串範例:

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

在 Microsoft SQL Server 上，建立使用網域及 gMSA 名稱的登入，後面加上 $。 一旦您建立好登入, 您就可以將它新增到資料庫上的使用者, 然後給予其適當的存取許可權。

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

若要查看它的運作方式, 請參閱在會話中使用 Microsoft Ignite 2016 的[錄製示範](https://youtu.be/cZHPz80I-3s?t=2672), 此逐步說明如何 Containerization 將工作負載轉換成容器。

## <a name="troubleshooting"></a>疑難排解

### <a name="known-issues"></a>已知問題

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主機名稱必須符合 Windows Server 2016 和 Windows 10 (版本1709和 1803) 的 gMSA 名稱。

如果您執行的是 Windows Server 2016、版本1709或 1803, 則您的容器主機名稱必須符合您的 gMSA SAM 帳戶名稱。

當主機名稱與 gMSA 名稱不相符、入站 NTLM 驗證要求和名稱/SID 轉換 (由許多文件庫 (例如 ASP.NET 成員資格角色提供者) 使用時, 將會失敗。 即使主機名稱與 gMSA 名稱不相符, Kerberos 仍能正常運作。

這個限制是在 Windows Server 2019 中修正, 而容器現在總是在網路上使用它的 gMSA 名稱 (無論指派的主機名稱為何)。

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>在多個容器中同時使用 gMSA 會導致 Windows Server 2016 和 Windows 10 (版本1709和 1803) 的間歇性失敗。

因為所有容器都必須使用相同的主機名稱, 所以第二個問題會影響 Windows Server 2019 和 Windows 10 (版本 1809) 之前的 Windows 版本。 將多個容器指派成相同的身分識別和主機名稱時, 當兩個容器同時與同一個網網域控制站交談時, 可能會發生爭用情況。 當另一個容器與同一個網網域控制站交談時, 它會取消與任何先前使用相同身分識別的容器的通訊。 這可能會導致不穩定的驗證失敗, 有時在容器內執行`nltest /sc_verify:contoso.com`時, 可能會被視為信任失敗。

我們變更了 Windows Server 2019 中的行為, 以將容器身分識別與電腦名稱稱分開, 允許多個容器同時使用相同的 gMSA。

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>在 Windows 10 版本1703、1709和1803上, 您無法搭配 Hyper-v 獨立容器使用 gMSAs

當您嘗試在 Windows 10 和 Windows Server 版本1703、1709和1803上使用具有 Hyper-v 隔離容器的 gMSA 時, 容器初始化將會掛起或失敗。

此錯誤已在 Windows Server 2019 和 Windows 10 (版本 1809) 中修正。 您也可以在 Windows Server 2016 和 Windows 10 (版本 1607) 上執行 Hyper-v 隔離容器與 gMSAs。

### <a name="general-troubleshooting-guidance"></a>一般疑難排解指南

如果您在使用 gMSA 執行容器時遇到錯誤, 下列指示可能會協助您找出根本原因。

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>請確定主機可以使用 gMSA

1. 確認主機已加入網域且可以存取網網域控制站。
2. 從 RSAT 安裝 AD PowerShell 工具並執行[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) , 以查看電腦是否有存取權來取得 gMSA。 如果 Cmdlet 傳回**False**, 電腦就沒有 gMSA 密碼的存取權。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果**ADServiceAccount**返回**False**, 請確認主機屬於可存取 gMSA 密碼的安全性群組。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果您的主機屬於已獲授權取得 gMSA 密碼的安全性群組, 但仍未通過**測試-ADServiceAccount**, 您可能需要重新開機電腦, 以取得反映其目前群組成員資格的新票證。

#### <a name="check-the-credential-spec-file"></a>檢查認證規格檔案

1. 從[CredentialSpec PowerShell 模組](https://aka.ms/credspec)中執行**CredentialSpec** , 以找出電腦上的所有認證規格。 認證規格必須儲存在 Docker 根目錄下的「CredentialSpecs」目錄中。 您可以透過執行**docker 資訊-f "{{], 找到 docker 根目錄。DockerRootDir}} "**。
2. 開啟 CredentialSpec 檔案, 並確認以下欄位已正確填入:
    - **Sid**: gMSA 帳戶的 sid
    - **MachineAccountName**: GMSA SAM 帳戶名稱 (不包括完整的網功能變數名稱稱或貨幣符號)
    - **DnsTreeName**: Active Directory 林的 FQDN
    - **DnsName**: gMSA 所屬之網域的 FQDN
    - **NetBiosName**: gMSA 所屬網域的 NETBIOS 名稱
    - **GroupManagedServiceAccounts/Name**: gMSA SAM 帳戶名稱 (不包括完整的網功能變數名稱稱或貨幣符號)
    - **GroupManagedServiceAccounts/作用域**: 一個針對網域 FQDN 的專案, 另一個用於 NETBIOS

    您的輸入應該看起來就像是完整認證規格的下列範例:

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

3. 確認認證規格檔案的路徑對於您的 orchestration 方案而言是正確的。 如果您使用的是 Docker, 請確認容器 [執行] `--security-opt="credentialspec=file://NAME.json"`命令包括, 其中 "name. json" 會以**CredentialSpec**的名稱輸出取代。 這個名稱是一個一般檔案名, 相對於 Docker 根目錄下的 CredentialSpecs 資料夾。

#### <a name="check-the-firewall-configuration"></a>檢查防火牆設定

如果您使用的是容器或主機網路上的嚴格防火牆原則, 可能會封鎖 Active Directory 網網域控制站或 DNS 伺服器所需的連線。

| 通訊協定和埠 | 用途 |
|-------------------|---------|
| TCP 和 UDP 53 | DNS |
| TCP 和 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 和 UDP 389 | 適用 |
| TCP 636 | LDAP SSL |

根據您的容器傳送至網網域控制站的流量類型, 您可能需要允許存取其他埠。
請參閱[Active directory 及 Active Directory 網域服務埠需求](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers), 以取得 Active directory 使用的完整埠清單。

#### <a name="check-the-container"></a>檢查容器

1. 如果您執行的是 Windows Server 2019 或 Windows 10 (版本 1809) 之前的 Windows 版本, 您的容器主機名稱必須符合 gMSA 的名稱。 確保`--hostname`參數符合 gMSA 短名稱 (沒有網域元件, 例如, "webapp01", 而不是 "webapp01.contoso.com")。

2. 檢查容器網路設定, 以確認容器可以解析並存取 gMSA 網域的網網域控制站。 錯誤配置容器中的 DNS 伺服器是識別問題的常見原因。

3. 在容器中執行下列 Cmdlet (使用`docker exec`或對等), 檢查容器是否有有效的連線至網域:

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    `NERR_SUCCESS`如果有可用的 gMSA, 信任驗證應該會傳回, 而且網路連線允許容器與網域交談。 如果失敗, 請確認主機和容器的網路設定。 兩者都需要能夠與網網域控制站進行通訊。

4. 確定您的 app 已[設定為使用 gMSA](#configure-your-application-to-use-the-gmsa)。 當您使用 gMSA 時, 容器內的使用者帳戶不會變更。 相反地, 系統帳戶會在與其他網路資源交談時使用 gMSA。 這表示您的 app 必須以網路服務或本機系統的身分執行, 才能利用 gMSA 身分識別。

    > [!TIP]
    > 如果您在`whoami`容器中執行或使用其他工具來識別您目前的使用者內容, 就不會看到 gMSA 名稱本身。 這是因為您永遠都是以本機使用者身分登入容器, 而不是網域身分識別。 GMSA 會在與網路資源交談時由電腦帳戶使用, 這就是為什麼您的應用程式必須以網路服務或本機系統的方式執行。

5. 最後, 如果您的容器配置正確, 但是使用者或其他服務無法自動驗證到您的容器化 app, 請檢查您的 gMSA 帳戶上的 Spn。 用戶端會根據其到達您應用程式的名稱來找出 gMSA 帳戶。 這可能表示您需要 gMSA 的其他`host` spn (例如, 用戶端透過負載平衡器或不同的 DNS 名稱連線至您的 app)。

## <a name="additional-resources"></a>其他資源

- [群組受管理的服務帳戶概述](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
