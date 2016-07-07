---
title: "進行中的 Windows 容器工作"
description: "進行中的 Windows 容器工作"
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5d9f1cb4-ffb3-433d-8096-b085113a9d7b
redirect_url: ../containers_welcome
translationtype: Human Translation
ms.sourcegitcommit: e3f5535594123f6b4f8931e41a91d92f3b837814
ms.openlocfilehash: 085bb8c0158aedf4270cf2423114ec1901af1ebd

---

# 進行中的工作

如果您在這裡沒有看到您問題的相關討論，或有任何問題，請將問題張貼在[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)。

-----------------------

## 一般功能

### 容器和主機組建編號相符
Windows 容器需有一個作業系統映像，且映像的組建和修補程式等級需與容器主機相符。 不相符會導致潛在的不穩定，或容器和/或主機發生無法預期的行為。

如果您安裝 Windows 容器主機 OS 的更新，則必須更新容器基底 OS 映像，使其符合更新。
<!-- Can we give examples of behavior or errors?  Makes it more searchable -->

**因應措施：**   
下載並安裝符合容器主機的 OS 版本及修補程式等級的容器基底映像。

### 防火牆預設行為
在容器主機和容器環境中，您只有容器主機的防火牆。 在容器主機中設定的所有防火牆規則都會傳播到它的所有容器。

### Windows 容器啟動緩慢
如果您的容器花 30 秒以上才能啟動，它可能正在執行許多重複的病毒掃描。

許多反惡意程式解決方案，例如 Windows Defender，可能會對容器映像進行不必要的檔案掃描，包括容器 OS 映像中所有的 OS 二進位檔和檔案。  建立新的容器時便會發生這種情況，因為從反惡意程式的觀點來看，所有「容器的檔案」看起來都像之前沒掃描過的新檔案。  因此，當容器內的處理序嘗試讀取這些檔案，反惡意程式元件會先掃描它們，才能允許檔案的存取。  實際上，在容器映像匯入或提取到伺服器時，已經掃描過這些檔案。 對於 Windows Server Technical Preview 5，會採用新的基礎結構，讓 Windows Defender 等反惡意程式解決方案能了解這些情況，並據此採取行動以免多次掃描。 反惡意程式碼解決方案可以使用[這裡](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)所述的指南來更新其解決方案，並避免多次掃描。 

### 如果限制記憶體 < 48 MB，有時會啟動/停止失敗
當記憶體限制為小於 48 MB 時，Windows 容器會發生隨機的不一致錯誤。

執行下列 PowerShell 並重複多次啟動、停止動作，會導致啟動或停止失敗。

```PowerShell
new-container "Test" -containerimagename "WindowsServerCore" -MaximumBytes 32MB
start-container test
stop-container test
```

**因應措施：**  
將記憶體值變更為 48 MB。 


### 當 4 核心 VM 上的處理器計數為 1 或 2，啟動容器會失敗

Windows 容器啟動失敗，出現錯誤訊息：  
`failed to start: This operation returned because the timeout period expired. (0x800705B4).`

當 4 核心 VM 上的處理器計數設為 1 或 2，就會發生此情況。

``` PowerShell
new-container "Test2" -containerimagename "WindowsServerCore"
Set-ContainerProcessor -ContainerName test2 -Maximum 2
Start-Container test2

Start-Container : 'test2' failed to start.
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4).
'test2' failed to start. (Container ID 133E9DBB-CA23-4473-B49C-441C60ADCE44)
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4). (Container ID
133E9DBB-CA23-4473-B49C-441C60ADCE44)
At line:1 char:1
+ Start-Container test2
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (:) [Start-Container], VirtualizationException
    + FullyQualifiedErrorId : OperationTimeout,Microsoft.Containers.PowerShell.Cmdlets.StartContainer
PS C:\> Set-ContainerProcessor -ContainerName test2 -Maximum 3
PS C:\> Start-Container test2
```

**因應措施：**  
增加容器可使用的處理器、不要明確指定容器可使用的處理器、或減少 VM 可使用的處理器。

--------------------------

## 網路

### 網路區間隔離和影響
每個容器使用一個網路區間來提供隔離。 所有連接到指定容器的容器網路介面卡 (端點)，都將位於相同的網路區間。 根據使用的網路模式 (驅動程式)，您可能無法使用相同的 IP 位址或連接埠來存取兩個不同的容器端點。 此外，Windows 防火牆規則無法感知區間或容器，因此任何套用的防火牆規則將適用於容器主機上的所有容器，而無視於特定端點。

*** 透明網路 ***


***NAT 網路*** – 您可能會使用 NAT 連接埠轉送規則，公開附加至單一容器的多個端點，這些轉送規則是以每個端點為基礎來套用。 這些轉送規則對應到相同的內部連接埠 (在容器中) 時，必須使用不同的外部連接埠 (在容器主機上)。  不過，如上所述，套用的防火牆規則將會具有跨容器主機的全域範圍。



### 靜態 NAT 對應可能會與透過 Docker 的連接埠對應相衝突
從 Windows Server Technical Preview 5 開始，NAT 建立和連接埠對應規則已整合至 *ContainerNetwork* Cmdlet 和 Docker 命令。 Windows 主機網路服務 (HNS) 會代表您管理 NAT。 不過，有可能外部用戶端會嘗試並使用 HNS 所建立的相同 NAT 來建立重複的連接埠對應規則。


