---
title: Windows 容器版本相容性
description: Windows 要如何跨多個版本執行組建及執行容器
keywords: 中繼資料, 容器, 版本
author: taylorb-microsoft
ms.openlocfilehash: 76549bbfbaf374acb79f1be4280949aecf4e87f0
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610278"
---
# <a name="windows-container-version-compatibility"></a>Windows 容器版本相容性

Windows Server 2016 和 Windows 10 年度更新版 （這兩個版本 14393） 是第一個 Windows 版本可以建置及執行 Windows Server 容器。 使用這些版本建置的容器可以在類似 Windows Server (版本 1709) 的較新版本上執行，但是在您開始之前必須先知道幾件事。

我們已改善 Windows 容器的功能，但是我們必須進行一些可能會影響相容性的變更。 較舊的容器將會執行相同使用[HYPER-V 隔離](../manage-containers/hyperv-container.md)，較新主機上，並將會使用相同的 （較舊的） 核心版本。 不過，如果您想要執行容器，根據較新的 Windows 組建，它只能執行較新的主機組建上。

|容器 OS 版本|主機 OS 版本|相容性|
|---|---|---|
|WindowsServer 2016<br>組建：14393.* |WindowsServer 2016<br>組建：14393.* |支援`process`或`hyperv`隔離|
|WindowsServer 2016<br>組建：14393.* |Windows Server 版本 1709<br>組建：16299.* |僅支援`hyperv`隔離|
|WindowsServer 2016<br>組建：14393.* |Windows 10 Fall Creators Update<br>組建：16299.* |僅支援`hyperv`隔離|
|WindowsServer 2016<br>組建：14393.* |Windows Server 版本 1803<br>組建 17134.* |僅支援`hyperv`隔離|
|WindowsServer 2016<br>組建：14393.* |Windows10 (版本 1803)<br>組建 17134.* |僅支援`hyperv`隔離|
|WindowsServer 2016<br>組建：14393.* |Windows Server 2019<br>組建 17763.* |僅支援`hyperv`隔離|
|WindowsServer 2016<br>組建：14393.* |Windows 10 版本 1809<br>組建 17763.* |僅支援`hyperv`隔離|
|Windows Server 版本 1709<br>組建：16299.* |WindowsServer 2016<br>組建：14393.* |不支援|
|Windows Server 版本 1709<br>組建：16299.* |Windows Server 版本 1709<br>組建：16299.* |支援`process`或`hyperv`隔離|
|Windows Server 版本 1709<br>組建：16299.* |Windows 10 Fall Creators Update<br>組建：16299.* |僅支援`hyperv`隔離|
|Windows Server 版本 1709<br>組建：16299.* |Windows Server 版本 1803<br>組建 17134.* |僅支援`hyperv`隔離|
|Windows Server 版本 1709<br>組建：16299.* |Windows10 (版本 1803)<br>組建 17134.* |僅支援`hyperv`隔離|
|Windows Server 版本 1709<br>組建：16299.* |Windows Server 2019<br>組建 17763.* |僅支援`hyperv`隔離|
|Windows Server 版本 1709<br>組建：16299.* |Windows 10 版本 1809<br>組建 17763.* |僅支援`hyperv`隔離|
|Windows Server 版本 1803<br>組建 17134.* |WindowsServer 2016<br>組建：14393.* |不支援|
|Windows Server 版本 1803<br>組建 17134.* |Windows Server 版本 1709<br>組建：16299.* |不支援|
|Windows Server 版本 1803<br>組建 17134.* |Windows 10 Fall Creators Update<br>組建：16299.* |不支援|
|Windows Server 版本 1803<br>組建 17134.* |Windows Server 版本 1803<br>組建 17134.* |支援`process`或`hyperv`隔離|
|Windows Server 版本 1803<br>組建 17134.* |Windows10 (版本 1803)<br>組建 17134.* |僅支援`hyperv`隔離|
|Windows Server 版本 1803<br>組建 17134.* |Windows Server 2019<br>組建 17763.* |僅支援`hyperv`隔離|
|Windows Server 版本 1803<br>組建 17134.* |Windows 10 版本 1809<br>組建 17763.* |僅支援`hyperv`隔離|
|Windows Server 2019<br>組建 17763.* |WindowsServer 2016<br>組建：14393.* |不支援|
|Windows Server 2019<br>組建 17763.* |Windows Server 版本 1709<br>組建：16299.* |不支援
|Windows Server 2019<br>組建 17763.* |Windows 10 Fall Creators Update<br>組建：16299.* |不支援|
|Windows Server 2019<br>組建 17763.* |Windows Server 版本 1803<br>組建 17134.* |不支援|
|Windows Server 2019<br>組建 17763.* |Windows10 (版本 1803)<br>組建 17134.* |不支援|
|Windows Server 2019<br>組建 17763.* |Windows Server 2019<br>組建 17763.* |支援`process`或`hyperv`隔離|
|Windows Server 2019<br>組建 17763.* |Windows 10 版本 1809<br>組建 17763.* |僅支援`hyperv`隔離|

