---
author: scooley
redirect_url: ../quick_start/manage_docker
---


# 使用 PowerShell 和 Docker 管理 Windows 容器的比較

有許多方式可管理 Windows 容器，包括使用現成的 Windows 工具 (在此預覽中使用 PowerShell)，以及開放原始碼管理工具 (例如 Docker)。  
要點指南分別如下：
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">以 Docker 管理 Windows 容器</g><g id="1CapsExtId3" ctype="x-title"></g></g>
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">以 PowerShell 管理 Windows 容器</g><g id="1CapsExtId3" ctype="x-title"></g></g>

此頁面是更深度的 Docker 工具和 PowerShell 管理工具的比較參考。

## 用於容器與 Hyper-V VM 的 PowerShell

您可以使用 PowerShell Cmdlet 來建立、執行 Windows 容器以及與容器互動。 您需要的所有東西都是內建現成的。

如果您用過 Hyper-V PowerShell，對於 Cmdlet 的設計應該很熟悉。 其許多工作流程類似於使用 Hyper-V 模組管理虛擬機器。 不使用 <g id="2" ctype="x-code">New-VM</g>、<g id="4" ctype="x-code">Get-VM</g>、<g id="6" ctype="x-code">Start-VM</g>、<g id="8" ctype="x-code">Stop-VM</g>，而是用 <g id="10" ctype="x-code">New-Container</g>、<g id="12" ctype="x-code">Get-Container</g>、<g id="14" ctype="x-code">Start-Container</g>、<g id="16" ctype="x-code">Stop-Container</g>。 有不少容器專用的 Cmdlet 和參數，但 Windows 容器的一般生命週期和管理看起來和 Hyper-V 虛擬機器的週期和管理相似。

## PowerShell 管理與 Docker 的比較？

PowerShell 的容器系列 Cmdlet 展現的 API 和 Docker 的大不相同，一般而言，Cmdlet 在作業上較精細。 某些 Docker 命令在 PowerShell 中有很直接的同等 Cmdlet：

| Docker 命令| PowerShell Cmdlet|
|----|----|
| <g id="1" ctype="x-code">docker ps -a</g>| <g id="1" ctype="x-code">Get-Container</g>|
| <g id="1" ctype="x-code">docker images</g>| <g id="1" ctype="x-code">Get-ContainerImage</g>|
| <g id="1" ctype="x-code">docker rm</g>| <g id="1" ctype="x-code">Remove-Container</g>|
| <g id="1" ctype="x-code">docker rmi</g>| <g id="1" ctype="x-code">Remove-ContainerImage</g>|
| <g id="1" ctype="x-code">docker create</g>| <g id="1" ctype="x-code">New-Container</g>|
| <g id="1" ctype="x-code">docker commit <container ID></g>| <g id="1" ctype="x-code">New-ContainerImage -Container &lt;container&gt;</g>|
| <g id="1" ctype="x-code">docker load &lt;tarball&gt;</g>| <g id="1" ctype="x-code">Import-ContainerImage <AppX package></g>|
| <g id="1" ctype="x-code">docker save</g>| <g id="1" ctype="x-code">Export-ContainerImage</g>|
| <g id="1" ctype="x-code">docker start</g>| <g id="1" ctype="x-code">Start-Container</g>|
| <g id="1" ctype="x-code">docker stop</g>| <g id="1" ctype="x-code">Stop-Container</g>|

PowerShell Cmdlet 並不是完美的同等對應，而且有很多的命令我們無法提供 PowerShell 替代品 * (尤其是 <g id="2" ctype="x-code">docker build</g> 和 <g id="4" ctype="x-code">docker cp</g>)。 不過最令您訝異的可能是 <g id="2" ctype="x-code">docker run</g> 沒有單一行的 PowerShell 替代品。

\ * 可能有所變更。

### 但是我需要 docker run！發生什麼事？

我們正在做一些事，希望為已經熟悉 PowerShell 的使用者提供更熟悉的互動模型。 當然，如果您習慣 Docker 的運作方式，將會有一些適應的過程。

