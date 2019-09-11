---
title: 使用 gMSA 的協調容器
description: 如何使用群組管理的服務帳戶（gMSA）來協調 Windows 容器。
keywords: docker、容器、active directory、gmsa、orchestration、kubernetes、群組 managed 服務帳戶、群組受管理的服務帳戶
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b4dac775dc7a4ee6375f0d803e921527e66aae5b
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079719"
---
## <a name="orchestrate-containers-with-a-gmsa"></a>使用 gMSA 的協調容器

在生產環境中，您通常會使用容器 orchestrator 來部署及管理您的應用程式和服務。 每個控制器都有自己的管理範例，並負責接受認證規格，以提供給 Windows 容器平臺。

當您使用群組管理的服務帳戶（gMSAs）來協調容器時，請確定：

> [!div class="checklist"]
> * 所有可安排在 gMSAs 上執行容器的容器主機都會加入網域
> * 容器主機擁有存取權，以取得容器所使用之所有 gMSAs 的密碼
> * 認證規格檔案會建立並上傳到 orchestrator，或複製到每個容器主機（視 orchestrator 對處理它們的方式而定）。
> * 容器網路可讓容器與 Active Directory 網網域控制站通訊，以取得 gMSA 票證

### <a name="how-to-use-gmsa-with-service-fabric"></a>如何在 Service Fabric 中使用 gMSA

當您在應用程式資訊清單中指定認證規格位置時，Service Fabric 支援以 gMSA 執行 Windows 容器。 您需要建立認證規格檔案，並放置在每個主機上 Docker 資料目錄的**CredentialSpecs**子目錄中，以便讓服務結構能找到它。 您可以執行**CredentialSpec** Cmdlet （ [CredentialSpec PowerShell 模組](https://aka.ms/credspec)的一部分），以驗證您的認證規格是否位於正確的位置。

如需如何設定您的應用程式的詳細資訊，請參閱[快速入門：將 windows 容器部署到 Service fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers)和[設定 gMSA，以取得在 service fabric 上執行的 windows 容器](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers)。

### <a name="how-to-use-gmsa-with-docker-swarm"></a>如何搭配 Docker 使用 gMSA Swarm

若要在由 Docker 所管理的容器中使用 gMSA Swarm，請使用`--credential-spec`參數執行[Docker 服務 create](https://docs.docker.com/engine/reference/commandline/service_create/)命令：

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

如需如何將認證規格搭配 Docker 服務使用的詳細資訊，請參閱[Docker Swarm 範例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)。

### <a name="how-to-use-gmsa-with-kubernetes"></a>如何搭配 Kubernetes 使用 gMSA

支援在 Kubernetes 中使用 gMSAs 進行排程的 Windows 容器，可做為 Kubernetes 1.14 中的 Alpha 功能。 如需有關此功能的最新資訊，以及如何在您的 Kubernetes 發佈中進行測試，請參閱[設定適用于 Windows 盒和容器的 gMSA](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) 。

## <a name="next-steps"></a>後續步驟

除了 [協調] 容器之外，您也可以使用 gMSAs 進行下列作業：

- [設定應用程式](gmsa-configure-app.md)
- [執行容器](gmsa-run-container.md)

如果您在安裝期間遇到任何問題，請參閱我們的[疑難排解指南](gmsa-troubleshooting.md)，以取得可能的解決方案。