## <a name="matching-container-host-version-with-container-image-versions"></a>使容器主機版本與容器映像版本相符

### <a name="windows-server-containers"></a>WindowsServer 容器

Windows Server 容器和基礎主機共用單一核心，因此容器的基本映像必須符合主機。 如果兩個版本不同，容器可能會啟動，但不保證的完整功能。 Windows 作業系統有四個層級的版本設定： 主要、 次要、 組建和修訂。 例如，版本 10.0.14393.103 會有的主要版本為 10，0 的次要版本，14393，組建編號和 103 修訂編號。 組建編號才會變更時，例如版本 1709、 1803、 Fall Creators Update 中，會發佈新的 OS 版本等等。 修訂編號會隨著套用 Windows 更新時進行更新。

#### <a name="build-number-new-release-of-windows"></a>組建編號 （新的 Windows 版本）

Windows Server 容器會遭到封鎖，無法啟動時有不同的容器主機和容器映像之間的組建編號。 例如，當容器主機是版本 10.0.14393.* (Windows Server 2016) 和容器映像是版本 10.0.16299.* (Windows Server 版本 1709年)，不會啟動容器。  

#### <a name="revision-number-patching"></a>修訂編號 （修補）

Windows Server 容器未封鎖而無法套用 kb4051033 的容器主機和容器映像的修訂編號不同。 例如，如果容器主機是版本 10.0.14393.1914 (Windows Server 2016 套用 KB4051033) 和容器映像是版本 10.0.14393.1944 (Windows Server 2016 套用 KB4053579)，然後將映像還是會啟動即使其修訂數字是不同。

針對 Windows Server 2016 為基礎的主機或影像，容器映像的修訂必須符合主機，才能加入至支援的組態中。 不過，對於主機或使用 Windows Server 版本 1709年或更高版本的映像，將不會套用此規則，而且主機和容器映像不需要有相符修訂。 我們建議您保留您的系統保持在最新的修補程式和更新的最新狀態。

#### <a name="practical-application"></a>實際應用

範例 1： 容器主機已套用 kb4041691 執行 Windows Server 2016。 部署至此主機的任何 Windows Server 容器必須根據版本 10.0.14393.1770 容器基本映像。 如果您套用 KB4053579 的主機容器時，您也必須更新若要確保主機容器支援它們的映像。

範例 2： 容器主機已套用 kb4043961 執行 Windows Server 版本 1709年。 部署至此主機的任何 Windows Server 容器都必須以基礎 Windows Server 版本 1709 (10.0.16299) 容器基本映像，但不需要符合主機 KB。 如果 KB4054517 已套用至主機，容器映像會仍受支援，但我們建議您更新這些解決任何潛在的安全性問題。

#### <a name="querying-version"></a>查詢版本

方法 1： 會版本 1709年中導入，cmd 提示字元和**ver**命令現在會傳回修訂詳細資料。

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

方法 2： 查詢下列登錄機碼： HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

例如：

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

若要檢查版本為何您的基礎映像使用，檢閱 Docker hub 或映像描述中提供的映像雜湊表上的標籤。 在[Windows 10 更新歷程記錄](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)頁面會列出當每個組建與修訂發行的時間。

### <a name="hyper-v-isolation-for-containers"></a>適用於容器的 HYPER-V 隔離

您可以使用或不使用 HYPER-V 隔離來執行 Windows 容器。 Hyper-V 隔離會使用最佳化的 VM，在容器周圍建立安全的界限。 不像標準 Windows 容器的共用容器和主機之間的核心，每個 HYPER-V 隔離的容器有自己的 Windows 核心執行個體。 這表示您可以有不同的作業系統版本中的容器主機和映像 （如需詳細資訊，請參閱下列的相容性對照表）。  

