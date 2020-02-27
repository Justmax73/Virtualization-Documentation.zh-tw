---
title: 建立適用于 Windows 容器的 Gmsa
description: 如何建立適用于 Windows 容器的群組受管理的服務帳戶（Gmsa）。
keywords: docker，容器，active directory，gmsa，群組受管理的服務帳戶，群組受管理的服務帳戶
author: rpsqrd
ms.date: 01/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 36061cfc491dd9dd581d1e6bce92a29e4a6f217d
ms.sourcegitcommit: 530712469552a1ef458883001ee748bab2c65ef7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/26/2020
ms.locfileid: "77628933"
---
# <a name="create-gmsas-for-windows-containers"></a>建立適用于 Windows 容器的 Gmsa

以 Windows 為基礎的網路通常會使用 Active Directory （AD）來加速使用者、電腦及其他網路資源之間的驗證和授權。 企業應用程式開發人員通常會將其應用程式設計成 AD 整合並在加入網域的伺服器上執行，以利用整合式 Windows 驗證，讓使用者和其他服務能夠輕鬆地自動且透明地登入應用程式及其身分識別。

雖然 Windows 容器無法加入網域，但仍可使用 Active Directory 網域身分識別來支援各種驗證案例。

若要達到此目的，您可以設定 Windows 容器以[群組受管理的服務帳戶](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)（gMSA）執行，這是 windows Server 2012 中引進的一種特殊的服務帳戶，其設計目的是讓多部電腦共用身分識別，而不需要知道密碼。

當您使用 gMSA 執行容器時，容器主機會從 Active Directory 網域控制站抓取 gMSA 密碼，並將其提供給容器實例。 每次其電腦帳戶（系統）需要存取網路資源時，容器都會使用 gMSA 認證。

本文說明如何開始搭配使用 Active Directory 群組受管理的服務帳戶與 Windows 容器。

## <a name="prerequisites"></a>必要條件

若要使用群組受管理的服務帳戶執行 Windows 容器，您將需要下列各項：

- 至少具有一個執行 Windows Server 2012 或更新版本之網域控制站的 Active Directory 網域。 您不需要使用 Gmsa 的樹系或網域功能等級，但 gMSA 的密碼只能由執行 Windows Server 2012 或更新版本的網域控制站散發。 如需詳細資訊，請參閱[gmsa 的 Active Directory 需求](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)。
- 建立 gMSA 帳戶的許可權。 若要建立 gMSA 帳戶，您必須是網域系統管理員，或使用已被委派 [*建立 msds-groupmanagedserviceaccount 物件*] 許可權的帳戶。
- 存取網際網路以下載 CredentialSpec PowerShell 模組。 如果您是在中斷連線的環境中工作，您可以[將模組儲存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)在具有網際網路存取的電腦上，並將它複製到您的開發電腦或容器主機。

## <a name="one-time-preparation-of-active-directory"></a>一次 Active Directory 的準備工作

如果您尚未在您的網域中建立 gMSA，您必須產生金鑰發佈服務（KDS 根金鑰）根機碼。 KDS 根金鑰負責建立、輪替和發行 gMSA 密碼給授權的主機。 當容器主機需要使用 gMSA 來執行容器時，它會連線到 KDS 根金鑰以取得目前的密碼。

若要檢查是否已建立 KDS 根金鑰根金鑰，請在已安裝 AD PowerShell 工具的網域控制站或網域成員上，以網域系統管理員身分執行下列 PowerShell Cmdlet：

```powershell
Get-KdsRootKey
```

