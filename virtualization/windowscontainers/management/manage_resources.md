# 容器資源管理

**這是初版內容，後續可能會變更。**

Windows 容器能夠管理可使用的 CPU、磁碟 IO、網路和記憶體資源數量。 限制容器資源耗用量，可讓主機資源有效使用，並防止過度使用。 本文將詳細說明如何使用 PowerShell 和 Docker 來管理容器資源。

## 使用 PowerShell 管理資源

### Memory

建立容器時，可使用 `New-Container` 命令的 `-MaximumMemoryBytes` 參數設定容器記憶體限制。 此範例將最大記憶體設為 256 MB。

```powershell
PS C:\> New-Container –Name TestContainer –MaximumMemoryBytes 256MB -ContainerimageName WindowsServerCore
```
您也可以使用 `Set-ContainerMemory` Cmdlet 設定現有容器的記憶體限制。

```powershell
PS C:\> Set-ContainerMemory -ContainerName TestContainer -MaximumBytes 500mb
```

### 網路頻寬

在現有容器上可以設定網路頻寬限制。 若要這樣做，請使用 `Get-ContainerNetworkAdapter` 命令確定容器具有網路介面卡。 如果網路介面卡不存在，請使用 `Add-ContainerNetworkAdapter` 命令建立一個。 最後，使用 `Set-ContainerNetworkAdapter` 命令限制容器的最大輸出網路頻寬。

下列範例將最大頻寬設為 100Mbps。

```powershell
PS C:\> Set-ContainerNetworkAdapter –ContainerName TestContainer –MaximumBandwidth 100000000
```

### CPU

您可以設定 CPU 百分比上限，或設定容器的相對權數，藉以限制容器可使用的計算量。 雖然混用這兩種 CPU 管理機制是可行的，但不建議這麼做。 根據預設，所有容器都具有完整的處理器使用率 (最大值 100%)，以及相對權數 100。

以下將容器的相對權數設為 1000。 容器的預設權數為 100，因此，此容器的優先順序會是設為預設值的容器的 10 倍。 最大值為 10000。

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 –RelativeWeight 10000.
```

您也可以用 CPU 時間的百分比形式，對容器可使用的 CPU 數量設定固定限制。 根據預設，容器可以使用 100%的 CPU。 以下將容器可使用的 CPU 百分比上限設為 30%。 使用 –Maximum 旗標，會自動將 RelativeWeight 設為 100。

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -Maximum 30
```

### 儲存體 IO

您可以用頻寬 (每秒位元組) 或 8k 標準化 IOPS 的形式，限制容器可使用的 IO 數量。 這兩個參數可一起設定。 到達第一個限制時，就會發生節流。

```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumBandwidth 1000000
```
```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumIOPS 32
```

## 使用 Docker 管理資源

我們提供透過 Docker 來管理容器資源子集的功能。 具體來說，我們讓使用者能夠指定 CPU 在容器間的共用方式。

### CPU

容器間的 CPU 共用可在執行階段透過 --cpu-shares 旗標來管理。 根據預設，所有容器會均分相等的 CPU 時間。 若要變更容器使用的相對 CPU 數量，請執行值從 1 到 10000 的 --cpu-shares 旗標。 根據預設，所有容器的權數皆為 5000。

```powershell 
C:\> docker run –it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## 已知問題

- CPU 和 IO 資源控制目前尚不支援 Hyper-V 容器。
- IO 資源控制目前尚不支援容器共用資料夾。





