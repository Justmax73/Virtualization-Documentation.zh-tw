---
title: 群組受管理服務帳戶適用於 Windows 容器
description: 群組受管理服務帳戶適用於 Windows 容器
keywords: docker，容器，active directory gmsa
author: rpsqrd
ms.date: 06/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 77eadf9c1f842ab679b23813cbdd305c2f2de7e9
ms.sourcegitcommit: a5ee3e35eb272c77dd61f5e5384aab26a26fab76
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/12/2019
ms.locfileid: "9770233"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>群組受管理服務帳戶適用於 Windows 容器

Windows 網路通常會使用 Active Directory (AD) 以促進驗證及授權的使用者、 電腦及其他網路資源之間。 企業應用程式開發人員通常設計是 AD 整合，以充分利用整合式 Windows 驗證，這可讓您自動而明確的方式登入的使用者和其他服務的輕鬆的加入網域的伺服器上執行其應用程式其身分識別與應用程式。

雖然 Windows 容器無法加入網域，他們還是可以使用 Active Directory 網域識別，以支援各種不同的驗證案例。

為了達成此目的，您可以設定使用[群組受管理的服務帳戶](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)(gMSA)，也就是一種特殊類型的 Windows Server 2012 導入設計可讓多部電腦共用身分識別而不需要的服務帳戶來執行 Windows 容器了解其密碼。

當您使用群組 gMSA 執行容器時，容器主機從 Active Directory 網域控制站擷取 gMSA 密碼，並讓它成為的容器執行個體。 每當其電腦帳戶 （系統） 需要存取網路資源，容器將會使用 gMSA 認證。

這篇文章說明如何開始使用 Windows 容器使用 Active Directory 群組受管理服務帳戶。

## <a name="prerequisites"></a>必要條件

若要執行 Windows 容器與群組受管理服務帳戶，您將需要下列項目：

