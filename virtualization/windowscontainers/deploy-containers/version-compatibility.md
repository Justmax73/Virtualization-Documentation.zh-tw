---
title: Windows 容器版本相容性
description: Windows 要如何跨多個版本執行組建及執行容器
keywords: 中繼資料, 容器, 版本
author: taylorb-microsoft
ms.openlocfilehash: 1f068cd011b2172e75c240d566473ccab25d984a
ms.sourcegitcommit: 48ede8f27e089926b5b867037f31d14500af84ce
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/14/2019
ms.locfileid: "10296028"
---
# <a name="windows-container-version-compatibility"></a>Windows 容器版本相容性

Windows Server 2016 和 Windows 10 周年紀念更新（兩個版本14393）是可以建立並執行 Windows Server 容器的第一項 Windows 版本。 使用這些版本建立的容器可以在較新的版本上執行，但在開始之前，您需要知道幾個事項。

我們已改善 Windows 容器的功能，但是我們必須進行一些可能會影響相容性的變更。 較舊的容器會在使用[hyper-v 隔離](../manage-containers/hyperv-container.md)的較新主機上執行相同，且會使用相同（較舊）的內核版本。 不過，如果您想要根據較新的 Windows 組建來執行容器，它只能在較新的主機組建上執行。

## <a name="windows-server-host-os-compatibility"></a>Windows Server 主機作業系統相容性

<!-- start tab view -->
# [<a name="windows-server-version-1909"></a>Windows Server、版本1909](#tab/windows-server-1909)

|容器基底影像 OS 版本|支援 Hyper-v 隔離|支援進程隔離|
|---|:---:|:---:|
|Windows Server、版本1909|&#10004;|&#10004;|
|Windows Server、版本1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|WindowsServer 2016|&#10004;|&#10060;|

# [<a name="windows-server-version-1903"></a>Windows Server、版本1903](#tab/windows-server-1903)

|容器基底影像 OS 版本|支援 Hyper-v 隔離|支援進程隔離|
|---|:---:|:---:|
|Windows Server、版本1909|&#10060;|&#10060;|
|Windows Server、版本1903|&#10004;|&#10004;|
|Windows Server 2019|&#10004;|&#10060;|
|WindowsServer 2016|&#10004;|&#10060;|

# [<a name="windows-server-2019"></a>Windows Server 2019](#tab/windows-server-2019)

|容器基底影像 OS 版本|支援 Hyper-v 隔離|支援進程隔離|
|---|:---:|:---:|
|Windows Server、版本1909|&#10060;|&#10060;|
|Windows Server、版本1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10004;|
|WindowsServer 2016|&#10004;|&#10060;|

# [<a name="windows-server-2016"></a>WindowsServer 2016](#tab/windows-server-2016)

|容器基底影像 OS 版本|支援 Hyper-v 隔離|支援進程隔離|
|---|:---:|:---:|
|Windows Server、版本1909|&#10060;|&#10060;|
|Windows Server、版本1903|&#10060;|&#10060;|
|Windows Server 2019|&#10060;|&#10060;|
|WindowsServer 2016|&#10004;|&#10004;|

---
<!-- stop tab view -->

## <a name="windows-10-host-os-compatibility"></a>Windows 10 主機作業系統相容性

<!-- start tab view -->

# [<a name="windows-10-version-1909"></a>Windows 10 版本1909](#tab/windows-10-1909)

|容器基底影像 OS 版本|支援 Hyper-v 隔離|支援進程隔離|
|---|:---:|:---:|
|Windows Server、版本1909|&#10004;|&#10060;|
|Windows Server、版本1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|WindowsServer 2016|&#10004;|&#10060;|

# [<a name="windows-10-version-1903"></a>Windows 10 版本1903](#tab/windows-10-1903)

|容器基底影像 OS 版本|支援 Hyper-v 隔離|支援進程隔離|
|---|:---:|:---:|
|Windows Server、版本1909|&#10060;|&#10060;|
|Windows Server、版本1903|&#10004;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|WindowsServer 2016|&#10004;|&#10060;|

# [<a name="windows-10-version-1809"></a>Windows 10 版本 1809](#tab/windows-10-1809)

