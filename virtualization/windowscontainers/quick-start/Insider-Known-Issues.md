# <a name="known-issues-for-insider-builds"></a>Insider build 的已知問題

## <a name="build-16237"></a>組建 16237

- Hyper-v 隔離未正常運作。 使用組建16237中的 Hyper-v 隔離需要此因應措施。 在 PowerShell 中執行這些命令：

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server 現在會以使用者身分執行，因此需要系統管理員許可權的命令將會失敗。 加入 "RUN setx /M PATH" 之類的命令列會導致組建失敗。 針對此案例，您可以使用這個替代方案：

```dockerfile
RUN setx PATH <path>
```
