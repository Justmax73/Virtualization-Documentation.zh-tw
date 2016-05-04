



# 進行中的工作

如果您在這裡沒有看到您的問題的相關討論，或有任何問題，請將問題張貼在[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)。

-----------------------


## 一般功能

### 容器和主機組建編號相符

Windows 容器需有一個作業系統映像，且映像的組建和修補程式等級需與容器主機相符。 不相符會導致潛在的不穩定，或容器和/或主機發生無法預期的行為。

如果您安裝 Windows 容器主機 OS 的更新，則必須更新容器基底 OS 映像，使其符合更新。


**因應措施：**   
下載並安裝符合容器主機的 OS 版本及修補程式等級的容器基底映像。

### 防火牆預設行為

在容器主機和容器環境中，您只有容器主機的防火牆。 在容器主機中設定的所有防火牆規則都會傳播到它的所有容器。

### Windows 容器啟動緩慢

如果您的容器花 30 秒以上才能啟動，它可能正在執行許多重複的病毒掃描。

許多反惡意程式解決方案，例如 Windows Defender，可能會對容器映像進行不必要的掃描檔案，包括容器 OS 映像中所有的 OS 二進位檔和檔案。 建立新的容器時便會發生這種情況，因為從反惡意程式的觀點來看，所有「容器的檔案」看起來不都像之前沒掃描過的新檔案。 因此，當容器內的處理序嘗試讀取這些檔案，反惡意程式元件會先掃描它們，才能允許檔案的存取。 實際上，在容器映像匯入或提取到伺服器時，已經掃描過這些檔案。 在未來的預覽版中，會採用新的基礎結構，讓 Windows Defender 等反惡意程式解決方案能了解這些情況，並據此採取行動以免多次掃描。

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
`failed to start: This operation returned because the timeout period expired. (0x800705B4).` (啟動失敗：因為逾時期間過期，此操作傳回。)

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


## 網路功能

### 有限的網路區間

在此版本中，我們支援每個容器一個網路區間。 這表示如果您的容器有多個網路介面卡，則您無法在每張介面卡存取相同網路連接埠 (例如 192.168.0.1: 80 和 192.168.0.2:80 屬於相同容器)。

**因應措施：**  
如果容器需公開多個端點，請使用 NAT 連接埠對應。


### 靜態 NAT 對應可能會與透過 Docker 的連接埠對應相衝突

如果要使用 Windows PowerShell 建立容器，並加入靜態 NAT 對應，在未於啟動容器之前使用 `docker-p <來源>: <目的地>` 移除靜態 NAT 對應，就會發生衝突。

以下是與連接埠 80 上靜態對應發生衝突的範例
```
PS C:\IISDemo> Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress
 172.16.0.2 -InternalPort 80 -ExternalPort 80


StaticMappingID               : 1
NatName                       : ContainerNat
Protocol                      : TCP
RemoteExternalIPAddressPrefix : 0.0.0.0/0
ExternalIPAddress             : 0.0.0.0
ExternalPort                  : 80
InternalIPAddress             : 172.16.0.2
InternalPort                  : 80
InternalRoutingDomainId       : {00000000-0000-0000-0000-000000000000}
Active                        : True



PS C:\IISDemo> docker run -it -p 80:80 microsoft/iis cmd
docker: Error response from daemon: Cannot start container 30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66: 
HCSShim::CreateComputeSystem - Win32 API call returned error r1=2147942452 err=You were not connected because a 
duplicate name exists on the network. If joining a domain, go to System in Control Panel to change the computer name
 and try again. If joining a workgroup, choose another workgroup name. 
 id=30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66 configuration= {"SystemType":"Container",
 "Name":"30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66","Owner":"docker","IsDummy":false,
 "VolumePath":"\\\\?\\Volume{4b239270-c94f-11e5-a4c6-00155d016f0a}","Devices":[{"DeviceType":"Network","Connection":
 {"NetworkName":"Virtual Switch","EnableNat":false,"Nat":{"Name":"ContainerNAT","PortBindings":[{"Protocol":"TCP",
 InternalPort":80,"ExternalPort":80}]}},"Settings":null}],"IgnoreFlushesDuringBoot":true,
 "LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66",
 "Layers":[{"ID":"4b91d267-ecbc-53fa-8392-62ac73812c7b","Path":"C:\\ProgramData\\docker\\windowsfilter\\39b8f98ccaf1ed6ae267fa3e98edcfe5e8e0d5414c306f6c6bb1740816e536fb"},
 {"ID":"ff42c322-58f2-5dbe-86a0-8104fcb55c2a",
"Path":"C:\\ProgramData\\docker\\windowsfilter\\6a182c7eba7e87f917f4806f53b2a7827d2ff0c8a22d200706cd279025f830f5"},
{"ID":"84ea5d62-64ed-574d-a0b6-2d19ec831a27",
"Path":"C:\\ProgramData\\Microsoft\\Windows\\Images\\CN=Microsoft_WindowsServerCore_10.0.10586.0"}],
"HostName":"30b17cbe8553","MappedDirectories":[],"SandboxPath":"","HvPartition":false}.
```


