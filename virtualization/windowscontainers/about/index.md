---
title: About Windows Containers
description: Learn about Windows containers.
keywords: docker, containers
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 2be7a06c7b7b154e392c30981cdf954d2d1b796e
ms.sourcegitcommit: 8e193d8c274a549aef497f16dcdb00d7855e9fa7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2017
---
# Windows 容器

## 什麼是容器

容器是一種將應用程式封裝在隔離箱內的方式。 對於容器中的應用程式而言，它不了解存在箱外的任何其他應用程式或處理序。 此應用程式順利執行所仰賴的一切也都存放在這個容器內部。  不論這個隔離箱移動到哪裡，都永遠能滿足該應用程式的執行條件，因為它執行所需的一切都封裝在一起。

假設有一間廚房。 我們裝入所有設備和家具、鍋碗瓢盆、清潔劑和抹布。 這就是我們的容器。

<center style="margin: 25px">![](media/box1.png)</center>

現在，我們可以將這個容器放入想要的任何主機公寓中，而且它都會是相同的廚房。 我們只需要接通水電，就可以開始烹飪了 (因為我們擁有所需的所有設備)！

<center style="margin: 25px">![](media/apartment.png)</center>

容器與這個廚房十分相似。 可能會有不同種類的房間，也可能會有許多相同種類的房間。 重點在於，容器與它們所需的一切封裝在一起。

觀看簡短的概觀：[Windows 容器：具備企業級控制能力的現代化應用程式開發](https://youtu.be/Ryx3o0rD5lY) (英文)。

## 容器的基礎

容器是一種隔離、由資源控制的可攜式執行階段環境，而且這種環境會在主機電腦或虛擬機器上執行。 在容器中執行的應用程式或處理序會與所有必要的相依性和組態檔封裝在一起，使其產生出容器外部沒有其他執行中的處理序這種錯覺。

容器的主機會為容器佈建一組資源，而且容器只會使用這些資源。 就容器所知，除了提供給它的資源以外，沒有其他資源存在，因此也無法接觸可能已經佈建給鄰近容器的資源。

在您開始建立及使用 Windows 容器時，下列主要概念將有所幫助。

**Container Host:** Physical or Virtual computer system configured with the Windows Container feature. 容器主機將會執行一或多個 Windows 容器。

**容器映像：**對容器檔案系統或登錄進行修改時 (例如進行軟體安裝時)，會在沙箱中加以擷取。 在許多情況下您都可能會想擷取此狀態，以建立繼承這些變更的新容器。 That’s what an image is – once the container has stopped you can either discard that sandbox or you can convert it into a new container image. For example, let’s imagine that you have deployed a container from the Windows Server Core OS image. You then install MySQL into this container. Creating a new image from this container would act as a deployable version of the container. This image would only contain the changes made (MySQL), however would work as a layer on top of the Container OS Image.

**Sandbox:** Once a container has been started, all write actions such as file system modifications, registry modifications or software installations are captured in this ‘sandbox’ layer.

**Container OS Image:** Containers are deployed from images. The container OS image is the first layer in potentially many image layers that make up a container. 此映像提供作業系統環境。 容器 OS 映像不可變更。 也就是說，無法修改。

**容器存放庫：**每當容器映像建立時，容器映像和其相依性即會儲存在本機存放庫中。 這些映像可在容器主機上重複使用多次。 容器映像也可以儲存在公用或私人登錄中 (例如 DockerHub)，以便在許多不同的容器主機之間使用。

<center>![](media/containerfund.png)</center>

對於熟悉虛擬機器的人而言，可能會覺得容器極為相似。 A container runs an operating system, has a file system and can be accessed over a network just as if it was a physical or virtual computer system. 即便如此，容器的基本技術和概念還是與虛擬機器非常不同。

Microsoft Azure 專家 Mark Russinovich 寫過[一篇絕佳的部落格文章](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/)就詳細說明了兩者的差異。

## Windows 容器類型

Windows Containers include two different container types, or runtimes.

**Windows Server Containers** – provide application isolation through process and namespace isolation technology. A Windows Server Container shares a kernel with the container host and all containers running on the host. 這些容器沒有提供惡意防護界限，不應用來隔離未受信任的程式碼。 由於共用核心空間，因此這些容器需要相同的核心版本和組態。

**Hyper-V 隔離** – 藉由在高度最佳化的虛擬機器中執行每個容器，擴充 Windows Server 容器所提供的隔離能力。 In this configuration, the kernel of the container host is not shared with other containers on the same host. 這些容器的設計是要以虛擬機器的相同安全保證來管控惡意多組織用戶共享。 因為這些容器不會與主機或主機上的其他容器共用核心，所以它們可以執行具有不同版本和設定的核心 (支援的版本) - 例如，Windows 10 上的所有 Windows 容器都會使用 Hyper-V 隔離，以便運用 Windows Server 核心版本和設定。

使用或不使用 Hyper-V 隔離在 Windows 上執行容器是執行階段決策。 您可以選擇一開始使用 Hyper-V 隔離來建立容器，之後在執行階段選擇改以 Windows Server 容器的形式來執行。

## 什麼是 Docker？

當您閱讀容器的相關資訊時，一定會看到 Docker。 Docker 是用以封裝和傳遞容器映像的器具。 這個自動化處理序會產生映像 (實際上就是範本)，而且這些映像能以容器的形式在內部部署、雲端或個人電腦等任何環境中執行。

<center>![](media/docker.png)</center>

如同任何其他容器，Windows Server 容器可使用 [Docker](https://www.docker.com) 管理。

## 開發人員的容器 ##

From a developer’s desktop to a testing machine to a set of production machines, a Docker image can be created that will deploy identically across any environment in seconds. This story has created a massive and growing ecosystem of applications packaged in Docker containers, with DockerHub, the public containerized-application registry that Docker maintains, currently publishing more than 180,000 applications in the public community repository.

When you containerize an app, only the app and the components needed to run the app are combined into an "image". Containers are then created from this image as you need them. You can also use an image as a baseline to create another image, making image creation even faster. Multiple containers can share the same image, which means containers start very quickly and use fewer resources. For example, you can use containers to spin up light-weight and portable app components – or ‘micro-services’ – for distributed apps and quickly scale each service separately.

Because the container has everything it needs to run your application, they are very portable and can run on any machine that is running Windows Server 2016. You can create and test containers locally, then deploy that same container image to your company's private cloud, public cloud or service provider. The natural agility of Containers supports modern app development patterns in large scale, virtualized and cloud environments.

With containers, developers can build an app in any language. These apps are completely portable and can run anywhere - laptop, desktop, server, private cloud, public cloud or service provider - without any code changes.  

Containers helps developers build and ship higher-quality applications, faster.

## Containers for IT Professionals ##

IT Professionals can use containers to provide standardized environments for their development, QA, and production teams. They no longer have to worry about complex installation and configuration steps. By using containers, systems administrators abstract away differences in OS installations and underlying infrastructure.

Containers help admins create an infrastructure that is simpler to update and maintain.

## Video Overview

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## 試用 Windows Server 容器

準備好開始運用容器的強大威力了嗎？ 請點選以下連結，開始實際部署您的第一個容器： <br/>
若為 Windows Server 使用者，請前往此處 - [Windows Server 快速入門簡介](../quick-start/quick-start-windows-server.md) <br/>
若為 Windows 10 使用者，請前往此處 - [Windows 10 快速入門簡介](../quick-start/quick-start-windows-10.md)

