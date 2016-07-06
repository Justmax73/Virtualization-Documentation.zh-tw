---
title: "使用 Hyper-V 與 Windows PowerShell"
description: "使用 Hyper-V 與 Windows PowerShell"
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: a8e567b6447aa73f14825b7054d977d2b003a726

---

# 使用 Hyper-V 與 Windows PowerShell

您已經逐步進行部署 Hyper-V、建立虛擬機器、管理這些虛擬機器的基本作業，現在，讓我們來探索如何使用 PowerShell 將這許多活動自動化。

### 傳回 Hyper-V 命令清單

1.  按一下 Windows 開始按鈕，輸入 **PowerShell**。
2.  執行下列命令，以顯示搜尋到的 Hyper-V PowerShell 模組可用 PowerShell 命令清單。

 ```powershell
get-command -module hyper-v | out-gridview
```
  結果類似這樣：

  ![](media\command_grid.png)

3. 若要深入了解特定 PowerShell 命令，請使用 `get-help`。 例如，執行下列命令會傳回 `get-vm` Hyper-V 命令的相關資訊。

  ```powershell
get-help get-vm
```
 輸出會顯示如何建構命令、有哪些必要和選擇性參數、以及您可以使用的別名。

 ![](media\get_help.png)


### 傳回虛擬機器清單

使用 `get-vm` 命令，以傳回虛擬機器的清單。

1. 在 PowerShell 中執行下列命令：
 
 ```powershell
get-vm
```
 結果類似這樣：

 ![](media\get_vm.png)

2. 若只要傳回已開機的虛擬機器清單，請在 `get-vm` 命令中加入篩選條件。 使用 where 物件命令可以加入篩選器。 如需篩選的詳細資訊，請參閱 [Using the Where-Object Cmdlet](https://technet.microsoft.com/en-us/library/ee177028.aspx) (使用 Where-Object Cmdlet) 文件。   

 ```powershell
 get-vm | where {$_.State -eq ‘Running’}
 ```
3.  若要列出所有電源關閉狀態的虛擬機器清單，執行下列命令。 此命令是步驟 2 中使用的命令，只是篩選器從 Running 改為 Off。

 ```powershell
 get-vm | where {$_.State -eq ‘Off’}
 ```

### 啟動和關閉虛擬機器

1. 若要啟動特定虛擬機器，請用虛擬機器的名稱執行下列命令：

 ```powershell
 Start-vm -Name <virtual machine name>
 ```

2. 若要啟動所有目前電源關閉的虛擬機器，取得這些機器的清單，並使用管線傳送此清單給 start-vm 命令：

  ```powershell
 get-vm | where {$_.State -eq ‘Off’} | start-vm
 ```
3. 若要關閉所有執行中的虛擬機器，執行此命令：
 
  ```powershell
 get-vm | where {$_.State -eq ‘Running’} | stop-vm
 ```

### 建立 VM 檢查點

若要使用 PowerShell 建立檢查點，請使用 `get-vm` 命令選取虛擬機器，並將此輸送給 `checkpoint-vm` 命令。 最後，使用 `-snapshotname` 為檢查點命名。 完整的命令看起來像這樣：

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### 建立新的虛擬機器

下列範例示範如何在 PowerShell 整合式指令碼環境 (ISE) 中建立新的虛擬機器。 這是一個簡單的範例，可以再擴大包，含其他 PowerShell 功能和更進階的 VM 部署。

1. 若要開啟 PowerShell ISE，請按一下開始，輸入 **PowerShell ISE**。
2. 執行下列程式碼建立虛擬機器。 如需 New-VM 命令的詳細資訊，請參閱 [New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx) 文件。

  ```powershell
 $VMName = "VMNAME"

 $VM = @{
     Name = $VMName 
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName "
     SwitchName = (get-vmswitch).Name[0]
 }

 New-VM @VM
  ```

## 結語與參考資料

本文件示範了一些簡單的步驟，來介紹 Hyper-V PowerShell 模組以及一些範例案例。 如需 Hyper-V PowerShell 模組的詳細資訊，請參閱 [Windows PowerShell 參考中的 Hyper-V Cmdlet](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx) 。  
 


<!--HONumber=Jun16_HO4-->


