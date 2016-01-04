# Windows 容器快速入門 - PowerShell

Windows 容器可用來在單一電腦系統上快速部署許多隔離的應用程式。 本快速入門示範如何使用 PowerShell 來部署及管理 Windows Server 和 Hyper-V 容器。 在這個練習中，您將從頭建置在 Windows Server 和 Hyper-V 容器中執行的簡易 ’hello world’ 應用程式。 在此過程中，您將建立容器映像、使用容器的共用資料夾，以及管理容器的生命週期。 完成之後，您將對 Widows 容器的部署和管理有基本的了解。

此逐步解說將詳細說明 Windows Server 容器和 Hyper-V 容器。 這兩種容器都有其本身的基本需求。 Windows 容器文件包含快速部署容器主機的程序。 要快速開始使用 Windows 容器，這是最簡單的方式。 如果您還沒有容器主機，請參閱[容器主機部署快速入門](./container_setup.md)。

每個練習分別需要下列項目。

**Windows Server 容器：**

- 在內部部署或 Azure 中執行 Windows Server 2016 Core 的 Windows 容器主機。

**Hyper-V 容器：**

- 已啟用巢狀虛擬化的 Windows 容器主機。
- Windows Server 2016 媒體 - [下載](https://aka.ms/tp4/serveriso)。

> Microsoft Azure 不支援 Hyper-V 容器。 若要完成 Hyper-V 練習，您必須要有內部部署容器主機。

## Windows Server 容器

Windows Server 容器提供隔離、可攜式、由資源控制的作業環境，用以執行應用程式和主控程序。 Windows Server 容器可透過程序和命名空間的隔離，提供容器與主機之間的隔離，以及在主機上執行的容器之間的隔離。

### 建立容器

在 TP4 期間，在 Windows Server 2016 或 Windows Server 2016 Core 上執行的 Windows Server 容器，必須要有 Windows Server 2016 Core OS 映像。

輸入 `powershell` 以啟動 PowerShell 工作階段。

```powershell
C:\> powershell
Windows PowerShell
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\>
```

若要驗證已安裝 Windows Server Core OS 映像，請使用 `Get-ContainerImage` 命令。 您可能會看見多個 OS 映像，這是正常的。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

若要建立 Windows Server 容器，請使用 `New-Container` 命令。 下列範例會從 `WindowsServerCore` OS 映像建立名為 `TP4Demo` 的容器，並將容器連接到名為 `Virtual Switch` 的 VM 交換器。 請注意，輸出 (代表容器的物件) 會儲存在變數 `$con` 中。 此變數會在後續的命令中使用。

```powershell
PS C:\> New-Container -Name TP4Demo -ContainerImageName WindowsServerCore -SwitchName "Virtual Switch"

Name    State Uptime   ParentImageName
----    ----- ------   ---------------
TP4Demo Off   00:00:00 WindowsServerCore
```

使用 `Start-Container` 命令啟動容器。

```powershell
PS C:\> Start-Container -Name TP4Demo
```

使用 `Enter-Pssession` 命令連接到容器。 請注意，在建立容器的 PowerShell 工作階段後，PowerShell 提示會變更以反映容器名稱。

```powershell
PS C:\> Enter-PSSession -ContainerName TP4Demo -RunAsAdministrator

[TP4Demo]: PS C:\Windows\system32>
```

### 建立 IIS 映像

現在已可修改容器，並擷取這些修改來建立新的容器映像。 此範例會安裝 IIS。

若要在容器中安裝 IIS 角色，請使用 `Install-WindowsFeature` 命令。

```powershell
[TP4Demo]: PS C:\> Install-WindowsFeature web-server

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Common HTTP Features, Default Document, D...
```

完成 IIS 安裝之後，請輸入 `exit` 以結束容器。 這會使 PowerShell 工作階段回到容器主機的工作階段。

```powershell
[TP4Demo]: PS C:\> exit
PS C:\>
```

最後，使用 `Stop-Container` 命令停止容器。

```powershell
PS C:\> Stop-Container -Name TP4Demo
```

此容器的狀態現在可以擷取至新的容器映像。 若要這麼做，請使用 `New-ContainerImage` 命令。

此範例會建立名稱為 `WindowsServerCoreIIS`、發行者為 `Demo`、版本為 `1.0` 的新容器映像。

```powershell
PS C:\> New-ContainerImage -ContainerName TP4Demo -Name WindowsServerCoreIIS -Publisher Demo -Version 1.0

Name                 Publisher Version IsOSImage
----                 --------- ------- ---------
WindowsServerCoreIIS CN=Demo   1.0.0.0 False
```

### 建立 IIS 容器

建立新的容器，但這次從 `WindowsServerCoreIIS` 容器映像建立。

```powershell
PS C:\> New-Container -Name IIS -ContainerImageName WindowsServerCoreIIS -SwitchName "Virtual Switch"

Name State Uptime   ParentImageName
---- ----- ------   ---------------
IIS  Off   00:00:00 WindowsServerCoreIIS
```
啟動容器。

```powershell
PS C:\> Start-Container -Name IIS
```

### 設定網路功能

Windows 容器快速入門的預設網路設定，是讓容器連接到設定了網路位址轉譯 (NAT) 的虛擬交換器。 因此，為了連接到在容器內執行的應用程式，容器主機的連接埠必須對應到容器的連接埠。

在此練習中，會將一個網站裝載在執行於容器內的 IIS 中。 若要在連接埠 80 上存取網站，請將容器主機 IP 位址的連接埠 80 對應至容器 IP 位址的連接埠 80。

執行下列命令，以傳回容器的 IP 位址。

```powershell
PS C:\> Invoke-Command -ContainerName IIS {ipconfig}

Windows IP Configuration


Ethernet adapter vEthernet (Virtual Switch-7570F6B1-E1CA-41F1-B47D-F3CA73121654-0):

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::ed23:c1c6:310a:5c10%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

若要建立 NAT 連接埠對應，請使用 `Add-NetNatStaticMapping` 命令。 下列範例會檢查現有的連接埠對應規則，且若有某個規則不存在，就會加以建立。 請注意，`-InternalIPAddress` 必須符合容器的 IP 位址。

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```

連接埠對應建立後，您還必須為設定的連接埠設定輸入防火牆規則。 若要為連接埠 80 執行此動作，請執行下列指令碼。 請注意，如果您已為 80 以外的外部連接埠建立 NAT 規則，則必須建立相應的防火牆規則。

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

如果您在 Azure 中運作，且尚未建立網路安全性群組，您必須立即加以建立。 如需網路安全性群組的詳細資訊，請參閱這篇文章：[什麼是網路安全性群組](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-nsg/)。

### 建立應用程式

現在，您已從 IIS 映像建立容器，並設定網路功能，接著請開啟瀏覽器，並瀏覽至容器主機的 IP 位址。 您應該會看到 IIS 啟動顯示畫面。

![](media/iis1.png)

此時，IIS 執行個體驗證為執行中，因此您可以建立 ’Hello World’ 應用程式，並將其裝載在 IIS 執行個體中。 若要這樣做，請建立容器的 PowerShell 工作階段。

```powershell
PS C:\> Enter-PSSession -ContainerName IIS -RunAsAdministrator
[IIS]: PS C:\Windows\system32>
```

執行下列命令以移除 IIS 啟動顯示畫面。

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
執行下列命令，將預設 IIS 網站取代為新的靜態網站。

```powershell
[IIS]: PS C:\> "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

再次瀏覽至容器主機的 IP 位址，此時您應會看見 ‘Hello World’ 應用程式。 注意：您可能必須關閉任何現有的瀏覽器連線，或清除瀏覽器快取，才能看見更新的應用程式。

![](media/HWWINServer.png)

結束遠端容器工作階段。

```powershell
[IIS]: PS C:\> exit
PS C:\>
```

### 移除容器

容器必須先停止，才可移除。

```powershell
PS C:\> Stop-Container -Name IIS
```

容器已停止後，即可使用 `Remove-Container` 命令加以移除。

```powershell
PS C:\> Remove-Container -Name IIS -Force
```

最後，可以使用 `Remove-ContainerImage` 命令移除容器映像。

```powershell
PS C:\> Remove-ContainerImage -Name WindowsServerCoreIIS -Force
```

## Hyper-V 容器

Hyper-V 容器可提供比 Windows Server 容器更深層的隔離。 每個 Hyper-V 容器都是在高度最佳化的虛擬機器內建立的。 在 Windows Server 容器與容器主機以及所有其他在該主機上執行的 Windows Server 容器共用核心的環境中，Hyper-V 容器與其他容器是完全隔離的。 Hyper-V 容器的建立和管理方式與 Windows Server 容器完全相同。 如需 Hyper-V 容器的詳細資訊，請參閱[管理 Hyper-V 容器](../management/hyperv_container.md)。

> Microsoft Azure 不支援 Hyper-V 容器。 若要完成 Hyper-V 容器練習，您必須要有內部部署容器主機。

### 建立容器

在 TP4 期間，Hyper-V 容器必須使用 Nano Server Core OS 映像。 若要驗證已安裝 Nano Server OS 映像，請使用 `Get-ContainerImage` 命令。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

若要建立 Hyper-V 容器，請使用 `New-Container` 命令，並指定 HyperV 的執行階段。

```powershell
PS C:\> New-Container -Name HYPV -ContainerImageName NanoServer -SwitchName "Virtual Switch" -RuntimeType HyperV

Name State Uptime   ParentImageName
---- ----- ------   ---------------
HYPV Off   00:00:00 NanoServer
```

建立容器後，**請不要啟動它**。

### 建立共用資料夾

共用資料夾會將容器中的目錄公開給容器。 共用資料夾建立後，放置在共用資料夾中的任何檔案都可在容器中使用。 在此範例中，共用資料夾會用來將 Nano Server IIS 封裝複製到容器中。 接著，這些封裝會用來安裝 IIS。 如需共用資料夾的詳細資訊，請參閱[管理容器資料](../management/manage_data.md)。

在容器主機上建立名為 `c:\share\en-us` 的目錄。

```powershell
S C:\> New-Item -Type Directory c:\share\en-us

    Directory: C:\share

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

使用 `Add-ContainerSharedFolder` 命令，在新的容器上建立新的共用資料夾。

> 在建立共用資料夾時，容器必須處於停止狀態。

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName HYPV -SourcePath c:\share -DestinationPath c:\iisinstall

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
HYPV          c:\share   c:\iisinstall   ReadWrite
```

共用資料夾建立後，請啟動容器。

```powershell
PS C:\> Start-Container -Name HYPV
```
使用 `Enter-PSSession` 命令，建立容器的 PowerShell 遠端工作階段。

```powershell
PS C:\> Enter-PSSession -ContainerName HYPV -RunAsAdministrator
[HYPV]: PS C:\windows\system32\config\systemprofile\Documents>cd /
```
在遠端工作階段中，請留意共用資料夾 `c:\iisinstall\en-us` 已建立，不過是空的。

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

### 建立 IIS 映像

由於容器執行 Nano Server OS 映像，因此必須要有 Nano Server IIS 封裝才能安裝 IIS。 這些項目可在 Windows Sever 2016 TP4 安裝媒體中的 `NanoServer\Packages` 目錄下找到。

將 `Microsoft-NanoServer-IIS-Package.cab` 從 `NanoServer\Packages` 複製到容器主機上的 `c:\share`。

將 `NanoServer\Packages\en-us\Microsoft-NanoServer-IIS-Package.cab` 複製到容器主機上的 `c:\share\en-us`。

在 c:\share 資料夾中建立名為 unattend.xml 的檔案，並將此文字複製到 unattend.xml 檔案中。

```powershell
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <servicing>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" />
            <source location="c:\iisinstall\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="en-US" />
            <source location="c:\iisinstall\en-us\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
    </servicing>
</unattend>
```

完成之後，容器主機上的 `c:\share` 目錄應設定如下。

```
c:\share
|-- en-us
|    |-- Microsoft-NanoServer-IIS-Package.cab
|
|-- Microsoft-NanoServer-IIS-Package.cab
|-- unattend.xml
```

回到容器上的遠端工作階段，請留意到，IIS 封裝和 unattended.xml 檔案現在會顯示在 c:\iisinstall 目錄中。

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:32 PM                en-us
-a----       10/29/2015  11:51 PM        1922047 Microsoft-NanoServer-IIS-Package.cab
-a----       11/18/2015   5:31 PM            789 unattend.xml
```

執行下列命令以安裝 IIS。

```powershell
[HYPV]: PS C:\> dism /online /apply-unattend:c:\iisinstall\unattend.xml

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0


[                           1.0%                           ]

[=====                      10.1%                          ]

[=====                      10.3%                          ]

[===============            26.2%                          ]
```

IIS 安裝完成後，請使用下列命令手動啟動 IIS。

```powershell
[HYPV]: PS C:\> Net start w3svc
The World Wide Web Publishing Service service is starting.
The World Wide Web Publishing Service service was started successfully.
```

結束容器工作階段。

```powershell
[HYPV]: PS C:\> exit
```

停止容器。

```powershell
PS C:\> Stop-Container -Name HYPV
```

此容器的狀態現在可以擷取至新的容器映像。

此範例會建立名稱為 `NanoServerIIS`、發行者為 `Demo`、版本為 `1.0` 的新容器映像。

```powershell
PS C:\> New-ContainerImage -ContainerName HYPV -Name NanoServerIIS -Publisher Demo -Version 1.0

Name          Publisher Version IsOSImage
----          --------- ------- ---------
NanoServerIIS CN=Demo   1.0.0.0 False
```

### 建立 IIS 容器

使用 `New-Container` 命令，從 IIS 映像建立新的 Hyper-V 容器。

```powershell
PS C:\> New-Container -Name IISApp -ContainerImageName NanoServerIIS -SwitchName "Virtual Switch" -RuntimeType HyperV

Name   State Uptime   ParentImageName
----   ----- ------   ---------------
IISApp Off   00:00:00 NanoServerIIS
```

啟動容器。

```powershell
PS C:\> Start-Container -Name IISApp
```

### 設定網路功能

Windows 容器快速入門的預設網路設定，是讓容器連接到設定了網路位址轉譯 (NAT) 的虛擬交換器。 因此，為了連接到在容器內執行的應用程式，容器主機的連接埠必須對應到容器的連接埠。

在此練習中，會將一個網站裝載在執行於容器內的 IIS 中。 若要在連接埠 80 上存取網站，請將容器主機 IP 位址的連接埠 80 對應至容器 IP 位址的連接埠 80。

執行下列命令，以傳回容器的 IP 位址。

```powershell
PS C:\> Invoke-Command -ContainerName IISApp {ipconfig}

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::c574:5a5e:d5f5:18a0%4
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

若要建立 NAT 連接埠對應，請使用 `Add-NetNatStaticMapping` 命令。 下列範例會檢查現有的連接埠對應規則，且若有某個規則不存在，就會加以建立。 請注意，`-InternalIPAddress` 必須符合容器的 IP 位址。

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```
您也需要開啟容器主機的連接埠 80。 請注意，如果您已為 80 以外的外部連接埠建立 NAT 規則，則必須建立相應的防火牆規則。

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

### 建立應用程式

現在，您已從 IIS 映像建立容器，並設定網路功能，接著請開啟瀏覽器，並瀏覽至容器主機的 IP 位址，您應會看見 IIS 啟動顯示畫面。

![](media/iis1.png)

此時，IIS 執行個體驗證為執行中，因此您可以建立 ’Hello World’ 應用程式，並將其裝載在 IIS 執行個體中。 若要這樣做，請建立容器的 PowerShell 工作階段。

```powershell
PS C:\> Enter-PSSession -ContainerName IISApp -RunAsAdministrator
[IISApp]: PS C:\windows\system32\config\systemprofile\Documents>
```

執行下列命令以移除 IIS 啟動顯示畫面。

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
執行下列命令，將預設 IIS 網站取代為新的靜態網站。

```powershell
[IISApp]: PS C:\> "Hello World From a Hyper-V Container" > C:\inetpub\wwwroot\index.html
```

再次瀏覽至容器主機的 IP 位址，此時您應會看見 ‘Hello World’ 應用程式。 注意：您可能必須關閉任何現有的瀏覽器連線，或清除瀏覽器快取，才能看見更新的應用程式。

![](media/HWWINServer.png)

結束遠端容器工作階段。

```powershell
exit
```




