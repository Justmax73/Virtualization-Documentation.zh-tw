



# Windows Server 容器管理

**這是初版內容，後續可能會變更。**

容器的生命週期包含啟動、停止和移除容器等動作。 在執行這些動作時，您可能還必須擷取容器映像清單、管理容器網路功能，以及限制容器資源。 本文件將詳細說明使用 PowerShell 的基本容器管理工作。

如需使用 Docker 管理 Windows 容器的相關文件，請參閱 Docker 文件[使用容器](https://docs.docker.com/userguide/usingdocker/)。

## PowerShell

### 建立容器

在建立新容器時，您需要將做為容器基底之容器映像的名稱。 可使用 `Get-ContainerImage` 命令來找到此名稱。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version         IsOSImage
----              ---------    -------         ---------
NanoServer        CN=Microsoft 10.0.10584.1000 True
WindowsServerCore CN=Microsoft 10.0.10584.1000 True
```

使用 `New-Container` 命令建立新容器。 使用 `-ContainerComputerName` 參數也可為容器提供 NetBIOS 名稱。

```powershell
PS C:\> New-Container -ContainerImageName WindowsServerCore -Name demo -ContainerComputerName demo

Name State Uptime   ParentImageName
---- ----- ------   ---------------
demo  Off   00:00:00 WindowsServerCore
```

容器建立後，請將網路介面卡新增至容器。

```powershell
PS C:\> Add-ContainerNetworkAdapter -ContainerName demo
```

若要將容器網路介面卡連接到虛擬交換器，必須要有交換器名稱。 使用 `Get-VMSwitch` 傳回虛擬交換器的清單。

```powershell
PS C:\> Get-VMSwitch

Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
DHCP External   Microsoft Hyper-V Network Adapter
NAT  NAT
```

使用 `Connect-ContainerNetowkrAdapter`，將網路介面卡連接到虛擬交換器。 **注意** – 建立容器時使用 -SwitchName 參數也可完成此動作。

```powershell
PS C:\> Connect-ContainerNetworkAdapter -ContainerName demo -SwitchName NAT
```

### 啟動容器

在啟動容器時，必須列舉代表容器的 PowerShell 物件。 將 `Get-Container` 的輸出放入 PowerShell 變數中，即可完成此動作。

```powershell
PS C:\> $container = Get-Container -Name demo
```

接著，這項資料將可與 `Start-Container` 命令搭配使用，以啟動容器。

```powershell
PS C:\> Start-Container $container
```

下列指令碼會啟動主機上的所有容器。

```powershell
PS C:\> Get-Container | Start-Container
```

### 與容器連接

PowerShell Direct 可用來連接到容器。 如果您需要手動執行安裝軟體、啟動程序或對容器進行疑難排解之類的工作，這可能會有幫助。 由於使用了 PowerShell Direct，因此無論網路設定為何，都可以建立容器的 PowerShell 工作階段。 如需 PowerShell Direct 的詳細資訊，請參閱 [PowerShell Direct 部落格](http://blogs.technet.com/b/virtualization/archive/2015/05/14/powershell-direct-running-powershell-inside-a-virtual-machine-from-the-hyper-v-host.aspx)

若要建立容器的互動式工作階段，請使用 `Enter-PSSession` 命令。

 ```powershell
PS C:\> Enter-PSSession -ContainerName demo -RunAsAdministrator
 ```

請注意，在遠端 PowerShell 工作階段建立後，殼層提示即會變更，以反映容器名稱。

```powershell
[demo]: PS C:\>
```

此外也可直接對容器執行命令，而無需建立持續性 PowerShell 工作階段。 若要這麼做，請使用 `Invoke-Command`。

下列範例會在容器中建立名為 ‘Application’ 的資料夾。

```powershell

PS C:\> Invoke-Command -ContainerName demo -ScriptBlock {New-Item -ItemType Directory -Path c:\application }

Directory: C:\
Mode                LastWriteTime         Length Name                                                 PSComputerName
----                -------------         ------ ----                                                 --------------
d-----       10/28/2015   3:31 PM                application                                          demo
```

### 停止容器

若要停用容器，將需要一個代表該容器的 PowerShell 物件。 將 `Get-Container` 的輸出放入 PowerShell 變數中，即可完成此動作。

```powershell
PS C:\> $container = Get-Container -Name demo
```

接著，此資料將可與 `Stop-Container` 命令搭配使用，以停止容器。

```powershell
PS C:\> Stop-Container $container
```

下列命令會停止主機上的所有容器。

```powershell
PS C:\> Get-Container | Stop-Container
```

### 移除容器

當容器已不再需要時，即可被移除。 若要移除容器，容器必須處於停止狀態，且必須建立代表容器的 PowerShell 物件。

```powershell
PS C:\> $container = Get-Container -Name demo
```

若要移除容器，請使用 `Remove-Container` 命令。

```powershell
PS C:\> Remove-Container $container -Force
```

下列命令會移除主機上的所有容器。

```powershell
PS C:\> Get-Container | Remove-Container -Force
```

## Docker

### 建立容器

使用 `docker run`，以使用 Docker 建立容器。

```powershell
PS C:\> docker run -p 80:80 windowsservercoreiis
```

如需 Docker run 命令的詳細資訊，請參閱 [Docker run reference}( https://docs.docker.com/engine/reference/run/)。

### 停止容器

使用 `docker stop` 命令，以使用 Docker 停止容器。

```powershell
PS C:\> docker stop tender_panini

tender_panini
```

此範例會使用 Docker 停止所有執行中的容器。

```powershell
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### 移除容器

若要使用 Docker 移除容器，請使用 `docker rm` 命令。

```powershell
PS C:\> docker rm prickly_pike

prickly_pike
```

若要使用 Docker 移除所有容器。

```powershell
PS C:\> docker rm $(docker ps -a -q)

dc3e282c064d
2230b0433370
```

如需 Docker rm 命令的詳細資訊，請參閱 [Docker rm 參考](https://docs.docker.com/engine/reference/commandline/rm/)。






<!--HONumber=Feb16_HO4-->


