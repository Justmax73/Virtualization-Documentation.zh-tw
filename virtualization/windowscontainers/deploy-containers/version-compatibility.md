---
title: Windows 容器版本相容性
description: Windows 要如何跨多個版本執行組建及執行容器
keywords: 中繼資料, 容器, 版本
author: patricklang
ms.openlocfilehash: 8657c03ad71685b0f01532894781c44d76e1b0bc
ms.sourcegitcommit: d69ed13d505e96f514f456cdae0f93dab4fd3746
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "4340876"
---
# <a name="windows-container-version-compatibility"></a>Windows 容器版本相容性

Windows Server 2016 和 Windows 10 年度更新版 (兩者皆為版本 14393) 是第一個可以建置及執行 Windows Server 容器的 Windows 版本。 使用這些版本建置的容器可以在類似 Windows Server (版本 1709) 的較新版本上執行，但是在您開始之前必須先知道幾件事。

我們已改善 Windows 容器的功能，但是我們必須進行一些可能會影響相容性的變更。 較舊的容器將會在具有 [Hyper-V 隔離](../manage-containers/hyperv-container.md)功能的較新主機上以相同方式執行，而且依然會使用相同 (較舊的) 核心版本。 不過，如果您想要根據較新的 Windows 組建執行容器，它只能在較新的主機組建上執行。



<table>
    <tr>
    <th style="background-color:#BBDEFB">容器 OS 版本</th>
    <th span='6' style="background-color:#DCEDC8">主機 OS 版本</th>
    </tr>
    <tr>
        <td/>
        <td style="background-color:#F1F8E9"><b>WindowsServer 2016</b><br/>組建：14393.*</td>
        <td style="background-color:#F1F8E9"><b>Windows 10 1609、1703</b><br/>組建：14393.*、15063.*</td>
        <td style="background-color:#F1F8E9"><b>Windows Server 版本 1709</b><br/>組建：16299.*</td>
        <td style="background-color:#F1F8E9"><b>Windows 10 Fall Creators Update</b><br/>組建：16299.*</td>
        <td style="background-color:#F1F8E9"><b>Windows Server 版本 1803</b><br/>組建 17134.*</td>
        <td style="background-color:#F1F8E9"><b>Windows 10 版本 1803</b><br/>組建 17134.*</td>
        <td style="background-color:#F1F8E9"><b>Windows Server 2019</b><br/>組建 17763.*</td>
        <td style="background-color:#F1F8E9"><b>Windows 10 版本 1809</b><br/>組建 17763.*</td>
    </tr>
    <tr>
        <td style="background-color:#E3F2FD"><b>WindowsServer 2016</b><br/>組建：14393.*</td>
        <td>支援<br/> `process` 或 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
    </tr>
    <tr>
        <td style="background-color:#E3F2FD"><b>Windows Server 版本 1709</b><br/>組建：16299.*</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>支援<br/> `process` 或 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
    </tr>
    <tr>
        <td style="background-color:#E3F2FD"><b>Windows Server 版本 1803</b><br/>組建 17134.*</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>支援<br/> `process` 或 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
    </tr>
    <tr>
        <td style="background-color:#E3F2FD"><b>Windows Server 2019</b><br/>組建 17763.*</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>不支援</td>
        <td>支援<br/> `process` 或 `hyperv` 隔離</td>
        <td>支援<br/> 僅限 `hyperv` 隔離</td>
    </tr>
</table>               