如果命令傳回金鑰識別碼，您就已全部設定，而且可以直接跳到[建立群組受管理的服務帳戶](#create-a-group-managed-service-account)一節。 否則，請繼續建立 KDS 根金鑰根金鑰。

在具有多個網域控制站的生產環境或測試環境中，以網域系統管理員身分在 PowerShell 中執行下列 Cmdlet，以建立 KDS 根金鑰根金鑰。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

雖然此命令暗示金鑰會立即生效，但您必須等待10小時，才能複寫 KDS 根金鑰根金鑰，並可在所有網域控制站上使用。

如果您的網域中只有一個網域控制站，您可以將金鑰設定為在10小時前生效，以加速此程式。

>[!IMPORTANT]
>請勿在生產環境中使用這項技術。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>建立群組受管理的服務帳戶

使用整合式 Windows 驗證的每個容器都需要至少一個 gMSA。 每當以系統或網路服務身分執行的應用程式存取網路上的資源時，就會使用主要 gMSA。 無論指派給容器的主機名稱為何，gMSA 的名稱將會成為網路上的容器名稱。 如果您想要以不同于容器電腦帳戶的身分識別，在容器中執行服務或應用程式，也可以使用其他 Gmsa 來設定容器。

當您建立 gMSA 時，也會建立可在許多不同的電腦上同時使用的共用身分識別。 GMSA 密碼的存取權是由 Active Directory 存取控制清單所保護。 建議您為每個 gMSA 帳戶建立一個安全性群組，並將相關的容器主機新增到安全性群組，以限制密碼的存取權。

最後，由於容器不會自動註冊任何服務主體名稱（SPN），因此您必須為您的 gMSA 帳戶手動建立至少一個主機 SPN。

一般而言，主機或 HTTP SPN 會使用與 gMSA 帳戶相同的名稱來註冊，但如果用戶端從負載平衡器後方存取容器化應用程式，或與 gMSA 名稱不同的 DNS 名稱，您可能需要使用不同的服務名稱。

例如，如果 gMSA 帳戶命名為 "WebApp01"，但您的使用者在 `mysite.contoso.com`存取網站，您應該在 gMSA 帳戶上註冊 `http/mysite.contoso.com` SPN。

某些應用程式可能需要額外的 Spn，才能擁有其獨特的通訊協定。 例如，SQL Server 需要 `MSSQLSvc/hostname` SPN。

下表列出建立 gMSA 時所需的屬性。

|gMSA 屬性 | 必要值 | 範例 |
|--------------|----------------|--------|
|名稱 | 任何有效的帳戶名稱。 | `WebApp01` |
|DnsHostName | 附加至帳戶名稱的功能變數名稱。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 至少設定主機 SPN，並視需要新增其他通訊協定。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | 包含容器主機的安全性群組。 | `WebApp01Hosts` |

一旦您決定 gMSA 的名稱，請在 PowerShell 中執行下列 Cmdlet，以建立安全性群組和 gMSA。

> [!TIP]
> 您必須使用屬於**Domain Admins**安全性群組的帳戶，或是已被委派**Create msds-groupmanagedserviceaccount objects**許可權，才能執行下列命令。
> [Uninstall-adserviceaccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) Cmdlet 是[遠端伺服器管理工具](https://aka.ms/rsat)的 AD PowerShell 工具的一部分。

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
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01$", "ContainerHost02$", "ContainerHost03$"
```

我們建議您為開發、測試和生產環境建立個別的 gMSA 帳戶。

## <a name="prepare-your-container-host"></a>準備您的容器主機

將使用 gMSA 執行 Windows 容器的每個容器主機必須已加入網域，並具有可取得 gMSA 密碼的存取權。

1. 將您的電腦加入您的 Active Directory 網域。
2. 請確定您的主機屬於控制 gMSA 密碼存取權的安全性群組。
3. 重新開機電腦，讓它取得新的群組成員資格。
4. 設定[適用于 Windows 10](https://docs.docker.com/docker-for-windows/install/)或[適用於 Windows 的 Docker Server](https://docs.docker.com/install/windows/docker-ee/)的 Docker Desktop。
5. 使用執行[測試 uninstall-adserviceaccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)，確認主機可以使用 gMSA 帳戶。 如果命令傳回**False**，請遵循[疑難排解指示](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>建立認證規格

認證規格檔案是一份 JSON 檔，其中包含您想要容器使用之 gMSA 帳戶的相關中繼資料。 藉由將身分識別設定與容器映射分開，您可以變更容器所使用的 gMSA，方法是直接交換認證規格檔案，而不需要變更程式碼。

認證規格檔案是使用已加入網域之容器主機上的[CredentialSpec PowerShell 模組](https://aka.ms/credspec)所建立。
建立檔案之後，您可以將它複製到其他容器主機或容器協調器。
認證規格檔案不包含任何秘密，例如 gMSA 密碼，因為容器主機會代表容器來抓取 gMSA。

Docker 預期會在 Docker data 目錄中的**CredentialSpecs**目錄下尋找認證規格檔案。 在預設安裝中，您會在 `C:\ProgramData\Docker\CredentialSpecs`找到此資料夾。

若要在您的容器主機上建立認證規格檔案：

1. 安裝 RSAT AD PowerShell 工具
    - 若是 Windows Server，請執行**Install-**
    - 針對 Windows 10 1809 版或更新版本，請執行**WindowsCapability-Online-Name ' Rsat**. node.js. 0.0.1.0 '。
    - 如需舊版的 Windows 10，請參閱 <https://aka.ms/rsat>。
2. 執行下列 Cmdlet 來安裝最新版的[CredentialSpec PowerShell 模組](https://aka.ms/credspec)：

    ```powershell
    Install-Module CredentialSpec
    ```

    如果您的容器主機上沒有網際網路存取，請在連線到網際網路的電腦上執行 `Save-Module CredentialSpec`，並將模組資料夾複製到容器主機上 `$env:PSModulePath` 中 `C:\Program Files\WindowsPowerShell\Modules` 或另一個位置。

3. 執行下列 Cmdlet 來建立新的認證規格檔案：

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    根據預設，此 Cmdlet 會使用提供的 gMSA 名稱作為容器的電腦帳戶，來建立認證規格。 檔案將會使用檔案名的 gMSA 網域和帳戶名稱儲存在 Docker CredentialSpecs 目錄中。

    如果您想要將檔案儲存到另一個目錄，請使用 `-Path` 參數：

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -Path "C:\MyFolder\WebApp01_CredSpec.json"
    ```

    如果您在容器中執行服務或進程做為次要 gMSA，您也可以建立包含其他 gMSA 帳戶的認證規格。 若要這麼做，請使用 `-AdditionalAccounts` 參數：

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    如需支援參數的完整清單，請執行 `Get-Help New-CredentialSpec -Full`。

4. 您可以使用下列 Cmdlet 來顯示所有認證規格及其完整路徑的清單：

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>後續步驟

既然您已設定 gMSA 帳戶，就可以使用它來執行下列動作：

- [設定應用程式](gmsa-configure-app.md)
- [執行容器](gmsa-run-container.md)
- [協調容器](gmsa-orchestrate-containers.md)

如果您在安裝期間遇到任何問題，請參閱我們的[疑難排解指南](gmsa-troubleshooting.md)以取得可能的解決方案。

## <a name="additional-resources"></a>其他資源

- 若要深入瞭解 Gmsa，請參閱[群組受管理的服務帳戶總覽](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)。
- 如需影片示範，請觀賞 Ignite 2016 中[錄製的示範](https://youtu.be/cZHPz80I-3s?t=2672)。