若要使用 Hyper-V 隔離來執行容器，只要將 `--isolation=hyperv` 標記新增至您的 docker run 命令即可。

## <a name="errors-from-mismatched-versions"></a>不相符版本的錯誤

如果您嘗試執行不受支援的組合，您會收到下列錯誤：

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

有三種方式，您可以解決這個錯誤：

- 重新建置容器基礎的正確版本的`microsoft/nanoserver`或 `microsoft/windowsservercore`
- 如果主機是較新的執行**docker run--隔離 = hyperv...**
- 請嘗試在具有相同的 Windows 版本的不同主機上執行容器

## <a name="choose-which-container-os-version-to-use"></a>選擇要使用的容器 OS 版本

>[!NOTE]
>"Latest"標記將會更新，以及 Windows Server 2016 中，目前的[長期維護通道產品](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview)。 下列指示適用於容器映像符合 Windows Server 版本 1709年版本。

您必須知道您需要適用於您的容器使用哪一個版本。 例如，如果您使用 Windows Server 版本 1709 開始，並想要為其有最新的修補程式，您應該使用標記`1709`時指定您要哪一個版本的基本 OS 容器映像，像這樣：

``` dockerfile
FROM microsoft/windowsservercore:1709
...
```

不過，如果您想要 Windows Server 版本 1709年的特定修補程式，您可以在標記中指定知識庫文章編號。 例如，若要取得從 Windows Server 版本 1709年 kb4043961 套用到它的 Nano Server 基本 OS 容器映像，您會指定它就像這樣：

``` dockerfile
FROM microsoft/nanoserver:1709_KB4043961
...
```

如果您需要 Nano Server 基本 OS 容器映像從 Windows Server 2016，則您仍然可以透過使用"latest"標記取得這些基本 OS 容器映像的最新版本：

``` dockerfile
FROM microsoft/nanoserver
...
```

您也可以指定您需要的確切修補程式與我們先前使用過，藉由在標記中指定的 OS 版本的結構描述：

``` dockerfile
FROM microsoft/nanoserver:10.0.14393.1770
...
```

## <a name="matching-versions-using-docker-swarm"></a>使用 Docker Swarm 比對版本

Docker 群集目前沒有內建的方式，以符合容器所使用的主機時相同版本的 Windows 版本。 如果您更新服務，以使用較新的容器，它會成功執行。

如果您需要執行多個 Windows 版本長一段時間，有兩個您可以採取的方法： 無論是 Windows 主機設定為永遠使用 HYPER-V 隔離，或使用標籤限制。

### <a name="finding-a-service-that-wont-start"></a>尋找不會啟動的服務

如果某個服務不會啟動，您會看到`MODE`是`replicated`，但`REPLICAS`將會停滯在 0。 若要查看的 OS 版本是否為問題所在，請執行下列命令：

執行**docker 服務 ls**來尋找服務名稱：

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

執行**docker 服務 ps （服務名稱）** 取得最新嘗試與狀態：

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

如果您看到`starting container failed: ...`，您可以查看完整的錯誤及**docker 服務 ps-否 trunc （容器名稱）**：

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

這是做為相同的錯誤`CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`。

### <a name="fix---update-the-service-to-use-a-matching-version"></a>修正 - 更新服務，以使用相符的版本

Docker Swarm 有兩個考量事項。 在您擁有 compose 檔案的服務會使用未建立的映像的案例中，您會想要適當地更新參考。 例如：

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

