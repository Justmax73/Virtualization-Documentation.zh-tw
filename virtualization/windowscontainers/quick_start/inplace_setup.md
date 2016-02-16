# 將 Windows 容器主機部署至現有的虛擬或實體系統

本文將引導您逐步使用 PowerShell 指令碼在現有的實體或虛擬系統上部署及設定 Windows 容器角色。

若要以指令碼逐步部署設定為 Windows 容器主機的新 Hyper-V 虛擬機器，請參閱[新的 Hyper-V Windows 容器主機](./container_setup.md)。

**安裝容器 OS 映像之前請先閱讀：**需遵守 Microsoft Windows Server 發行前版本軟體的授權條款 (「授權條款」)，才能使用 Microsoft Windows 容器 OS 映像補充軟體 (「補充軟體」)。 下載並使用補充軟體後，即表示您同意「授權條款」，如果您不接受授權條款，則無法使用。 Windows 伺服器發行前軟體和補充軟體皆由 Microsoft Corporation 授權。

必須符合下列條件，才能完成此快速入門中的 Windows Server 容器和 Hyper-V 容器練習。

* 執行 Windows Server Technical Preview 4 或更新版本的系統。
* 10 GB 的可用儲存體供容器主機映像、OS 基本映像和安裝指令碼使用。
* 系統的系統管理員權限。

## 為容器設定現有的虛擬機器或裸機主機

Windows 容器需要容器 OS 基本映像。 我們為您整合出一個可下載並安裝此項目的指令碼。 請遵循下列步驟，將您的系統設定為 Windows 容器主機。 如需詳細資訊，請參閱 [Windows Server 2016 Technical Preview](https://tnstage.redmond.corp.microsoft.com/en-US/library/dn765471.aspx#BKMK_nested) 上的「Hyper-V 新功能」。

以系統管理員身分啟動 PowerShell 工作階段。 這可藉由從命令列執行下列命令來完成。

``` powershell
PS C:\> powershell.exe
```

請確定視窗的標題是「系統管理員：Windows PowerShell」。 如果不是顯示為「系統管理員」，請執行下列命令，以系統管理員的權限執行：

``` powershell
PS C:\> start-process powershell -Verb runas
```

使用下列命令下載安裝指令碼。 您也可以手動從這個位置下載指令碼 - [設定指令碼](https://aka.ms/tp4/Install-ContainerHost)。

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/Install-ContainerHost -OutFile C:\Install-ContainerHost.ps1
```

 下載完成後，請執行指令碼。
``` PowerShell
PS C:\> powershell.exe -NoProfile C:\Install-ContainerHost.ps1 -HyperV
```

接著，指令碼會開始下載及設定 Windows 容器元件。 執行此程序可能需要一段時間，因為下載內容較大。 在程序執行期間，電腦可能會重新開機。 完成之後，您的機器即已設定並就緒，可讓您透過 PowerShell 和 Docker 來建立及管理 Windows 容器和 Windows 容器映像。

 這些項目皆完成後，您的系統即應可供 Windows 容器使用。

## 後續步驟：開始使用容器

現在您已有執行 Windows 容器功能的 Windows Server 2016 系統，接下來請移至下列指南，開始使用 Windows Server 容器和 Hyper-V 容器。

[快速入門：Windows 容器和 Docker](./manage_docker.md)

[快速入門：Windows 容器和 PowerShell](./manage_powershell.md)




<!--HONumber=Feb16_HO2-->