以下範例是連接埠 80 上的靜態對應衝突，以及發生此情況時 Docker 報告的錯誤。
```
C:\Users\Administrator>docker run -it -p 80:80 windowsservercore cmd
docker: Error response from daemon: failed to create endpoint berserk_bassi on network nat: hnsCall failed in Win32: The remote procedure call failed. (0x6be).
```


***緩和*** – 一般不太可能發生此情形，因為 HNS 會管理 NAT。 所有的連接埠轉送/對應規則應該使用 `docker run -p <external>:<internal>` 或 Add-ContainerNetworkAdapterStaticMapping 來建立。 不過，如果 HNS 未自動清除對應，使用 PowerShell 來移除連接埠對應可能可以解決此錯誤。 如此可解決上述範例中所引發的連接埠 80 衝突。
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Windows 容器沒收到 IP
如果您要使用 DHCP VM 交換器連線到 Windows 容器，有可能容器主機收到 IP 但容器沒收到。

容器取得 169.254.***.*** APIPA IP 位址。

**解決辦法：**這是共用核心的副作用。  所有容器實際上具有相同的 MAC 位址。

在裝載容器主機 VM 的實體主機上啟用 MAC 位址詐騙。

這可以使用 PowerShell 來達成
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
--------------------------

## 應用程式相容性

因為有許多關於在 Windows 容器中哪些應用程式可以運作哪些不可以的問題，我們決定將應用程式相容性的資訊分割成一篇[獨立文章](../reference/app_compat.md)。

在這裡也可以找到最常見的問題。

### 容器內沒有事件記錄檔

PowerShell 命令，例如 `get-eventlog Application`，以及查詢事件記錄檔的 API 會傳回類似如下的錯誤︰
```
get-eventlog : Cannot open log Application on machine .. Windows has not provided an error code.
At line:1 char:1
```

若要解決，您可以將此步驟新增至 Dockerfile。 建置時包含此步驟的映像將會啟用事件記錄。
```
RUN powershell.exe -command Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\WMI\Autologger\EventLog-Application Start 1
```


### LocalDB 執行個體 API 方法呼叫內發生意外的錯誤
LocalDB 執行個體 API 方法呼叫內發生意外的錯誤

### RTerm 無法運作
RTerm 已安裝，但無法在 Windows Server 容器內啟動。

錯誤：  
```
'C:\Program' is not recognized as an internal or external command,
operable program or batch file.
```


### 容器：Visual C++ Runtime x64/x86 2015 未安裝

看到的行為：在容器內：
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

這是重複資料刪除篩選器的互通性問題。 重複資料刪除會檢查重新命名目標，查看它是否為已重複資料刪除的檔案。 建立失敗就會發出 `STATUS_IO_REPARSE_TAG_NOT_HANDLED`，因為 Windows Server 容器篩選器設定在重複資料刪除之上。


如需可以將哪些應用程式容器化的相關資訊，請參閱[應用程式相容性文件](../reference/app_compat.md)。

--------------------------


## Docker 管理

### 並非所有的 Docker 命令都能運作
* Docker 在 Hyper-V 容器中執行失敗。

如果不在此清單上的任何項目失敗 (或如果命令失敗原因與預期不同)，請透過[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)讓我們知道。

### 在互動式 Docker 工作階段貼上的命令限制為 50 個字元
在互動式 Docker 工作階段貼上的命令限制為 50 個字元。  
如果複製命令到互動式 Docker 工作階段，目前有 50 個字元的限制。 超過的貼上字串會被截斷。

這不是原始設計，我們正努力改善這項限制。

### net use 錯誤
net use 傳回系統錯誤 1223，而不會提示輸入使用者名稱或密碼

**因應措施：**  
執行 net use 時，指定使用者名稱和密碼。

``` PowerShell
net use S: \\your\sources\here /User:shareuser [yourpassword]
``` 


--------------------------



## 遠端桌面 

在 TP5 中無法透過 RDP 工作階段來管理 Windows 容器或與之互動。

--------------------------

### 無法以 "exit" 來結束 Nano Server 容器主機中的容器
如果您嘗試結束 Nano Server 容器主機中的容器，使用 "exit" 會讓您中斷與 Nano Server 容器主機的連線，但不會結束容器。

**解決辦法：**請改用 Exit-PSSession 來結束容器。

歡迎您在[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中提出功能要求。 


--------------------------


## 使用者和網域

### 本機使用者
可以建立本機使用者帳戶，並用於執行容器中的 Windows 服務和應用程式。


### 網域成員資格
容器無法加入 Active Directory 網域，而且無法以網域使用者、服務帳戶或電腦帳戶的身分執行服務或應用程式。 

容器的設計為可快速啟動至已知的一致狀態，且大致不受環境限制。 加入網域並套用網域的群組原則設定會增加啟動容器所需的時間、持續變更其運作方式，以及限制在開發人員之間及跨部署移動或共用映像的能力。

我們一直都很仔細考慮有關服務和應用程式如何使用 Active Directory 以及在容器中部署這些項目之交集的意見反應。 如果您有最適合您之使用方式的詳細資訊，請在[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中與我們分享。 

我們正積極尋求解決方案來支援這些案例類型。



<!--HONumber=Jun16_HO5-->