## <a name="matching-container-host-version-with-container-image-versions"></a>使容器主機版本與容器映像版本相符
### <a name="windows-server-containers"></a>Windows Server 容器
Windows Server 容器和基礎主機共用單一核心，因此容器的基本映像必須與主機相符。  如果版本不同，容器仍可啟動，但無法保證可使用完整功能。 Windows 作業系統有 4 個層級的版本設定：主要、次要、組建和修訂，例如 10.0.14393.103。 只有在發行新的 OS 版本，例如版本 1709、1803、Fall Creators Update 等，組建編號才會變更 (亦即 14393) 變更。修訂編號 (亦即 103) 會隨著套用 Windows 更新時進行更新。
#### <a name="build-number-new-release-of-windows"></a>組建編號 (新的 Windows 版本)
容器主機和容器映像之間的組建編號不同時，Windows Server 容器便無法啟動 - 例如 10.0.14393.* (Windows Server 2016) 和 10.0.16299.* (Windows Server 版本 1709)。  
#### <a name="revision-number-patching"></a>修訂編號 (修補)
容器主機和容器映像之間的修訂編號不同時，Windows Server 容器的啟動_不會_被封鎖 - 例如 10.0.14393.1914 (已套用 KB4051033 的 Windows Server 2016) 和 10.0.14393.1944 (已套用 KB4053579 的 Windows Server 2016)。  
針對 Windows Server 2016 主機/映像 – 容器映像的修訂必須符合主機，才能加入至支援的組態中。  從 Windows Server 版本 1709 開始，這不再適用，而且主機和容器映像不需要有相符修訂。  一如往常，建議將系統保持在最新的修補程式和更新。
#### <a name="practical-application"></a>實際應用
範例 1：容器主機執行已套用 KB4041691 的 Windows Server 2016。  部署至此主機的任何 Windows Server 容器都必須以 10.0.14393.1770 容器基本映像為基礎。  如果 KB4053579 已套用至主機，容器映像必須同時更新，才能繼續受到支援。
範例 2：容器主機執行已套用 KB4043961 的 Windows Server 版本 1709。  部署至此主機的任何 Windows Server 容器都必須以 Windows Server 版本 1709 (10.0.16299) 容器基本映像為基礎，但不需要符合主機 KB。  如果 KB4054517 已套用至主機，容器映像不需要更新，不過為了完整解決任何安全性問題，最好還是更新。
#### <a name="querying-version"></a>查詢版本
方法 1：版本 1709 中導入，cmd 提示字元和 `ver` 命令現在會傳回修訂詳細資料。
```
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125] 
```
方法 2：查詢下列登錄機碼：HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion 例如：
```
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```
或
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

若要檢查基本映像使用的版本為何，您可以檢閱 Docker 中樞的標籤，或是映像描述中提供的映像雜湊表。  [Windows 10 更新歷程記錄](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)頁面上列出每個組建與修訂發行的時間。

### <a name="hyper-v-isolation-for-containers"></a>容器的 Hyper-V 隔離
Windows 容器可以在使用或不使用 Hyper-V 隔離的情況下執行。  Hyper-V 隔離會使用最佳化的 VM，在容器周圍建立安全的界限。  每個 Hyper-V 隔離的容器都具有自身的 Windows 核心執行個體，不像標準 Windows 容器會和其他容器與主機共用核心。  因此，您可以在容器主機和映像中使用不同的 OS 版本 (請參閱下面的相容性對照表)。  

若要使用 Hyper-V 隔離來執行容器，只要將 `--isolation=hyperv` 標記新增至您的 docker run 命令即可。

## <a name="errors-from-mismatched-versions"></a>不相符版本的錯誤

如果您嘗試執行不支援的組合，您將會遇到錯誤：

