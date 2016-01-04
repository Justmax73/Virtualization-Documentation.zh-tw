# 建立 .NET 3.5 Server Core 容器映像

本指南詳述建立一個包含 .NET 3.5 Framework 的 Windows Server Core 容器。 開始此練習之前，您需要 Windows Server 2016 .iso 檔案，或存取 Windows Server 2016 媒體。

## 準備媒體

在建立已啟用 .NET 3.5 的容器映像之前的，需要預備 .NET 3.5 封裝供容器內使用。 在此範例中，需要從 Windows Server 2016 媒體中將 `Microsoft-windows-netfx3-ondemand-package.cab` 檔案複製到容器主機。

在容器主機上建立名為 `dotnet3.5\source` 的目錄。

```powershell
New-Item -ItemType Directory c:\dotnet3.5\source
```

將 `Microsoft-windows-netfx3-ondemand-package.cab` 檔案複製到此目錄中。 在 Windows Server 2016 媒體的 sources\sxs 資料夾下可以找到這個檔案。

```powershell
$file = "d:\sources\sxs\Microsoft-windows-netfx3-ondemand-package.cab"
Copy-Item -Path $file -Destination c:\dotnet3.5\source
```

或者，如果容器主機在 Hyper-V 虛擬機器中執行，而且已利用快速啟動指令碼完成部署，則可以執行下列步驟。 請注意，這是在 Hyper-V 主機上執行，而不是在容器主機上。

```powershell
$vm = "<Container Host VM Name>"
$iso = "$((Get-VMHost).VirtualHardDiskPath)".TrimEnd("\") + "\WindowsServerTP4.iso"
Mount-DiskImage -ImagePath $iso
$ISOImage = Get-DiskImage -ImagePath $iso | Get-Volume
$ISODrive = "$([string]$iSOImage.DriveLetter):"
Get-VM -Name $vm | Enable-VMIntegrationService -Name "Guest Service Interface"
Copy-VMFile -Name $vm -SourcePath "$iSODrive\sources\sxs\microsoft-windows-netfx3-ondemand-package.cab" -DestinationPath "c:\dotnet3.5\source\microsoft-windows-netfx3-ondemand-package.cab" -FileSource Host -CreateFullPath
Dismount-DiskImage -ImagePath $iso
```

現在可以建立一個將包含 .NET 3.5 Framework 的容器映像。 這可以使用 PowerShell 或 Docker 來完成。 兩者的範例如下。

## 建立映像 - PowerShell

若要使用 PowerShell 建立新的映像，將會建立容器、進行所有需要的變更完成修改，然後擷取到新的映像中。

從 Windows Server Core 基本映像建立新的容器。

```powershell
New-Container -Name dotnet35 -ContainerImageName windowsservercore -SwitchName “Virtual Switch”
```

建立新容器的共用資料夾。 這可讓 .NET 3.5 cab 檔可供新的容器內存取。 請注意，執行下列命令時，必須停止容器。

```powershell
Add-ContainerShareFolder -ContainerName dotnet35 -SourcePath C:\dotnet3.5\source -DestinationPath c:\sxs
```

啟動容器，並執行下列命令來安裝 .NET 3.5。

```powershell
Start-Container dotnet35
Invoke-Command -ContainerName dotnet35 -ScriptBlock {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs} -RunAsAdministrator
```

安裝完成時，請停止容器。

```powershell
Stop-Container dotnet35
```

若要從這個容器建立映像，請在容器主機上執行下列命令。

```powershell
New-ContainerImage -ContainerName dotnet35 -Name dotnet35 -Publisher Demo -Version 1.0
```

執行 `Get-ContainerImages` 以查看新的映像。 現在可以使用此映像執行已預先安裝 .NET 3.5 Framework 的容器。

```powershell
Get-ContainerImages
```

## 建立映像 - Docker

若要使用 Docker 建立新的映像，需要根據如何建立新映像的指示來建立 dockerfile。 接著執行此 dockerfile，以產生新的容器映像。 請注意，下列命令是在容器主機 VM 上執行。

建立 dockerfile，並在 [記事本] 中開啟它。

```powershell
New-Item C:\dotnet3.5\dockerfile -Force
Notepad C:\dotnet3.5\dockerfile
```

將此文字複製到 dockerfile 中並儲存。

```powershell
FROM windowsservercore
ADD source /sxs
RUN powershell -Command "& {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs}"
```

執行 `docker build` 以取用 dockerfile 並建立新的容器映像。

```powershell
Docker build -t dotnet35 C:\dotnet3.5\
```

執行 “docker images” 以查看新的映像。 現在可以使用此映像執行已預先安裝 .NET 3.5 Framework 的容器。

```powershell
docker images
```




