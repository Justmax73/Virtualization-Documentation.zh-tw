# 容器主機部署 - Windows Server

**這是初版內容，後續可能會變更。**

部署 Windows 容器主機有不同的步驟，視作業系統和主機系統類型 (實體或虛擬) 而定。 本文件中的步驟用於將 Windows 容器主機部署至實體或虛擬系統上的 Windows Server 2016 或 Windows Server Core 2016。 若要將 Windows 容器主機安裝到 Nano Server，請參閱[容器主機部署 - Nano Server](./deployment_nano.md)。

如需系統需求的詳細資訊，請參閱 [Windows 容器主機系統需求](./system_requirements.md)。

PowerShell 指令碼也可用來自動化 Windows 容器主機的部署。
- [在新的 Hyper-V 虛擬機器中部署容器主機](../quick_start/container_setup.md)。
- [將容器主機部署至現有的系統](../quick_start/inplace_setup.md)。

# Windows Server 主機

下表所列的步驟可用於在 Windows Server 2016 和 Windows Server 2016 Core 上部署容器主機。 內含 Windows Server 和 Hyper-V 容器所需的組態。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>部署動作</strong></td>
<td width="70%"><strong>詳細資料</strong></td>
</tr>
<tr>
<td>[安裝容器功能](#role)</td>
<td>容器功能可讓您使用 Windows Server 和 Hyper-V 容器。</td>
</tr>
<tr>
<td>[建立虛擬交換器](#vswitch)</td>
<td>容器會連接到虛擬交換器，以進行網路連線。</td>
</tr>
<tr>
<td>[設定 NAT](#nat)</td>
<td>如果虛擬交換器設定了網路位址轉譯，NAT 本身將需要設定。</td>
</tr>
<tr>
<td>[安裝容器 OS 映像](#img)</td>
<td>OS 映像可提供容器部署的基礎。</td>
</tr>
<tr>
<td>[安裝 Docker](#docker)</td>
<td>選用，但若要使用 Docker 來建立和管理 Windows 容器，則必須執行。</td>
</tr>
</table>

將會使用 Hyper-V 容器時，才需要採取這些步驟。 注意，如果容器主機本身就是 Hyper-V 虛擬機器，才必須執行標示 * 的步驟。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>部署動作</strong></td>
<td width="70%"><strong>詳細資料</strong></td>
</tr>
<tr>
<td>[啟用 Hyper-V 角色](#hypv) </td>
<td>只有將會使用 Hyper-V 容器時，才需要 Hyper-V。</td>
</tr>
<tr>
<td>[啟用巢狀虛擬化 *](#nest)</td>
<td>如果容器主機本身就是 Hyper-V 虛擬機器，則必須啟用巢狀虛擬化。</td>
</tr>
<tr>
<td>[設定虛擬處理器 *](#proc)</td>
<td>如果容器主機本身就是 Hyper-V 虛擬機器，則至少必須設定兩個虛擬處理器。</td>
</tr>
<tr>
<td>[停用動態記憶體 *](#dyn)</td>
<td>如果容器主機本身就是 Hyper-V 虛擬機器，則必須停用動態記憶體。</td>
</tr>
<tr>
<td>[設定 MAC 位址詐騙 *](#mac)</td>
<td>如果容器主機是虛擬化的，則必須啟用 MAC 詐騙。</td>
</tr>
</table>

## 部署步驟

### <a name=role></a>安裝容器功能

容器功能可以使用 Windows 伺服器管理員或 PowerShell 安裝在 Windows Server 2016 或 Windows Server 2016 Core 上。

若要使用 PowerShell 安裝角色，請在提高權限的 PowerShell 工作階段中執行下列命令。

```powershell
PS C:\> Install-WindowsFeature containers
```
容器角色安裝完成後，必須重新啟動系統。

```powershell
PS C:\> shutdown /r 
```
系統重新開機之後，請使用 `Get-ContainerHost` 命令確認容器角色已成功安裝：

```powershell
PS C:\> Get-ContainerHost

Name            ContainerImageRepositoryLocation
----            --------------------------------
WIN-LJGU7HD7TEP C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store
```

### <a name=vswitch></a>建立虛擬交換器

每個容器都必須連接到虛擬交換器，才能透過網路進行通訊。 虛擬交換器是以 `New-VMSwitch` 命令來建立。 容器支援 `External` 或 `NAT` 類型的虛擬交換器。 如需 Windows 容器網路功能的詳細資訊，請參閱[容器網路功能](../management/container_networking.md)。

此範例會建立名稱為 “Virtual Switch”、類型為 NAT、Nat 子網路為 172.16.0.0/12 虛擬交換器。

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress 172.16.0.0/12
```

### <a name=nat></a>設定 NAT

除了建立虛擬交換器以外，如果交換器類型為 NAT，則還必須建立 NAT 物件。 此步驟可使用 `New-NetNat` 命令來完成。 此範例會以名稱 `ContainerNat` 以及與指派給容器交換器的 NAT 子網路相符的位址首碼來建立 NAT 物件。

```powershell
PS C:\> New-NetNat -Name ContainerNat -InternalIPInterfaceAddressPrefix "172.16.0.0/12"

Name                             : ContainerNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 172.16.0.0/12
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

### <a name=img></a>安裝 OS 映像

OS 映像可做為任何 Windows Server 或 Hyper-V 容器的基底。 此映像可用來部署容器，接著容器可以修改，並擷取到新的容器映像中。 目前已建立以 Windows Server Core 和 Nano Server 做為基礎作業系統的 OS 映像。

您可以使用 ContainerProvider PowerShell 模組尋找並安裝容器 OS 映像。 此模組必須先安裝，才可使用。 下列命令可用來安裝此模組。

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

使用 `Find-ContainerImage` 從 PowerShell OneGet 封裝管理員傳回映像清單：
```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```
若要下載並安裝 Nano Server 基本 OS 映像，請執行下列命令。

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

同樣地，此命令會下載並安裝 Windows Server Core 基本 OS 映像。

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

**問題：**Save-ContainerImage 和 Install-ContainerImage Cmdlet 無法在 PowerShell 遠端工作階段中使用 WindowsServerCore 容器映像。<br />**因應措施：**使用遠端桌面登入機器，並直接使用 Save-ContainerImage Cmdlet。

使用 `Get-ContainerImage` 命令確認已安裝映像。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```
如需容器映像管理的詳細資訊，請參閱 [Windows 容器映像](../management/manage_images.md)。


### <a name=docker></a>安裝 Docker

Docker 精靈和命令列介面並未隨附於 Windows，且不會隨 Windows 容器功能一起安裝。 使用 Windows 容器時不需要 Docker。 如果您想要安裝 Docker，請依照 [Docker 和 Windows](./docker_windows.md) 一文中的指示操作。


## Hyper-V 容器主機

### <a name=hypv></a>啟用 Hyper-V 角色

如果將部署 Hyper-V 容器，需要在容器主機上啟用 Hyper-V 角色。 Hyper-V 角色可以使用 `Install-WindowsFeature` 命令安裝在 Windows Server 2016 或 Windows Server 2016 Core 上。 如果容器主機本身就是 Hyper-V 虛擬機器，則必須先啟用巢狀虛擬化。 若要執行這項操作，請參閱[設定巢狀虛擬化](#nest)。

```powershell
PS C:\> Install-WindowsFeature hyper-v
```

### <a name=nest></a>設定巢狀虛擬化

如果容器主機本身將會在 Hyper-V 虛擬機器上執行，而且也會主控 Hyper-V 容器，則必須啟用巢狀虛擬化。 這可以使用下列 PowerShell 命令來完成。

**注意** - 執行此命令時，必須要關閉虛擬機器。

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>設定虛擬處理器

如果容器主機本身將會在 Hyper-V 虛擬機器上執行，而且也會主控 Hyper-V 容器，則虛擬機器至少要有兩個處理器。 這可以透過虛擬機器的設定，或使用下列命令來設定。

**注意** - 執行此命令時，必須要關閉虛擬機器。

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=dyn></a>停用動態記憶體

如果容器主機本身就是 Hyper-V 虛擬機器，則必須在容器主機虛擬機器上停用動態記憶體。 這可以透過虛擬機器的設定，或使用下列命令來設定。

**注意** - 執行此命令時，必須要關閉虛擬機器。

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>設定 MAC 位址詐騙

最後，如果容器主機在 Hyper-V 虛擬機器內執行，則必須啟用 MAC 詐騙。 這可讓每個容器接收 IP 位址。 若要啟用 MAC 位址詐騙，請在 Hyper-V 主機上執行下列命令。 VMName 屬性會是容器主機的名稱。

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```





<!--HONumber=Feb16_HO1-->