1.  PowerShell 模型中容器的生命週期有些許不同。 在容器 PowerShell 模組中，我們展現 <g id="2" ctype="x-code">New-Container</g> (可建立已停止的新容器) 及 <g id="4" ctype="x-code">Start-Container</g> 更精細的作業。

  在建立與啟動容器之間，您也可以設定容器的設定。針對 TP3，我們唯一計劃提供的其他設定是為容器設定網路連線的能力。 使用 (Add/Remove/Connect/Disconnect/Get/Set)-ContainerNetworkAdapter Cmdlet。

2.  目前您無法在啟動容器時傳遞要在容器內執行的命令。不過，您還是可以使用 <g id="2" ctype="x-code">Enter-PSSession -ContainerId <ID of a running containe></g> 取得執行中容器的互動式 PowerShell 工作階段，而且可以使用 <g id="4" ctype="x-code">Invoke-Command -ContainerId <container id> -ScriptBlock { code to run inside the container }</g> 或 <g id="6" ctype="x-code">Invoke-Command -ContainerId <container id> -FilePath <path to script></g> 在執行中的容器內執行命令。  
這兩個命令都可使用選擇性的 <g id="2" ctype="x-code">-RunAsAdministrator</g> 旗標以進行高權限的動作。


## 注意事項和已知問題

1.  現階段，容器系列 Cmdlet 對於透過 Docker 建立的容器或映像毫無認知，Docker 完全不認識透過 PowerShell 建立的容器和映像。 如果您在 Docker 中建立，就用 Docker 管理，如果您透過 PowerShell 建立，就透過 PowerShell 管理。

2.  我們想做相當多的工作來改善使用者體驗 -- 更好的錯誤訊息、更好的進度報告、無效事件字串等等。 如果您遇到您希望能獲得更多或更好資訊的情況，歡迎您將建議傳送到論壇。

## 流程快速示範

以下是一些常見工作流程的逐步解說。

範例假定您已安裝名為 "ServerDatacenterCore" 的作業系統容器映像，且已建立名為 "Virtual Switch" 的虛擬交換器 (使用 New-VMSwitch)。

``` PowerShell
### 1. Enumerating images
# At this point, you can enumerate the images on the system:
Get-ContainerImage

# Get-ContainerImage also accepts filters.
# For example, this will return all container images whose Name starts with S (case-insensitive):
Get-ContainerImage -Name S*

# You can save the results of this to a variable.
# (If you're not familiar with PowerShell, the "$" denotes a variable.)
$baseImage = Get-ContainerImage -Name ServerDatacenterCore
$baseImage

### 2. Creating and enumerating containers
# Now, we can create a new container using this image:
New-Container -Name "MyContainer" -ContainerImage $baseImage -SwitchName "Virtual Switch"

# Now we can enumerate all containers.
Get-Container

# Similarly, we can save this container to a variable:
$container1 = Get-Container -Name "MyContainer"

### 3. Starting containers, interacting with running containers, and stopping containers
# Now let's go ahead and start the container.
Start-Container -Name "MyContainer"

# (We could've also started this container using "Start-Container -Container $container1".)

# With the container now running, let's go ahead and enter an interactive PowerShell session:
Enter-PSSession -ContainerId $container1.Id

# This should eventually bring up a PowerShell prompt from inside the container.
# You can try all the things that you did in the interactive cmd prompt given by "docker run -it".
# For now, just to prove we've been here, we can create a new file:
cd \
mkdir Test
cd Test
echo "hello world" > hello.txt
exit

# Now we should be back in the outside world. Even though we've exited the PowerShell session,
# the container itself is still running, as you can see by printing out the container again:
$container1

# Before we can commit this container to a new image, we need to stop the container.
# Let's do that now.
Stop-Container -Container $container1

### 4. Creating a new container image
# And now let's commit it to a new image.
$image1 = New-ContainerImage -Container $container1 -Publisher Test -Name Image1 -Version 1.0

# Enumerate all the images again, for sanity's sake:
Get-ContainerImage

# Rinse and repeat! Make another container based on the new image.
$container2 = New-Container -Name "MySecondContainer" -ContainerImage $image1 -SwitchName "Virtual Switch"

# (If you like, you can start the second container and verify that the new file
# "\Test\hello.txt" is there as expected.)

### 5. Removing a container
# The first container we created is now stopped. Let's get rid of it:
Remove-Container -Container $container1

# And confirm that it's gone:
Get-Container

### 6. Exporting, removing, and importing images
# For images that aren't the base OS image, we can export them into an .appx package file.
Export-ContainerImage -Image $image1 -Path "C:\exports"
# This should create a .appx file in the C:\exports folder.
# If you've given your image the same publisher, name, and version we used earlier,
# you'd expect the resulting .appx to be named "CN=Test_Image1_1.0.0.0.appx".

# Before we can try importing the image again, we need to remove the image.
# (If you have any running containers that depend on this image, you'll want to stop them first.)
Remove-ContainerImage -Image $image1

# Now let's import the image again:
Import-ContainerImage -Path C:\exports\CN=Test_Image1_1.0.0.0.appx

# We'd previously created a container dependent on this image. You should be able to start it:
Start-Container -Container $container2 
```

