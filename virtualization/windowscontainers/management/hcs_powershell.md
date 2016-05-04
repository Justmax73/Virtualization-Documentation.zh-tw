



# 管理互通性

**這是初版內容，後續可能會變更。**

大多數的情況下，以 PowerShell 建立的 Windows 容器必須使用 PowerShell 來管理，以 Docker 建立的容器則必須使用 Docker 來管理。 不過，主機計算 PowerShell 模組所提供的能力可找出並停止**執行中**的容器，無論其建立方式為何。 對於在容器主機上執行的容器，此模組會像「工作管理員」一般執行。

## 顯示所有容器

若要傳回容器清單，請使用 `Get-ComputeProcess` 命令。

```powershell
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## 停止容器

若要停止容器 (無論是以 PowerShell 還是 Docker 建立的)，請使用 `Stop-ComputeProcess` 命令。

> 在寫入時，VMMS 服務將必須重新啟動，使容器能夠在使用 `Get-Container` 命令時顯示為已停止。

```powershell
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```






<!--HONumber=Feb16_HO3-->


