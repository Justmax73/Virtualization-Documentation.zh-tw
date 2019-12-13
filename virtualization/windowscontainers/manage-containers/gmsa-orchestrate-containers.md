---
title: 使用 gMSA 來協調容器
description: 如何使用群組受管理的服務帳戶（gMSA）來協調 Windows 容器。
keywords: docker，容器，active directory，gmsa，協調流程，kubernetes，群組受管理的服務帳戶，群組受管理的服務帳戶
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910258"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>使用 gMSA 來協調容器

在生產環境中，您通常會使用容器協調器來部署和管理您的應用程式和服務。 每個協調器都有自己的管理範例，並負責接受認證規格以提供給 Windows 容器平臺。

當您使用群組受管理的服務帳戶（Gmsa）協調容器時，請確定：

> [!div class="checklist"]
> * 可以排程使用 Gmsa 執行容器的所有容器主機皆已加入網域
> * 容器主機具有可取得容器所使用之所有 Gmsa 密碼的存取權
> * 認證規格檔案會建立並上傳至 orchestrator，或複製到每個容器主機，視協調器偏好處理它們的方式而定。
> * 容器網路可讓容器與 Active Directory 網網域控制站進行通訊，以取得 gMSA 票證

## <a name="how-to-use-gmsa-with-service-fabric"></a>如何搭配 Service Fabric 使用 gMSA

當您在應用程式資訊清單中指定認證規格位置時，Service Fabric 支援以 gMSA 執行 Windows 容器。 您必須建立認證規格檔案，並將它放在每部主機上 Docker data 目錄的**CredentialSpecs**子目錄中，讓 Service Fabric 可以找到它。 您可以執行**CredentialSpec**指令程式，這是[CredentialSpec PowerShell 模組](https://aka.ms/credspec)的一部分，以驗證您的認證規格是否在正確的位置。

如需有關如何設定應用程式的詳細資訊，請參閱[快速入門：將 windows 容器部署至 Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) ，並[設定在 Service Fabric 上執行之 Windows 容器的 gMSA](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) 。

## <a name="how-to-use-gmsa-with-docker-swarm"></a>如何搭配使用 gMSA 與 Docker Swarm

若要使用 gMSA 搭配 Docker Swarm 所管理的容器，請使用 `--credential-spec` 參數執行[Docker 服務 create](https://docs.docker.com/engine/reference/commandline/service_create/)命令：

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

如需如何搭配使用認證規格與 Docker 服務的詳細資訊，請參閱[Docker Swarm 範例](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only)。

## <a name="how-to-use-gmsa-with-kubernetes"></a>如何搭配使用 gMSA 與 Kubernetes

在 Kubernetes 中使用 Gmsa 來排程 Windows 容器的支援可在 Kubernetes 1.14 中以 Alpha 功能的形式提供。 如需這項功能的最新資訊，以及如何在您的 Kubernetes 散發套件中進行測試，請參閱[Configure gMSA For Windows pod and](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) container。

## <a name="next-steps"></a>後續步驟

除了協調容器，您也可以使用 Gmsa 來執行下列動作：

- [設定應用程式](gmsa-configure-app.md)
- [執行容器](gmsa-run-container.md)

如果您在安裝期間遇到任何問題，請參閱我們的[疑難排解指南](gmsa-troubleshooting.md)以取得可能的解決方案。
