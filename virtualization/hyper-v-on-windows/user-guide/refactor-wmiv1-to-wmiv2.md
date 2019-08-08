---
title: 將 Hyper-V WMIv1 移植至 WMIv2
description: 了解如何將 Hyper-V WMIv1 移植至 WMIv2
keywords: windows 10, hyper-v, WMIv1, WMIv2, WMI, Msvm_VirtualSystemGlobalSettingData, root\virtualization
author: scooley
ms.date: 04/13/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: b13a3594-d168-448b-b0a1-7d77153759a8
ms.openlocfilehash: 963a1cc356c34c8d051c427a069c49021e3c0d27
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998905"
---
# <a name="move-from-hyper-v-wmi-v1-to-wmi-v2"></a>從 Hyper-V WMI v1 移至 WMI v2

Windows Management Instrumentation (WMI) 是 Hyper-V 管理員和 Hyper-V 的 PowerShell Cmdlet 的基礎管理介面。  雖然大部分的人都會使用我們的 PowerShell Cmdlet 或 Hyper-V 管理員，但有時候開發人員會直接需要 WMI。  

已經有兩個 Hyper-V WMI 命名空間 (或 Hyper-V WMI API 的版本)。
* WMI v1 命名空間 (root\virtualization) 於 Windows Server 2008 導入，最後可用於 Windows Server 2012
* WMI v2 命名空間 (root\virtualization\v2) 於 Windows Server 2012 導入

本文件包含將用於舊 WMI 命名空間的程式碼轉換成新程式碼的資源參考資料。  一開始，本文可當作 API 資訊和範例程式碼/指令碼的存放庫，協助將使用 Hyper-V WMI API 的任何程式或指令碼從 v1 命名空間移植至 v2 命名空間。

## <a name="msdn-samples"></a>MSDN 範例

[Hyper-V 虛擬機器移轉範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-machine-aef356ee)  
[Hyper-V 虛擬光纖通道範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-Fiber-35d27dcd)  
[Hyper-V 計劃的虛擬機器範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-planned-virtual-8c7b7499)  
[Hyper-V 應用程式健康情況監視範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-application-health-dc0294f2)  
[虛擬硬碟管理範例](http://code.msdn.microsoft.com/windowsdesktop/Virtual-hard-disk-03108ed3)  
[Hyper-V 複本範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-replication-sample-d2558867)  
[Hyper-V 計量範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-metrics-sample-2dab2cb1)  
[Hyper-V 動態記憶體範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-dynamic-memory-9b0b1d05)  
[Hyper-V 可擴充式交換器延伸模組篩選器驅動程式](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-Extensible-Virtual-e4b31fbb)  
[Hyper-V 網路功能範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-networking-sample-7c47e6f5)  
[Hyper-V 資源集區管理範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-resource-pool-df906d95)  
[Hyper-V 復原快照範例](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-recovery-snapshot-ea72320c)  

## <a name="samples-from-blogs"></a>擷取自部落格的範例

[使用 Hyper-V WMI V2 命名空間新增網路介面卡至 VM](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/adding-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空間將 VM 網路介面卡連接至交換器](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/connecting-a-vm-network-adapter-to-a-switch-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空間來變更 NIC 的 MAC 位址](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/changing-the-mac-address-of-nic-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空間從 VM 移除網路介面卡](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空間將 VHD 連接至 VM](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/attaching-a-vhd-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空間從 VM 移除 VHD](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-vhd-from-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[使用 Hyper-V WMI V2 命名空間來建立 VM](http://blogs.msdn.com/b/virtual_pc_guy/archive/2013/06/20/creating-a-virtual-machine-with-wmi-v2.aspx)