***降低此情況***
使用 PowerShell 移除連接埠對應可解決此問題。 如此可解決上述範例中所引發的連接埠 80 衝突。
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Windows 容器沒收到 IP

如果您要使用 DHCP VM 交換器連線到 Windows 容器，有可能容器主機收到 IP 但容器沒收到。

容器收到 169.254.***.*** APIPA IP 位址。

**因應措施：**
這是共用核心的副作用。 所有容器實際上具有相同的 MAC 位址。

啟用容器主機上的 MAC 位址詐騙。

這可以使用 PowerShell 來達成
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### 不支援 HTTPS 和 TLS

Windows Server 容器和 Hyper-V 容器都不支援 HTTPS 或 TLS。 我們正努力在未來達成此目標。

--------------------------


## 應用程式相容性

因為有許多關於在 Windows 容器中哪些應用程式可以運作哪些不可以的問題，我們決定將應用程式相容性的資訊分割成一篇[獨立文章](../reference/app_compat.md)。

在這裡也可以找到最常見的問題。

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

看到的行為：
在容器內：
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

### Docker 用戶端不安全

在此發行前版本，Docker 通訊是公開的，只要您知道哪裡可以找到。

### 並非所有的 Docker 命令都能運作

* Docker 在 Hyper-V 容器中執行失敗。
* 目前尚未支援與 DockerHub 相關的命令。

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

在 TP4 中無法透過 RDP 工作階段管理 Windows 容器或與之互動。

--------------------------


## PowerShell 管理

### 並非所有 *-PSSession 皆有 containerid 引數

這是正確的。 我們計劃在未來完整支援 CimSession。

### 無法以 "exit" 來結束 Nano Server 容器主機中的容器

如果您嘗試結束 Nano Server 容器主機中的容器，使用 "exit" 會讓您中斷與 Nano Server 容器主機的連線，但不會結束容器。

**因應措施：**
請改用 Exit-PSSession 來結束容器。

歡迎您在[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中提出功能要求。


--------------------------



## 使用者和網域

### 本機使用者

可以建立本機使用者帳戶，並用於執行容器中的 Windows 服務和應用程式。


### 網域成員資格

容器無法加入 Active Directory 網域，而且無法以網域使用者、服務帳戶或電腦帳戶的身分執行服務或應用程式。

容器的設計為可快速啟動至已知的一致狀態，且大致不受環境限制。 加入網域並套用網域的群組原則設定會增加啟動容器所需的時間、持續變更其運作方式，以及限制在開發人員之間及跨部署移動或共用映像的能力。

我們一直都很仔細考慮有關服務和應用程式如何使用 Active Directory 以及在容器中部署這些項目之交集的意見反應。如果您有最適合您的使用方式詳細資訊，請在[論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)中與我們分享。

我們正積極尋求解決方案來支援這些案例類型。






<!--HONumber=Feb16_HO4-->