其他考量是您所指向的映像的您自己所建立的其中一個 (例如，contoso/myimage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

在此情況下，您應該使用[不相符版本的錯誤](#errors-from-mismatched-versions)中所述的方法來修改該 dockerfile，而不是 docker-compose 行。

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>緩和措施 - 搭配 Docker Swarm 使用 Hyper-V 隔離

有一個提案是來支援使用 HYPER-V 隔離，以每個容器為基礎，但是尚未完成的程式碼。 您可以在 [GitHub](https://github.com/moby/moby/issues/31616) 上追蹤進度。 在完成之前，主機必須設定為永遠透過 Hyper-V 隔離執行。

這需要變更 Docker 服務設定，然後重新啟動 Docker 引擎。

1. 編輯 `C:\ProgramData\docker\config\daemon.json`
2. 新增一行，使用 `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >根據預設在 daemon.json 檔案不存在。 如果您發現這是您在查看目錄時的情況，您必須建立檔案。 然後您會想要複製的下列：

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. 關閉並儲存檔案，然後重新啟動 docker 引擎在 PowerShell 中執行下列 cmdlet:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. 您已重新啟動服務之後，啟動您的容器。 他們正在執行之後，您可以檢查下列 cmdlet 的容器來驗證容器的隔離層級：

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

它將會傳回 "process" 或 "hyperv"。 如果您已依照上面所述來修改並設定您的 daemon.json，它應該會顯示後者。

### <a name="mitigation---use-labels-and-constraints"></a>緩和措施 - 使用標籤和限制

以下是如何使用標籤和限制，以符合版本：

1. 將標籤新增至每個節點。

    每個節點上，新增兩個標籤：`OS`和`OsVersion`。 這是假設您在本機執行，但是可加以修改，將其改為設定在遠端主機上。

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    之後，您可以檢查這些執行**docker 節點檢查**命令，應該會顯示剛新增的標籤：

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. 新增服務限制。

    既然您已經標示為每個節點，您可以更新限制來決定服務的位置。 在下列範例中，請使用您的實際服務的名稱取代"contoso_service 為"：

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    這樣會強制執行及限制節點可執行的位置。

若要深入了解如何使用服務的限制，請查看[服務建立的參考](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)。

## <a name="matching-versions-using-kubernetes"></a>使用 Kubernetes 比對版本

在 Kubernetes 中排定 pod 時，可能會發生相同的問題，[使用 Docker 群集符合版本](#matching-versions-using-docker-swarm)中所述。 使用類似策略，可避免此問題：

- 重新建置容器根據開發和生產環境中的相同 OS 版本。 若要了解如何執行，請參閱[選擇哪一個容器 OS 版本來使用](#choose-which-container-os-version-to-use)。
- 使用節點標籤和節點選取器來確定 pod 會排定在相容的節點上，如果 Windows Server 2016 和 Windows Server 版本 1709年節點位在相同的叢集
- 根據 OS 版本使用不同的叢集

### <a name="finding-pods-failed-on-os-mismatch"></a>尋找因為 OS 不相符而失敗的 pod

在此情況下，部署包含的 pod 排定在具有不相符 OS 版本，而且未啟用的 HYPER-V 隔離的節點上。

以 `kubectl describe pod <podname>` 列出的事件中會顯示相同的錯誤。 在幾次嘗試後，pod 狀態可能會是`CrashLoopBackOff`。

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>緩和措施-使用節點標籤和節點選取器

執行**kubectl 取得節點**以取得所有節點的清單。 在此之後，您可以執行**kubectl 描述節點 （節點名稱）** 以取得詳細資料。

在下列範例中，兩個 Windows 節點執行不同版本：

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

讓我們使用此範例顯示如何符合的版本：

1. 記下每個節點名稱和`Kernel Version`系統資訊。

    在我們的範例，資訊看起來像這樣：

    名稱         | 版本
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. 在每個稱為 `beta.kubernetes.io/osbuild` 的節點中新增標籤。 Windows Server 2016 需要支援不含 HYPER-V 隔離的主要和次要版本 (14393.1715 在此範例中)。 Windows Server 版本 1709年只需要以符合的主要版本 (16299 在此範例中)。

    在此範例中，新增標籤命令看起來像這樣：

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. 檢查標籤，並且有執行**kubectl 取得節點-顯示標籤**。

    在此範例中，輸出看起來像這樣：

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. 部署中新增節點選取器。 在這個範例案例中，我們將新增`nodeSelector`容器規格`beta.kubernetes.io/os`= windows 和`beta.kubernetes.io/osbuild`= 14393.* 或 16299，以符合容器所使用的基本 OS。

    以下的完整範例是用來執行專為 Windows Server 2016 所打造的容器：

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    pod 現在可從更新的部署開始。 節點選取器也會顯示在`kubectl describe pod <podname>`，因此您可以執行來確認它們已新增該命令。

    針對我們的範例輸出如下所示：

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```