### 建立您自己的範例

您可以使用 <g id="2" ctype="x-code">Get-Command -Module Containers</g> 查看所有容器 Cmdlet。 其他幾個這裡沒有描述的 Cmdlet，留待您自己去了解。    
<g id="1" ctype="x-strong">附註</g> 這不會傳回 <g id="3" ctype="x-code">Enter-PSSession</g> 和 <g id="5" ctype="x-code">Invoke-Command</g>，這兩個 Cmdlet 屬於核心 PowerShell。

您也可以使用 <g id="2" ctype="x-code">Get-Help [cmdlet name]</g> 或等同的 <g id="4" ctype="x-code">[cmdlet name] -?</g> 取得任何 Cmdlet 的相關說明。 目前，「說明」輸出是自動產生，只是告訴您命令的語法，隨著 Cmdlet 的設計越來越接近完成，我們將更進一步加入文件。

PowerShell ISE 是一個探索語法的好方法，在您沒用過 PowerShell 之前可能沒有看過。 如果您執行的 SKU 可以使用它，請嘗試啟動 ISE 中，做法是開啟 [命令] 窗格，並選擇 [容器] 模組，這將會顯示 Cmdlet 及其參數的圖形化表示。

PS：只是為了證明可以這麼做，以下的 PowerShell 函式由我們已介紹過的一些 Cmdlet 組成，用在假造的 <g id="2" ctype="x-code">docker run</g>。 (先說清楚，這是概念證明，不是真的開發。)

``` PowerShell
function Run-Container ([string]$ContainerImageName, [string]$Name="fancy_name", [switch]$Remove, [switch]$Interactive, [scriptblock]$Command) {
    $image = Get-ContainerImage -Name $ContainerImageName
    $container = New-Container -Name $Name -ContainerImage $image
    Start-Container $container

    if ($Interactive) {
         Start-Process powershell ("-NoExit", "-c", "Enter-PSSession -ContainerId $($container.Id)") -Wait
    } else {
        Invoke-Command -ContainerId $container.Id -ScriptBlock $Command
    }

    Stop-Container $container

    if ($Remove) {
        Remove-Container $container -Force
    }
} 
```

## Docker

可以用 Docker 命令管理 Windows 容器。 雖然 Windows 容器應該是與其 Linux 同等項目比較，而且透過 Docker 有相同的管理體驗，但有些 Docker 命令用在 Windows 容器就是不合理。 有些則尚未經過測試 (我們快進行到這一步了)。

為了不與 Docker 的 API 文件重複，<g id="2" ctype="x-html"></g>這裡<g id="4" ctype="x-html"></g>是其管理 API 的連結。 他們的逐步解說很棒。

我們正在「進行的工作」文件中追蹤在 Docker API 中有作用和沒有作用的東西。






<!--HONumber=Apr16_HO4-->


