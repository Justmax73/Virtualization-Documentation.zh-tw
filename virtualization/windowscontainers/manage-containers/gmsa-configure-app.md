---
title: 將您的應用程式設定為使用群組受管理的服務帳戶
description: 如何設定應用程式以使用適用于 Windows 容器的群組受管理的服務帳戶（Gmsa）。
keywords: docker，容器，active directory，gmsa，應用程式，應用程式，群組受管理的服務帳戶，群組受管理的服務帳戶，設定
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909788"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>設定應用程式以使用 gMSA

在一般設定中，容器只會提供一個群組受管理的服務帳戶（gMSA），每當容器電腦帳戶嘗試向網路資源進行驗證時，就會使用此帳戶。 這表示如果您的應用程式需要使用 gMSA 身分識別，則必須以**本機系統**或**網路服務**的身分執行。

## <a name="run-an-iis-app-pool-as-network-service"></a>以網路服務的身分執行 IIS 應用程式集區

如果您要將 IIS 網站裝載在容器中，您只需要將應用程式集區身分識別設定為**Network Service**，即可充分利用 gMSA。 您可以在 Dockerfile 中新增下列命令來執行此動作：

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

如果您先前已針對 IIS 應用程式集區使用靜態使用者認證，請將 gMSA 視為這些認證的取代。 您可以變更開發、測試和生產環境之間的 gMSA，而 IIS 會自動挑選目前的身分識別，而不需要變更容器映射。

## <a name="run-a-windows-service-as-network-service"></a>以網路服務的形式執行 Windows 服務

如果您的容器化應用程式是以 Windows 服務的形式執行，您可以將服務設定為在 Dockerfile 中以**網路服務**的形式執行：

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>以網路服務的形式執行任意的主控台應用程式

針對未裝載于 IIS 或 Service Manager 的一般主控台應用程式，通常會最簡單的做法是以**Network Service**的形式執行容器，讓應用程式自動繼承 gMSA 內容。 Windows Server 1709 版提供這項功能。

將下行新增至您的 Dockerfile，讓它依預設以網路服務的形式執行：

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

您也可以使用 `docker exec`一次以網路服務的形式連接到容器。 如果您要針對執行中容器的連線問題進行疑難排解（當容器通常不是以網路服務的形式執行）時，這會特別有用。

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>後續步驟

除了設定應用程式之外，您也可以使用 Gmsa 來執行下列動作：

- [執行容器](gmsa-run-container.md)
- [協調容器](gmsa-orchestrate-containers.md)

如果您在安裝期間遇到任何問題，請參閱我們的[疑難排解指南](gmsa-troubleshooting.md)以取得可能的解決方案。
