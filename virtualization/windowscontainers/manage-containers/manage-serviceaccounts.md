---
title: 建立 Windows 樹枝的 gMSAs
description: 如何建立適用于 Windows 容器的群組 Managed 服務帳戶（gMSAs）。
keywords: docker、容器、active directory、gmsa、群組受管理的服務帳戶、群組受管理的服務帳戶
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079662"
---
# <a name="create-gmsas-for-windows-containers"></a>建立 Windows 樹枝的 gMSAs

Windows 的網路通常使用 Active Directory （AD）來協助使用者、電腦與其他網路資源之間進行驗證和授權。 企業應用程式開發人員通常會設計其應用程式，以進行 AD 整合，並在加入網域的伺服器上執行，以利用集成的 Windows 驗證，讓使用者與其他服務能夠自動、透明地登入應用程式與其身分識別。

雖然 Windows 容器不能加入網域，但仍可使用 Active Directory 網域標識來支援各種驗證案例。

若要達到這個目的，您可以將 Windows 容器設定為使用[群組管理的服務帳戶](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)（gMSA）執行，這是 Windows Server 2012 中引入的一種特殊類型的服務帳戶，可讓多部電腦共用身分識別，而不需要以知道其密碼。

當您使用 gMSA 執行容器時，容器主機會從 Active Directory 網網域控制站中檢索 gMSA 密碼，並將它提供給容器實例。 只要其電腦帳戶（系統）需要存取網路資源，容器就會使用 gMSA 認證。

本文說明如何開始使用 Active Directory 群組受管理的服務帳戶與 Windows 容器。

## <a name="prerequisites"></a>必要條件

若要使用群組管理的服務帳戶執行 Windows 容器，您將需要下列各項：

- 至少有一個執行 Windows Server 2012 或更新版本之網網域控制站的 Active Directory 網域。 您不需要使用 gMSAs 的林或網域功能層級需求，但 gMSA 密碼只能由執行 Windows Server 2012 或更新版本的網網域控制站來發佈。 如需詳細資訊，請參閱[gMSAs 的 Active Directory 需求](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)。
- 建立 gMSA 帳戶的許可權。 若要建立 gMSA 帳戶，您必須是網域系統管理員或使用已委派 [*建立 GroupManagedServiceAccount 物件*] 許可權的帳戶。
- 存取網際網路以下載 CredentialSpec PowerShell 模組。 如果您是在斷開連接的環境中工作，您可以[將模組儲存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)在具備網際網路存取權的電腦上，然後將它複製到您的開發電腦或容器主機。

## <a name="one-time-preparation-of-active-directory"></a>一次性準備 Active Directory

如果您在網域中尚未建立 gMSA，您將需要產生金鑰發佈服務（KDS）根金鑰。 KDS 負責建立、旋轉 gMSA 密碼，以及將其釋放至授權的主機。 當容器主機需要使用 gMSA 來執行容器時，它會與 KDS 取得聯繫，以取得目前的密碼。

若要檢查是否已建立 KDS 根金鑰，請在安裝了 AD PowerShell 工具的網網域控制站或網域成員上，以網域管理員身分執行下列 PowerShell Cmdlet：

```powershell
Get-KdsRootKey
```

