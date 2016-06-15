---
title: 巢狀虛擬化
description: 巢狀虛擬化
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
---

# 巢狀虛擬化

巢狀虛擬化提供在虛擬化環境中執行 HYPER-V 主機的能力。 換言之，使用巢狀虛擬化，HYPER-V 主機本身也能虛擬化。 舉例來說，巢狀虛擬化可讓您在虛擬化環境中執行 HYPER-V 實驗室，為其他人提供虛擬化服務而無須配備個別的硬體；當在虛擬化容器主機上執行 HYPER-V 容器時，Windows 容器技術會使用巢狀虛擬化。 本文件詳細說明軟硬體先決條件、設定步驟，以及如何疑難排解問題。

> 巢狀虛擬化目前為預覽版，不應用於生產環境中。

## 先決條件

- 執行組建 10565 或更新版本的 Windows 測試人員組建 (Windows Server 2016、Nano Server 或 Windows 10)。
- 兩個 Hypervisor (父項與子項) 都必須執行同一版 Windows 組建 (10565 或更新版本)。
- 最少 4 GB RAM。
- 搭載 Intel VT x 技術的 Intel 處理器。

## 設定巢狀虛擬化

首先須建立虛擬機器主機，而且其執行的組建必須與您的主機相同。**請勿啟動虛擬機器**。 如需詳細資訊，請參閱[建立虛擬機器](../quick_start/walkthrough_create_vm.md)。

建立好虛擬機器之後，請在父 Hypervisor 上執行下列命令，以啟用虛擬機器上的巢狀虛擬化。

```none
Set-VMProcessor -VMName <virtual machine> -ExposeVirtualizationExtensions $true -Count 2
```

執行巢狀 HYPER-V 主機時，必須停用虛擬機器上的動態記憶體。 您可以透過虛擬機器的屬性加以設定，也可以使用下列 PowerShell 命令來設定。

```none
Set-VMMemory <virtual machine> -DynamicMemoryEnabled $false
```

為使巢狀虛擬機器能夠接收 IP 位址，必須啟用 MAC 位址詐騙。 若要啟用此設定，請使用下列 PowerShell 命令。

```none
Get-VMNetworkAdapter -VMName <virtual machine> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

完成這些步驟後，即可啟動虛擬機器，Hyper-V 也完成安裝。 如需如何安裝 Hyper-V 的詳細資訊，請參閱 [安裝 Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)。

## 設定指令碼

您可以使用下列指令碼啟用及設定巢狀虛擬化，但並非必要。 下列命令會下載並執行該指令碼。
  
```none
# download script
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Enable-NestedVm.ps1 -OutFile .\Enable-NestedVm.ps1 

# run script
.\Enable-NestedVm.ps1 -VmName "DemoVM"
```

## 已知問題

- 啟用 Device Guard 的主機無法向客體公開虛擬化延伸模組。
- 啟用虛擬式安全性 (VBS) 的主機無法向客體公開虛擬化延伸模組。 若要使用巢狀虛擬化，必須先停用 VBS。
- 若使用空白密碼，可能會造成虛擬機器連線中斷。 變更密碼之後，問題應該就能迎刃而解。
- 虛擬機器一旦啟用巢狀虛擬化，下列功能將不再與該 VM 相容。  
  * 調整執行階段記憶體的大小。
  * 對執行中的虛擬機器套用檢查點。
  * 虛擬機器如有裝載其他虛擬機器，無法即時移轉。
  * 儲存與還原兩項動作無法運作。

## 常見問題和疑難排解

我的虛擬機器無法啟動，我該怎麼做？

1. 請確認動態記憶體已「關閉」。
2. 在主機電腦上，從提高權限的提示中，執行[此 PowerShell 指令碼](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1)。
  
## 意見反應

您可以利用 Windows 意見反應應用程式、[虛擬化論壇](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv)或 [GitHub](https://github.com/Microsoft/Virtualization-Documentation) 回報其他問題。



<!--HONumber=Jun16_HO2-->