- Active Directory 網域與執行 Windows Server 2012 或更新版本的至少一個網域控制站。 沒有樹系或網域功能層級需求，以使用 Gmsa，但 gMSA 密碼僅能透過執行 Windows Server 2012 網域控制站分散式或更新版本。 如需詳細資訊，請參閱[Active Directory 需求 Gmsa](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)。
- 若要建立 gMSA 帳戶的權限。 若要建立 gMSA 帳戶，您將需要網域系統管理員或使用帳戶已*建立 \ [Msds-syncserverurl\ GroupManagedServiceAccount 物件*權限的委派。
- 若要下載 CredentialSpec PowerShell 模組，網際網路存取。 如果您正在中斷連線的環境中，您可以與網際網路的電腦上的[儲存模組](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)存取，並將它複製到您的開發電腦或容器主機。

## <a name="one-time-preparation-of-active-directory"></a>一次性的 Active Directory 的準備工作

如果您不在您的網域中已經建立群組 gMSA，您將需要來產生金鑰發佈服務 (KDS) 根金鑰。 KDS 負責建立、 旋轉和釋放 gMSA 密碼已獲授權的主機。 當容器主機需要使用 gMSA 來執行容器時，它將會連絡 KDS 來擷取目前的密碼。

要檢查是否 KDS 根金鑰已經建立，已安裝的 AD PowerShell 工具，在網域控制站或網域成員上以網域系統管理員身分執行下列 PowerShell cmdlet:

```powershell
Get-KdsRootKey
```

如果命令傳回一個金鑰識別碼，您準備好所有設定，以及可以往前跳過[建立群組受管理服務帳戶](#create-a-group-managed-service-account)的區段。 否則，繼續建立 KDS 根金鑰。

在生產環境或具有多個網域控制站的測試環境中，執行下列 cmdlet 在 PowerShell 中做為網域系統管理員來建立 KDS 根金鑰。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

雖然命令所示的索引鍵立即將會生效，您將需要等待 10 小時之前 KDS 根金鑰複寫和可供所有網域控制站上使用。

如果您只會在您的網域中有一個網域控制站，您可以藉由設定才會生效前 10 個小時的索引鍵來加速程序。

>[!IMPORTANT]
>不要在生產環境中使用這項技術。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>建立群組受管理服務帳戶

使用整合式 Windows 驗證每個容器都必須至少一個 gMSA。 使用主要了 gMSA，每當系統或網路服務身分執行的應用程式存取網路上的資源。 GMSA 名稱也會在網路上，無論指派給容器的主機名稱的容器的名稱。 容器也與其他 Gmsa，設定，以防您想要在容器中執行的服務或應用程式，做為不同的身分識別，從容器的電腦帳戶。

當您建立群組 gMSA 時，也會建立許多不同的電腦上可以同時使用共用身分識別。 GMSA 密碼存取受保護 Active Directory 存取控制清單。 我們建議建立安全性群組，為每個 gMSA 帳戶並新增為安全性群組的相關容器主機来限制存取權的密碼。

最後，由於容器不會自動註冊任何服務主體名稱 (SPN)，您將需要手動建立 gMSA 帳戶至少一個主機 SPN。

一般而言，主機或 http SPN 已登錄為 gMSA 帳戶，使用相同的名稱，但您可能需要使用不同的服務名稱，如果用戶端存取容器化應用程式負載平衡器或此地址不同於 gMSA 名稱的 DNS 名稱後方。

例如，如果 gMSA 帳戶名為 「 WebApp01 」，但您的使用者存取網站在`mysite.contoso.com`，您應該登錄`http/mysite.contoso.com`SPN gMSA 帳戶上的。

某些應用程式可能需要額外的 Spn 其獨特的通訊協定。 例如，需要 SQL Server `MSSQLSvc/hostname` SPN。

下表列出用於建立群組 gMSA 的必要的屬性。

|gMSA 屬性 | 所需的值 | 範例 |
|--------------|----------------|--------|
|姓名 | 任何有效的帳戶名稱。 | `WebApp01` |
|只能用 | 網域名稱附加到帳戶名稱。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 至少設定主機 SPN，在必要時新增其他通訊協定。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 包含您的容器主機的安全性群組。 | `WebApp01Hosts` |

一旦您決定您 gMSA，執行下列 cmdlet 在 PowerShell 中建立安全性群組，然後 gMSA 的名稱。

> [!TIP]
> 您將需要使用的帳戶所屬的**網域系統管理員**安全性群組或已委派執行下列命令**建立 \ [Msds-syncserverurl\ GroupManagedServiceAccount 物件**的權限。
> [新增 ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) cmdlet 是 AD PowerShell 工具，從[遠端伺服器管理工具](https://aka.ms/rsat)的一部分。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

我們建議您針對您的開發人員、 測試及生產環境建立個別的 gMSA 帳戶。

## <a name="prepare-your-container-host"></a>準備您的容器主機

每個容器主機將執行 Windows 容器與群組 gMSA 必須是網域加入，可以存取擷取 gMSA 密碼。

1. 將電腦加入到您的 Active Directory 網域。
2. 請確定您的主機屬於控制存取 gMSA 密碼的安全性群組。
3. 重新啟動電腦，因此它會取得新的群組成員資格。
4. 設定[Docker 桌面適用於 Windows 10](https://docs.docker.com/docker-for-windows/install/)或[Docker Windows Server](https://docs.docker.com/install/windows/docker-ee/)。
5. （建議選項）確認主機可以藉由執行[測試 ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)使用 gMSA 帳戶。 如果命令傳回**False**，請參閱診斷步驟的[疑難排解](#troubleshooting)一節。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>建立認證規格

認證規格的檔案是包含您想要使用容器為 gMSA 帳戶的相關中繼資料的 JSON 文件。 藉由維持身分識別設定分開的容器映像，您可以變更容器則是只需交換認證規格檔案中，程式碼不會變更所需使用哪一個 gMSA。

在加入網域的容器主機上使用[CredentialSpec PowerShell 模組](https://aka.ms/credspec)建立認證規格檔案。
一旦您已經建立檔案，您可以將它複製到其他容器主機或您的容器協調器。
認證規格檔案不包含任何機密資料，例如 gMSA 密碼，因為容器主機擷取 gMSA 代表容器。

Docker 預期 Docker 資料目錄中尋找**CredentialSpecs**目錄下的認證規格檔案。 在預設安裝中，您會發現在此資料夾`C:\ProgramData\Docker\CredentialSpecs`。

若要建立您的容器主機上的認證規格檔案：

1. 安裝 RSAT AD PowerShell 工具
    - 適用於 Windows Server 執行**安裝 RSAT Add-windowsfeature-AD PowerShell**。
    - 適用於 Windows 10，版本 1809年或更新版本，執行**安裝 WindowsCapability-線上 ' Rsat.ActiveDirectory.DS-LDS.Tools~~~0.0.1.0'**。
    - 適用於較舊的 Windows 10 版本，請參閱<https://aka.ms/rsat>。
2. 執行下列 cmdlet 來安裝最新版的[CredentialSpec PowerShell 模組](https://aka.ms/credspec)：

    ```powershell
    Install-Module CredentialSpec
    ```

    如果您不需要網際網路存取您的容器主機上，執行`Save-Module CredentialSpec`連線到網際網路的電腦上，並複製到的模組資料夾`C:\Program Files\WindowsPowerShell\Modules`中的其他位置或`$env:PSModulePath`在容器主機上。

3. 執行下列 cmdlet 來建立新的認證規格的檔案：

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    根據預設，此 cmdlet 會建立使用提供的 gMSA 名稱的電腦帳戶當做容器是認證規格。 檔案將會儲存使用 gMSA 網域和帳戶名稱，用於檔案名稱的 Docker CredentialSpecs 目錄中。

    您可以建立包含額外 gMSA 帳戶，如果您正在為次要 gMSA，在容器中執行的服務或處理序 credential 規格。 若要這樣做，請使用`-AdditionalAccounts`參數：

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    如需支援的參數完整清單，請執行`Get-Help New-CredentialSpec`。

4. 您也可以顯示所有 credential 規格和其完整路徑，使用下列 cmdlet 的清單：

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configure-your-application-to-use-the-gmsa"></a>設定您的應用程式使用 gMSA

在典型的設定中，容器只會獲得一個 gMSA 帳戶可用每當容器電腦帳戶嘗試驗證網路資源。 這表示您的應用程式將會需要以**本機系統**或**網路服務**身分執行，如果它需要使用 gMSA 身分識別。

### <a name="run-an-iis-app-pool-as-network-service"></a>以網路服務身分執行 IIS 應用程式集區

如果您裝載您的容器中的 IIS 網站，您需要利用 gmsa 身分執行所有會設定您的應用程式集區身分識別為**網路服務**。 您可以在您 Dockerfile 中執行，藉由新增下列命令：

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

如果您先前使用靜態的使用者認證，為您的 IIS 應用程式集區，請考慮 gMSA，做為用來取代那些認證。 您可以變更開發人員、 測試及生產環境之間 gMSA，而不需要變更的容器映像目前的身分識別將會自動選取 IIS。

### <a name="run-a-windows-service-as-network-service"></a>以網路服務身分執行的 Windows 服務

如果您的容器化應用程式是執行為 Windows 服務，您可以設定為**網路服務**，您的 Dockerfile 中執行的服務：

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>以網路服務身分執行任意主控台應用程式

對於不會裝載於 IIS 或服務管理員的一般主控台應用程式，通常是最簡單的方式以**網路服務**身分執行容器，因此應用程式會自動繼承 gMSA 內容。 此功能已自 Windows Server 版本 1709年開始提供。

將下列程式碼行新增至您讓它以網路服務身分執行預設的 Dockerfile:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

您也可以連線至容器做為網路服務以一次性為基礎的`docker exec`。 這是特別有用，如果您正在進行執行中的容器中的連線問題疑難排解時容器通常不會執行做為網路服務。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>使用群組 gMSA 執行容器

若要使用 gMSA 執行容器，提供認證規格檔案，以`--security-opt`參數的[docker 執行](https://docs.docker.com/engine/reference/run)：

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>在 Windows Server 2016 版本 1709年及 1803年，容器的主機名稱必須符合 gMSA 簡短名稱。

在上一個範例中，gMSA SAM 帳戶名稱是 「 webapp01 」，因此容器主機名稱也會名為 「 webapp01 」。

在 Windows Server 2019 和更新版本的主機名稱欄位不是必要的但在容器仍然來識別本身 gMSA 名稱，而不是主機名稱，即使您明確地提供一種不同。

若要檢查 gMSA 正常運作，請在容器中執行下列 cmdlet:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

如果受信任 DC 連線狀態和信任的驗證狀態不是`NERR_Success`，檢查祕訣如何偵錯問題的[疑難排解](#troubleshooting)」 一節。

您可以執行下列命令，並檢查的用戶端名稱，以確認從容器中的 gMSA 身分識別：

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

若要開啟 PowerShell 或另一個主控台應用程式為 gMSA 帳戶，您可以詢問網路服務帳戶，而不是一般 ContainerAdministrator （或針對 NanoServer Containeradministrator） 帳戶下執行容器：

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

當您執行做為網路服務時，您可以嘗試連線到 SYSVOL 網域控制站上以 gmsa 身分測試網路驗證：

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrate-containers-with-gmsa"></a>協調容器與 gMSA

在生產環境中，您通常會使用容器協調器部署和管理您的應用程式和服務。 每個 orchestrator 有它自己的管理架構，並會負責接受 credential 規格，以提供給 Windows 容器平台。

當您正在協調容器與 Gmsa 時，請確定：

> [!div class="checklist"]
> * 可使用 Gmsa 執行容器排程的所有容器主機都已加入網域
> * 容器主機都有存取權擷取所有 Gmsa 容器所使用的密碼
> * 建立並上傳到 orchestrator 或複製到每個容器主機，取決於 orchestrator 來處理它們的慣用方式 credential 規格檔案。
> * 容器網路允許與 Active Directory 網域控制站，以擷取 gMSA 票證通訊的容器

### <a name="how-to-use-gmsa-with-service-fabric"></a>如何使用 gMSA 使用 Service Fabric

Service Fabric 支援執行 Windows 容器與群組 gMSA，當您在您的應用程式資訊清單中指定的認證規格的位置。 您將需要建立認證規格檔案放置在每個主機上的 Docker 資料目錄**CredentialSpecs**子目錄中，如此 Service Fabric 可以找到它。 您可以執行**Get-CredentialSpec** cmdlet，一部分的[CredentialSpec PowerShell 模組](https://aka.ms/credspec)，以確認您的認證規格是否正確的位置。

請參閱[快速入門： Service Fabric 來部署 Windows 容器](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)並[設定適用於 Service Fabric 上執行的 Windows 容器的 gMSA](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)如需有關如何設定您的應用程式。

### <a name="how-to-use-gmsa-with-docker-swarm"></a>如何搭配 Docker Swarm 使用 gMSA

若要使用 gMSA 與受 Docker 群集的容器，執行[docker 服務建立](https://docs.docker.com/engine/reference/commandline/service_create/)命令與`--credential-spec`參數：

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

如需有關如何使用認證規格與 Docker 服務在[Docker 群集的範例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)，請參閱。

### <a name="how-to-use-gmsa-with-kubernetes"></a>如何使用 gMSA 搭配 Kubernetes

做為 Kubernetes 1.14 alpha 功能已排程在 Kubernetes 中的 Gmsa 與 Windows 容器的支援。 如需這項功能，以及如何在您的 Kubernetes 散發測試最新資訊，請參閱[設定 gMSA 的 Windows pod 和容器](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa)。

## <a name="example-uses"></a>範例使用

### <a name="sql-connection-strings"></a>SQL 連接字串

當服務以本機系統或網路服務身分在容器中執行時，服務可以使用 Windows 整合式驗證連線至 Microsoft SQL Server。

以下是使用容器身分識別驗證至 SQL Server 的連接字串的範例：

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

在 Microsoft SQL Server 上，建立使用網域及 gMSA 名稱的登入，後面加上 $。 一旦建立登入之後，您可以將它新增到資料庫中的使用者，並為它提供適當的存取權限。

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

若要查看它情形，請查看[錄製的示範](https://youtu.be/cZHPz80I-3s?t=2672)可用從工作階段中的 Microsoft Ignite 2016 「 逐步路徑至容器化-將工作負載轉換為容器。 」

## <a name="troubleshooting"></a>疑難排解

### <a name="known-issues"></a>已知問題

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>容器主機名稱必須符合適用於 Windows Server 2016 和 Windows 10 版本 1709年及 1803 gMSA 名稱

如果您正在執行 Windows Server 2016，版本 1709年或 1803 起，您的容器的主機名稱必須符合您 gMSA SAM 帳戶名稱。

當主機名稱不相符的 gMSA 名稱時，輸入 NTLM 驗證要求與名稱/SID 轉譯 （使用許多程式庫，像是 ASP.NET 成員資格角色提供者） 將會失敗。 Kerberos 會繼續正常運作，即使您的主機名稱及 gMSA 名稱不相符。

其中容器會現在一律使用其 gMSA 名稱的受指派的主機名稱無論在網路的 Windows Server 2019 中，已修正這項限制。

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>與多個容器使用 gMSA，同時在 Windows Server 2016 和 Windows 10 版本 1709年及 1803年潛在客戶暫時性失敗

因為所有容器，都才能使用相同的主機名稱，在第二個問題會影響在 Windows Server 2019 之前的 Windows 和 Windows 10 版本 1809年的版本。 當多個容器都被指派相同的身分識別和主機名稱時，當兩個容器與同時交談相同的網域控制站，可能會發生競爭情形。 當另一個容器交談相同的網域控制站時，它將會取消通訊的任何先前的容器，使用相同的身分識別。 這可能會導致間歇性的驗證失敗，有時可觀察到為信任失敗當您執行`nltest /sc_verify:contoso.com`容器內。

我們會變更允許多個容器，同時使用同一個 gMSA 來分隔容器身分識別電腦名稱，從 Windows Server 2019 中的行為。

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>您無法使用 Gmsa 與 Windows 10 版本 1703年及 1709、 1803年上的 HYPER-V 隔離容器

容器初始化將會停止回應或當您嘗試使用 HYPER-V 隔離的容器，在 Windows 10 和 Windows Server 版本 1703年及 1709、 1803年上使用 gMSA 失敗。

在 Windows Server 2019 和 Windows 10 版本 1809年中已修正此 bug。 您也可以使用 Gmsa 執行 HYPER-V 隔離容器，Windows Server 2016 和 Windows 10，版本 1607年上。

### <a name="general-troubleshooting-guidance"></a>疑難排解的一般指導方針

如果您遇到錯誤以 gmsa 身分執行容器時，下列指示可協助您找出根本原因。

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>請確定主應用程式可以使用 gMSA

1. 確認主機網域加入，而且可以觸達的網域控制站。
2. 安裝 RSAT 的 AD PowerShell 工具，並執行[測試 ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)以查看電腦是否要擷取 gMSA 的存取權。 如果 cmdlet 沒有傳回**False**，表示電腦沒有存取 gMSA 密碼。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. 如果**測試 ADServiceAccount**傳回**False**，請確認主機屬於的安全性群組，可以存取 gMSA 密碼。

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. 如果您的主機所屬的授權來擷取 gMSA 密碼的安全性群組，但仍失敗**測試 ADServiceAccount**，您可能需要重新啟動電腦，以取得新的票證反映其目前的群組成員資格。

#### <a name="check-the-credential-spec-file"></a>檢查 Credential 規格的檔案

1. 執行**Get CredentialSpec**從[CredentialSpec PowerShell 模組](https://aka.ms/credspec)來找出在電腦上的所有認證規格。 必須在 Docker 根目錄的 「 CredentialSpecs 」 目錄中儲存的認證規格。 您可以找到 Docker 根目錄所執行的**docker 資訊 f 「 {{。DockerRootDir}} 」**。
2. 開啟 CredentialSpec 檔案，並確定已正確地填寫以下列欄位：
    - **Sid**： 您 gMSA 帳戶 SID
    - **MachineAccountName**: gMSA SAM 帳戶名稱 （不包括完整網域名稱或貨幣符號）
    - **DnsTreeName**： 您的 Active Directory 樹系的 FQDN
    - **DnsName**: gMSA 所屬的網域的 FQDN
    - **NetBiosName**: gMSA 所屬的網域的 NETBIOS 名稱
    - **GroupManagedServiceAccounts/名稱**： gMSA SAM 帳戶名稱 （不使用完整網域名稱或貨幣符號包含）
    - **GroupManagedServiceAccounts/範圍**： 的網域為 FQDN，另一個用於 NETBIOS 一個項目

    您的輸入應該看起來像下列完整 credential 規格的範例：

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

3. 確認 credential 規格檔案的路徑是正確的協調流程解決方案。 如果您使用 Docker，務必包含容器執行命令`--security-opt="credentialspec=file://NAME.json"`，其中 「 NAME.json 」 已取代為名稱輸出由**Get CredentialSpec**。 名稱是一般檔案名稱，相對於在 Docker 根目錄 CredentialSpecs 資料夾。

#### <a name="check-the-firewall-configuration"></a>檢查防火牆設定

如果您在容器或主機網路上使用嚴格的防火牆原則，它可能會封鎖需要的連線到 Active Directory 網域控制站或 DNS 伺服器。

| 通訊協定及連接埠 | 用途 |
|-------------------|---------|
| TCP 與 UDP 53 | DNS |
| TCP 與 UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP 與 UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

您可能需要允許存取其他連接埠，根據您的容器會傳送到網域控制站的流量的類型。
如需使用 Active Directory 的連接埠的完整清單，請參閱[Active Directory 和 Active Directory 網域服務連接埠需求](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers)。

#### <a name="check-the-container"></a>核取容器

1. 如果您正在執行的版本在 Windows Server 2019 之前的 Windows 或 Windows 10 版本 1809，您的容器主機名稱必須符合 gMSA 名稱。 確保`--hostname`參數符合 gMSA 的簡短名稱 （沒有網域元件; 例如，「 webapp01 」，而不是 「 webapp01.contoso.com 」）。

2. 檢查容器網路功能設定，以確認容器可以立即解析並存取 gMSA 網域的網域控制站。 在容器中的設定錯誤的 DNS 伺服器是常見的問題癥結的身分識別問題。

3. 檢查是否容器具備有效的連線到網域容器中執行下列 cmdlet (使用`docker exec`或對等項目):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    信任驗證應該要傳回`NERR_SUCCESS`gMSA，且網路連線可讓容器與交談，網域。 如果失敗，請確認在主機和容器的網路設定。 兩者都必須能夠與網域控制站進行通訊。

4. 請確定您的應用程式是[設定為使用 gMSA](#configure-your-application-to-use-the-gmsa)。 當您使用群組 gMSA，不會變更容器內的使用者帳戶。 而是在系統帳戶會使用 gMSA 時它討論其他網路資源。 這表示您的應用程式將會需要為以網路服務或本機系統利用 gMSA 身分執行。

    > [!TIP]
    > 如果您執行`whoami`或使用另一種工具來找出您目前的使用者內容，在容器中，您將不會看到 gMSA 名稱本身。 這是因為您一律登入容器以本機的使用者，而不是網域識別。 每當它溝通，此環境網路資源，也就是為什麼您的 app 必須為以網路服務或本機系統執行的電腦帳戶會使用 gMSA。

5. 最後，如果您的容器似乎已正確設定，但使用者或其他服務是無法自動驗證您的容器化應用程式，檢查您的 gMSA 帳戶 Spn。 用戶端會在到達您的應用程式的名稱來尋找 gMSA 帳戶。 這可能表示，您將需要其他`host`Spn 適用於您 gMSA，如果，例如，用戶端連線到您的應用程式透過負載平衡器或不同的 DNS 名稱。

## <a name="additional-resources"></a>其他資源

- [群組受管理的服務帳戶的概觀](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