```
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

若要解決此狀況，您可以：

- 根據 `microsoft/nanoserver` 版本重建容器，或是 `microsoft/windowsservercore`
- 如果主機是較新的版本，請使用 `docker run --isolation=hyperv ...`
- 在具有相同 Windows 版本的不同主機上執行

## <a name="choosing-container-os-versions"></a>選擇容器 OS 版本

> 注意："latest" 標記將會隨著 Windows Server 2016 目前的 [LTSC 產品](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview) 一起更新。 如果您希望容器映像符合 Windows Server (版本 1709) 發行版，請閱讀以下資訊。

請務必知道您的用途需要哪一個容器 OS 版本。 如果您使用 Windows Server (版本 1709)，而且想擁有這個版本的最新修補程式，當您在指定您所要的基本 OS 容器映像版本時，應該使用標記 "1709"，就像這樣：

``` Dockerfile
FROM microsoft/windowsservercore:1709
...
```

不過，如果您想要 Windows Server (版本 1709) 的特定修補程式，可以在標記中指定知識庫文章編號。 例如，如果您想要來自 Windows Server (版本 1709) 的 Nano Server 基本 OS 容器映像並且已套用 KB4043961，您將會以這樣的方式指定：

``` Dockerfile
FROM microsoft/nanoserver:1709_KB4043961
...
```

此外，如果您需要來自 Windows Server 2016 的 Nano Server 基本 OS 容器映像，您依然可以使用 "latest" 標記取得這些基本 OS 容器映像的最新版本：

``` Dockerfile
FROM microsoft/nanoserver
...
```
您也可以使用我們之前所用的結構描述來繼續指定完全相同的修補程式，只要在標記中指定 OS 版本即可：

``` Dockerfile
FROM microsoft/nanoserver:10.0.14393.1770
...
```

## <a name="matching-versions-using-docker-swarm"></a>使用 Docker Swarm 比對版本

目前 Docker Swarm 並沒有內建方式可以比對容器所使用的 Windows 版本與符合相同版本的主機。 如果更新服務來使用較新的容器，它將會順利執行。

如果您需要執行多個 Windows 版本一段時間，有兩個方法可以使用。  將 Windows 主機設定為永遠使用 Hyper-V 隔離，或使用標籤限制。

### <a name="finding-a-service-that-wont-start"></a>尋找不會啟動的服務

如果某個服務不會啟動，您會看到 `MODE` 為 `replicated` 但是 `REPLICAS` 將會停滯在 0。 若要了解 OS 版本是否為問題所在，請使用下列命令：

 `docker service ls` - 尋找服務名稱

```
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

`docker service ps <name>` - 取得狀態和最新嘗試。

```
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

如果您看到「啟動容器失敗：...」，您可以查看完整的錯誤及 `docker service ps --no-trunc <container name>`


```
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

這會顯示相同的錯誤 `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`


### <a name="fix---update-the-service-to-use-a-matching-version"></a>修正 - 更新服務，以使用相符的版本

Docker Swarm 有兩個考量事項。 如果您擁有 Compose 檔案，而該檔案的服務會使用不是由您所建立的映像，您可能會想要適當地更新參考。 請參閱底下：

``` docker-compose
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

另一個考量是您所指向的映像是您自己所建立的映像 (例如 contoso/myimage) 時：
``` docker-compose
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```
在此情況下，您會想要使用上一節所述的方法來修改該 dockerfile，而不是 docker-compose 行。

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>緩和措施 - 搭配 Docker Swarm 使用 Hyper-V 隔離

我們有一個提案是根據容器來支援使用 Hyper-V 隔離，但是這部分的程式碼尚未完成。 您可以在 [GitHub](https://github.com/moby/moby/issues/31616) 上追蹤進度。 在完成之前，主機必須設定為永遠透過 Hyper-V 隔離執行。

這需要變更 Docker 服務設定，然後重新啟動 Docker 引擎。

1. 編輯 `C:\ProgramData\docker\config\daemon.json`
2. 新增一行，使用 `"exec-opts":["isolation=hyperv"]`

注意：daemon.json 檔案在預設情況下不存在。 如果您發現這是您在查看目錄時的情況，您必須建立檔案。 然後您會想要複製底下的內容：

```JSON
{
    "exec-opts":["isolation=hyperv"]
}
```
關閉並儲存檔案。 然後重新啟動 docker 引擎：

```PowerShell
Stop-Service docker
Start-Service docker
```

在服務重新啟動之後，啟動您的容器。 一旦容器執行之後，您就可以檢查容器來確認容器的隔離層：

```PowerShell
docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
```

它將會傳回 "process" 或 "hyperv"。 如果您已依照上面所述來修改並設定您的 daemon.json，它應該會顯示後者。

### <a name="mitigation---use-labels-and-constraints"></a>緩和措施 - 使用標籤和限制

**步驟 1 - 為每個節點新增標籤**

在每個節點上，新增兩個標籤 - `OS` 和 `OsVersion`。 這是假設您在本機執行，但是可加以修改，將其改為設定在遠端主機上。

```powershell
docker node update --label-add OS="windows" $ENV:COMPUTERNAME
docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
```

之後，您可以檢查具有 `docker node inspect` 的項目，這些項目將會顯示新增的標籤

```
        "Spec": {
            "Labels": {
                "OS": "windows",
                "OsVersion": "10.0.16296"
            },
            "Role": "manager",
            "Availability": "active"
        }