|容器基底影像 OS 版本|支援 Hyper-v 隔離|支援進程隔離|
|---|:---:|:---:|
|Windows Server、版本1909|&#10060;|&#10060;|
|Windows Server、版本1903|&#10060;|&#10060;|
|Windows Server 2019|&#10004;|&#10060;|
|WindowsServer 2016|&#10004;|&#10060;|

---
<!-- stop tab view -->

## <a name="matching-container-host-version-with-container-image-versions"></a>與容器影像版本相符的容器主機版本

### <a name="windows-server-containers"></a>WindowsServer 容器

因為 Windows Server 容器和基礎主機共用單一內核，所以容器的基本映射 OS 版本必須符合主機的配置。 如果版本不同，容器可能會啟動，但不保證完整功能。 Windows 作業系統有四個層級的版本設定：主要、次要、組建及修訂。 例如，版本10.0.14393.103 的主要版本為10，次要版本為0，組建編號為14393，而修正版本號為103。 只有在發行新版作業系統（例如版本1709、1903等）時，才會變更組建編號。 修訂編號會隨著套用 Windows 更新時進行更新。

#### <a name="build-number-new-release-of-windows"></a>組建編號（新版本的 Windows）

當容器主機與容器影像之間的組建編號不同時，系統會封鎖 Windows Server 容器，無法啟動。 例如，如果容器主機是版本 10.0.14393. * （Windows Server 2016），而容器影像是版本 10.0.16299. * （Windows Server 版本1709），則無法啟動容器。  

#### <a name="revision-number-patching"></a>修訂編號（修補）

當容器主機的修訂號碼與容器影像不同時，系統不會封鎖 Windows Server 容器。 例如，如果容器主機是版本10.0.14393.1914 （已套用 KB4051033 的 Windows Server 2016），而容器影像是版本10.0.14393.1944 （套用 KB4053579 的 Windows Server 2016），則即使其修訂後，影像仍會啟動。數位不同。

對於 Windows Server 2016 的主機或影像，容器影像的修訂版本必須符合主機，才能使用受支援的配置。 不過，對於使用 Windows Server 版本1709及更高版本的主機或影像，此規則不適用，且主機和容器影像不需要有相符的修訂。 我們建議您使用最新的修補程式與更新，讓您的系統保持在最新狀態。

#### <a name="practical-application"></a>實用的應用程式

範例1：容器主機在已套用 KB4041691 的情況下執行 Windows Server 2016。 部署至此主機的任何 Windows Server 容器，都必須以版本10.0.14393.1770 容器基底影像為基礎。 如果您將 KB4053579 套用至主機容器，您也必須更新影像，以確保主機容器支援它們。

範例2：容器主機在已套用 KB4043961 的情況下執行 Windows Server 版本1709。 部署至此主機的任何 Windows Server 容器，都必須以 Windows Server 版本1709（10.0.16299）容器基本影像為基礎，但不需要與主機 KB 相符。 如果 KB4054517 已套用到主機，容器影像仍會受到支援，但我們建議您更新這些影像，以解決任何潛在的安全性問題。

#### <a name="querying-version"></a>查詢版本

方法1：在版本1709中引入，cmd 提示和**ver**命令現在會傳回修訂詳細資料。

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

方法2：查詢下列登錄機碼： HKEY_LOCAL_MACHINE \Software\Microsoft\Windows NT\CurrentVersion

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