如果命令傳回金鑰識別碼，您就全都是已設定好，而且可以跳到 [[建立群組管理的服務帳戶](#create-a-group-managed-service-account)] 區段。 否則，請繼續建立 KDS 根鍵。

在含多個網網域控制站的生產環境或測試環境中，請以網域管理員身分在 PowerShell 中執行下列 Cmdlet，以建立 KDS 根金鑰。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

雖然命令暗示金鑰會立即生效，但您必須先等待10小時，才能複製 KDS 根目錄，並將它提供給所有網網域控制站。

如果您的網域只有一個網網域控制站，您可以將此金鑰設定為有效的10小時前，以加速處理常式。

>[!IMPORTANT]
>請勿在生產環境中使用這項技術。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>建立群組受管理的服務帳戶

每個使用整合 Windows 驗證的容器，都至少需要一個 gMSA。 當以系統或網路服務方式執行的 app 在網路上存取資源時，就會使用主要 gMSA。 不論指派給容器的主機名稱為何，gMSA 的名稱都會成為網路上容器的名稱。 您也可以使用額外的 gMSAs 來設定容器，以防您想要在容器中以不同于容器電腦帳戶的身分識別的方式執行服務或應用程式。

當您建立 gMSA 時，您也會建立可在許多不同電腦上同時使用的共用身分識別。 GMSA 密碼的存取權受 Active Directory 存取控制清單的保護。 我們建議您為每個 gMSA 帳戶建立一個安全性群組，並將相關的容器主機新增到安全性群組，以限制對密碼的存取權。

最後，因為樹枝不會自動註冊任何服務主體名稱（SPN），所以您必須為 gMSA 帳戶手動建立至少一個主機 SPN。

一般來說，主機或 HTTP SPN 是使用與 gMSA 帳戶相同的名稱登錄，但如果用戶端從負載平衡器，或從 gMSA 名稱以外的 DNS 名稱存取已建立的應用程式，則您可能需要使用不同的服務名稱。

例如，如果 gMSA 帳戶名為「WebApp01」，但您的使用者存取該網站的`mysite.contoso.com`位置，您應該在`http/mysite.contoso.com` gMSA 帳戶上註冊 SPN。

某些應用程式可能需要專用的 Spn，才能擁有其獨特的通訊協定。 例如，SQL Server 需要`MSSQLSvc/hostname` SPN。

下表列出建立 gMSA 時所需的屬性。

|gMSA 屬性 | 必要值 | 範例 |
|--------------|----------------|--------|
|姓名 | 任何有效的帳戶名稱。 | `WebApp01` |
|DnsHostName | 附加到帳戶名稱的功能變數名稱。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 至少設定主機 SPN，視需要新增其他通訊協定。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 包含您容器主機的安全性群組。 | `WebApp01Hosts` |

在您決定 gMSA 的名稱之後，請在 PowerShell 中執行下列 Cmdlet，以建立安全性群組和 gMSA。

> [!TIP]
> 您需要使用屬於**網域系統管理員**安全群組的帳戶，或已被委派 [**建立 GroupManagedServiceAccount 物件**] 許可權來執行下列命令。
> [新的 ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) Cmdlet 是來自[遠端伺服器管理工具](https://aka.ms/rsat)的 AD PowerShell 工具的一部分。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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

使用 gMSA 執行 Windows 容器的每個容器主機都必須已加入網域，且具有存取權以取得 gMSA 密碼。

1. 將您的電腦加入您的 Active Directory 網域。
2. 確保您的主機屬於安全群組，控制對 gMSA 密碼的存取權。
3. 重新開機電腦，讓它取得新的群組成員資格。
4. 針對[適用于 Windows Server 的](https://docs.docker.com/install/windows/docker-ee/)windows 10 或 Docker 設定[docker 桌面](https://docs.docker.com/docker-for-windows/install/)。
5. 採用透過執行[Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)，確認主機可以使用 gMSA 帳戶。 如果命令傳回**False**，請依照[疑難排解指示進行](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>建立認證規格

認證規格檔案是一個 JSON 檔，其中包含您想要容器使用之 gMSA 帳戶的中繼資料。 只要將身分識別配置與容器影像分開，您就可以變更容器所使用的 gMSA，只需交換認證規格檔案，無需變更任何程式碼。

認證規格檔案是使用網域加入的容器主機上的[CredentialSpec PowerShell 模組](https://aka.ms/credspec)建立。
一旦您建立好檔案，您就可以將它複製到其他容器主機或容器 orchestrator。
認證規格檔案不包含任何機密（例如 gMSA 密碼），因為容器主機代表容器檢索 gMSA。

Docker 預期會在 Docker 資料目錄中的**CredentialSpecs**目錄下尋找認證規格檔案。 在預設安裝中，您會在`C:\ProgramData\Docker\CredentialSpecs`中找到此資料夾。

若要在您的容器主機上建立認證規格檔案：

1. 安裝 RSAT AD PowerShell 工具
    - 若是 Windows Server，請執行**安裝-add-windowsfeature 的 RSAT-AD-PowerShell**。
    - 如果您使用的是 Windows 10 版本1809或更新版本，請執行**WindowsCapability-Online-Name "Rsat. 0.0.1.0"**。
    - 如果是舊版 Windows 10，請參閱<https://aka.ms/rsat>。
2. 執行下列 Cmdlet 以安裝最新版本的[CredentialSpec PowerShell 模組](https://aka.ms/credspec)：

    ```powershell
    Install-Module CredentialSpec
    ```

    如果您在容器主機上沒有網際網路存取權，請`Save-Module CredentialSpec`在連線至網際網路的電腦上執行，並將模組`C:\Program Files\WindowsPowerShell\Modules`資料夾複製到容器`$env:PSModulePath`主機上的另一個位置。

3. 執行下列 Cmdlet 以建立新的認證規格檔案：

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    根據預設，此 Cmdlet 會使用提供的 gMSA 名稱作為容器的電腦帳戶，以建立身分身分規格。 檔案將會使用檔案名的 gMSA 網域和帳戶名稱，儲存在 Docker CredentialSpecs 目錄中。

    如果您是在容器中執行服務或處理做為次要 gMSA，您可以建立包含其他 gMSA 帳戶的認證規範。 若要這樣做，請`-AdditionalAccounts`使用參數：

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    如需支援參數的完整清單，請`Get-Help New-CredentialSpec`執行。

4. 您可以使用下列 Cmdlet 顯示所有認證規格及其完整路徑的清單：

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>後續步驟

現在您已經設定好您的 gMSA 帳戶，您可以使用它進行下列作業：

- [設定應用程式](gmsa-configure-app.md)
- [執行容器](gmsa-run-container.md)
- [協調容器](gmsa-orchestrate-containers.md)

如果您在安裝期間遇到任何問題，請參閱我們的[疑難排解指南](gmsa-troubleshooting.md)，以取得可能的解決方案。

## <a name="additional-resources"></a>其他資源

- 若要深入瞭解 gMSAs，請參閱[群組受管理的服務帳戶概述](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)。
- 如需影片示範，請觀看我們從 Ignite 2016 的[錄製示範](https://youtu.be/cZHPz80I-3s?t=2672)。