```

**步驟 2 - 新增服務限制**

一旦標示每個節點之後，我們就可以更新限制來決定服務的位置。 在底下的範例中，將 "contoso_service" 替換為您的實際服務名稱：

```
docker service update \
    --constraint-add "node.labels.OS == windows" \
    --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
    contoso_service
```

這樣會強制執行及限制節點可執行的位置。

如需有關如何使用服務限制的詳細資訊，請參閱服務建立[參考資料](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint)


## <a name="matching-versions-using-kubernetes"></a>使用 Kubernetes 比對版本

在 Kubernetes 中排定 pod 時，可能會發生相同的問題。 您可使用類似策略來避免此問題：

- 根據開發和生產環境中的相同 OS 版本重建容器 - 請參閱上面的**選擇容器 OS 版本**
- 當 Windows Server 2016 和 Windows Server (版本 1709) 節點位在相同的叢集時，使用節點標籤和節點選取器來確定 pod 會排定在相容的節點上
- 根據 OS 版本使用不同的叢集


### <a name="finding-pods-failed-on-os-mismatch"></a>尋找因為 OS 不相符而失敗的 pod

在此情況下 - 部署包含的 pod 排定在具有不相符 OS 版本的節點上，而且未啟用 Hyper-V 隔離。 以 `kubectl describe pod <podname>` 列出的事件中會顯示相同的錯誤。 在幾次嘗試後，pod 狀態可能會是 `CrashLoopBackOff`

```
$ kubectl -n plang describe po fabrikamfiber.web-789699744-rqv6p

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


### <a name="mitigation---using-node-labels--nodeselector"></a>緩和措施 - 使用節點標籤和節點選取器

`kubectl get node` 將會取得所有節點的清單，然後您可以使用 `kubectl describe node <nodename>` 來取得更多詳細資料。 

在此情況下，有兩個 Windows 節點正在執行不同的版本：

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


1. 請記下每個節點名稱及系統資訊中的 `Kernel Version`：

名稱         | 版本
-------------|--------------------------------------------------------
38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354



2. 在每個稱為 `beta.kubernetes.io/osbuild` 的節點中新增標籤。 Windows Server 2016 需要符合主要和次要版本 (14393.1715) 才能在沒有 Hyper-V 隔離的情況下受到支援。 Windows Server (版本 1709) 只需要符合主要版本 (16299)。

從上述範例中：

```
$ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


node "38519acs9010" labeled
$ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

node "38519acs9011" labeled

```

3. 檢查標籤存在，並且有 `kubectl get nodes --show-labels`

```
$ kubectl get nodes --show-labels

NAME                        STATUS                     AGE       VERSION                    LABELS
38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
```

4. 在部署中新增節點選取器

在容器規格中新增 `nodeSelector` 而且 `beta.kubernetes.io/os` = windows 且 `beta.kubernetes.io/osbuild` = 14393.* 或 16299，以符合容器所使用的基本 OS。

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

pod 現在可從更新的部署開始。 節點選取器也會顯示在 `kubectl describe pod <podname>` 中，好讓您可以確認它們已新增。

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