若要檢查您的基礎映射所使用的版本，請查看 Docker 中樞上的標記，或影像描述中提供的影像雜湊表。 [ [Windows 10 更新記錄](https://support.microsoft.com/help/12387/windows-10-update-history)] 頁面會列出每個版本發行時的清單。

### <a name="hyper-v-isolation-for-containers"></a>容器的 hyper-v 隔離

您可以在沒有 Hyper-v 隔離的情況下執行 Windows 容器。 Hyper-V 隔離會使用最佳化的 VM，在容器周圍建立安全的界限。 與在容器和主機之間共用內核的標準 Windows 容器不同，每個 Hyper-v 隔離容器都有自己的 Windows 內核實例。 這表示您可以在容器主機和影像中使用不同的 OS 版本（如需詳細資訊，請參閱下列相容性矩陣）。  

若要使用 Hyper-V 隔離來執行容器，只要將 `--isolation=hyperv` 標記新增至您的 docker run 命令即可。

## <a name="errors-from-mismatched-versions"></a>不相符版本的錯誤

如果您嘗試執行不受支援的組合，您會收到下列錯誤：

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

有三種方法可以解決此錯誤：

- 根據正確版本的`mcr.microsoft.com/windows/nanoserver` or 重建容器 `mcr.microsoft.com/windows/servercore`
- 如果主機較新，請執行**docker 執行--隔離 = hyperv ...**
- 嘗試在另一個有相同 Windows 版本的主機上執行容器

## <a name="choose-which-container-os-version-to-use"></a>選擇要使用的容器 OS 版本

>[!NOTE]
>從2019年4月16日起，不會再針對[Windows 基本 OS 容器影像](https://hub.docker.com/_/microsoft-windows-base-os-images)發佈或維護「最新」標記。 從這些存放庫中拉或參照影像時，請宣告特定的標籤。

您必須知道您需要使用哪個版本的容器。 例如，如果您想要將 Windows Server 版本1809作為您的容器作業系統，並想要提供最新的修補程式，您應該`1809`使用標籤來指定您想要的基本 OS 容器影像版本，如下所示：

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

不過，如果您想要 Windows Server 版本1809的特定修補程式，您可以在標記中指定 KB 數位。 例如，若要從 Windows Server 版本1809取得 Nano Server 基本 OS 容器影像，請將 KB4493509 加以指定，如下所示：

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

您也可以透過在標記中指定 OS 版本，來指定您所需的完整修補程式：

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

以 Windows Server 2019 和 Windows Server 2016 為基礎的伺服器核心基礎映射是[長期服務通道（LTSC）](https://docs.microsoft.com/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc)發行。 如果您想要將 Windows Server 2019 作為伺服器核心影像的容器作業系統，並想要取得最新的修補程式，您可以指定 LTSC 版本，如下所示：

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>使用 Docker Swarm 比對版本

Docker Swarm 目前沒有內建的方式，可以將容器使用的 Windows 版本與相同版本的主機相符。 如果您將服務更新為使用較新的容器，則會成功執行。

如果您需要在一段較長的時間內執行多個 Windows 版本，您可以採取兩種方法：將 Windows 主機設定為使用 Hyper-v 隔離，或使用標籤限制式。

### <a name="finding-a-service-that-wont-start"></a>尋找不會啟動的服務

如果服務無法啟動，您會看到 [ `MODE` is `replicated` ]，但`REPLICAS`會停滯在0。 若要查看作業系統版本是否有問題，請執行下列命令：

若要尋找服務名稱，請執行**docker 服務 ls** ：

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

執行**docker 服務 ps （服務名稱）** 以取得狀態及最新嘗試：

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

如果您看到`starting container failed: ...`這個畫面，您可以看到與**docker 服務 ps 的完整錯誤--無 trunc （容器名稱）**：

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

這與相同的錯誤`CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`。

### <a name="fix---update-the-service-to-use-a-matching-version"></a>修正 - 更新服務，以使用相符的版本

Docker Swarm 有兩個考量事項。 如果您有一個撰寫檔案，且該檔案的服務使用您未建立的影像，您會想要對參照進行相應的更新。 例如：

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

另一個考慮是，如果您指向的是自己所建立的影像（例如 contoso/myimage）：

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

在這種情況下，您應該使用不[匹配版本](#errors-from-mismatched-versions)中的錯誤所述的方法來修改該 dockerfile，而不是顯示 docker 組成的線條。

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>緩和措施 - 搭配 Docker Swarm 使用 Hyper-V 隔離

針對每個容器，都有一個支援使用 Hyper-v 隔離的建議，但程式碼尚未完成。 您可以在 [GitHub](https://github.com/moby/moby/issues/31616) 上追蹤進度。 在完成之前，主機必須設定為永遠透過 Hyper-V 隔離執行。

這需要變更 Docker 服務設定，然後重新啟動 Docker 引擎。

1. 編輯 `C:\ProgramData\docker\config\daemon.json`
2. 新增一行，使用 `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >預設不存在該守護程式的 json 檔案。 如果您發現這是您在查看目錄時的情況，您必須建立檔案。 接著，您將會想要複製下列專案：

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. 關閉並儲存檔案，然後在 PowerShell 中執行下列 Cmdlet 來重新開機 docker 引擎：

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. 重新開機服務之後，請啟動您的容器。 執行完後，您可以透過下列 Cmdlet 來檢查容器，以驗證容器的隔離層級：

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

它將會傳回 "process" 或 "hyperv"。 如果您已依照上面所述來修改並設定您的 daemon.json，它應該會顯示後者。

### <a name="mitigation---use-labels-and-constraints"></a>緩和措施 - 使用標籤和限制

以下說明如何使用標籤與限制搭配版本：

1. 在每個節點中新增標籤。

    在每個節點上，新增兩`OS`個`OsVersion`標籤：與。 這是假設您在本機執行，但是可加以修改，將其改為設定在遠端主機上。

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    之後，您可以執行**docker 節點檢查**命令來檢查這些專案，這會顯示新新增的標籤：

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

2. 新增服務限制式。

    現在您已標示每個節點，您可以更新決定放置服務的限制。 在下列範例中，將 [contoso_service] 取代為您實際服務的名稱：

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    這樣會強制執行及限制節點可執行的位置。

若要深入瞭解如何使用服務限制，請查看[服務建立參考](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)。

## <a name="matching-versions-using-kubernetes"></a>使用 Kubernetes 比對版本

在 Kubernetes 中排程盒時，[使用 Docker Swarm 的匹配版本](#matching-versions-using-docker-swarm)中所述的相同問題會發生。 您可以使用類似的戰略來避免這個問題：

- 根據開發和生產中相同的 OS 版本來重建容器。 若要瞭解做法，請參閱[選擇要使用的容器 OS 版本](#choose-which-container-os-version-to-use)。
- 如果 Windows Server 2016 和 Windows Server 版本1709節點都在相同的群集中，請使用節點標籤和 nodeSelectors，以確保在相容節點上排程盒
- 根據 OS 版本使用不同的叢集

### <a name="finding-pods-failed-on-os-mismatch"></a>尋找因為 OS 不相符而失敗的 pod

在這種情況下，部署包含已在具有不匹配 OS 版本的節點上排程的 pod，且未啟用 Hyper-v 隔離。

以 `kubectl describe pod <podname>` 列出的事件中會顯示相同的錯誤。 幾次嘗試之後，盒的狀態就很`CrashLoopBackOff`可能就是。

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

### <a name="mitigation---using-node-labels-and-nodeselector"></a>緩解-使用節點標籤與 nodeSelector

執行**kubectl [取得節點**]，取得所有節點的清單。 之後，您可以執行**kubectl 描述節點（節點名稱）** 來取得更多詳細資料。

在下列範例中，有兩個 Windows 節點正在執行不同的版本：

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

我們來使用這個範例來示範如何與版本相符：

1. 記下每個節點名稱， `Kernel Version`並從系統資訊中記錄。

    在我們的範例中，資訊看起來會像這樣：

    名稱         | 版本
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. 在每個稱為 `beta.kubernetes.io/osbuild` 的節點中新增標籤。 Windows Server 2016 需要主要和次要版本（在此範例中為14393.1715），才能在沒有 Hyper-v 隔離的情況下受到支援。 Windows Server 版本1709只需要主要版本（在此範例中為16299）來相符。

    在這個範例中，新增標籤的命令看起來像這樣：

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. 若要查看標籤，請執行**kubectl [取得節點]--顯示標籤**。

    在這個範例中，輸出看起來會像這樣：

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. 新增節點選擇器至部署。 在這個範例中，我們會將 [ `nodeSelector` `beta.kubernetes.io/os` = windows] 和`beta.kubernetes.io/osbuild` = 14393. * 或16299新增至容器規格，以符合容器所使用的基作業系統。

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

    pod 現在可從更新的部署開始。 節點選取器也會顯示在`kubectl describe pod <podname>`中，因此您可以執行該命令來驗證它們已新增。

    我們的範例輸出如下所示：

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
