---
title: 將您的 app 設定為使用群組管理的服務帳戶
description: 如何將應用程式設定為使用群組 Managed 服務帳戶（gMSAs）來進行 Windows 容器。
keywords: docker、容器、active directory、gmsa、app、應用程式、群組受管理的服務帳戶、群組受管理的服務帳戶、配置
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209865"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>將您的 app 設定為使用 gMSA

在一般設定中，容器只有一個受管理的服務帳戶（gMSA）會在容器電腦帳戶嘗試驗證網路資源時使用。 這表示如果您的應用程式需要使用 gMSA 身分識別，您的 app 必須以**本機系統**或**網路服務**的方式執行。

## <a name="run-an-iis-app-pool-as-network-service"></a>以網路服務的形式執行 IIS 應用程式池

如果您是在容器中託管 IIS 網站，您只需要執行才能利用 gMSA，就可以將您的應用程式池身分識別設定為**Network Service**。 您可以在 Dockerfile 中新增下列命令來執行這項作業：

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

如果您先前曾針對您的 IIS 應用程式池使用靜態使用者認證，請考慮 gMSA 做為那些認證的替換專案。 您可以在開發人員、測試及生產環境之間變更 gMSA，而 IIS 會自動挑選目前的身分識別，而不需要變更容器影像。

## <a name="run-a-windows-service-as-network-service"></a>以網路服務的形式執行 Windows 服務

如果您的已設定您的 app 是以 Windows 服務的方式執行，您可以將服務設定為在您的 Dockerfile 中以**網路服務**執行：

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>以網路服務的方式執行任意主控台 app

對於不是在 IIS 或 Service Manager 中託管的一般主控台應用程式，通常最簡單的做法是以**網路服務**的方式執行容器，讓 app 自動繼承 gMSA 內容。 此功能可從 Windows Server 版本1709中取得。

將下列行新增到您的 Dockerfile，讓它在預設情況下以網路服務的方式執行：

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

您也可以使用`docker exec`單一登出的方式，連線到容器做為「網路服務」。 如果您要在容器不是以網路服務的方式執行時，對正在執行的容器中的連線問題進行疑難排解，這項功能特別實用。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>後續步驟

除了設定 app 之外，您也可以使用 gMSAs 進行下列作業：

- [執行容器](gmsa-run-container.md)
- [協調容器](gmsa-orchestrate-containers.md)

如果您在安裝期間遇到任何問題，請參閱我們的[疑難排解指南](gmsa-troubleshooting.md)，以取得可能的解決方案。
