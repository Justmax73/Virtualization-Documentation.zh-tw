---
title: "製作您自己的整合服務"
description: "Windows 10 整合服務。"
keywords: Windows 10, Hyper-V, HVSocket, AF_HYPERV
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
translationtype: Human Translation
ms.sourcegitcommit: b6b63318ed71931c2b49039e57685414f869a945
ms.openlocfilehash: 19e8cf269b0bef127fb06d2c99391107cd8683b1
ms.lasthandoff: 02/16/2017

---

# 製作您自己的整合服務

從 Windows 10 年度更新版開始，任何人都可以讓在 Hyper-V 主機之間通訊，並使用虛擬電腦的應用程式使用 Hyper-V 通訊端 -- 此通訊端為使用新位址家族，且具備以虛擬電腦為目標之特殊端點的 Windows 通訊端。  所有透過 Hyper-V 通訊端的通訊，都不使用網路功能，且所有的資料會留在相同的實體記憶體上。   使用 Hyper-V 通訊端的應用程式，類似於 Hyper-V 的整合服務。

本文件會逐步介紹如何用 Hyper-V 通訊端為基礎來建立簡單的程式。

**支援的主機 OS**
* Windows 10 予以支援
* Windows Server 2016
* 未來版本 (Server 2016 +)

**支援的客體 OS**
* Windows 10
* Windows Server Technical Preview 4 及更新版本
* 未來版本 (Server 2016 +)
* 使用 Linux 整合服務的 Linux 客體 (請參閱 [Supported Linux and FreeBSD virtual machines for Hyper-V on Windows](https://technet.microsoft.com/library/dn531030(ws.12).aspx) (Windows 上 Hyper-V 支援的 Linux 及 FreeBSD 虛擬機器)

**功能和限制**  
* 支援核心模式或使用者模式動作  
* 僅限資料流      
* 沒有區塊記憶體 (並非備份/視訊的最佳選擇) 

--------------

## 開始使用

需求：
* C/C++ 編譯器。  如果您沒有的話，請查看 [Visual Studio 社群](https://aka.ms/vs)
* [Windows 10 SDK](https://developer.microsoft.com/windows/downloads/windows-10-sdk) -- 已預先安裝於 Visual Studio 2015 內，並包含 Update 3 及以上版本。
* 搭配至少一個虛擬電腦來執行其中一個上述主機作業系統的電腦。 -- 這是用以測試您的應用程式。

> **注意︰**Hyper-V 通訊端的 API 會在稍後於 Windows 10 中公開可用。  使用 HVSocket 的應用程式將在任何 Widnows 10 主機和客體上執行，但僅限使用 Windows SDK 組建 14290 之後的版本進行開發。  

## 註冊新的應用程式
若要使用 Hyper-V 通訊端，必須在 Hyper-V 主機的登錄中註冊應用程式。

在登錄中註冊服務後，您將獲得：
*  可啟用、停用即列出可用服務的 WMI 管理
*  直接與虛擬機器通訊的權限

下列 PowerShell 將會註冊名為 "HV Socket Demo" 的新應用程式。  這必須以系統管理員身分執行。  操作指示如下。

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name 
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```


**登錄位置和資訊：**  
``` 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```  
在此登錄位置中，您會看見數個 GUID。  這是我們的隨附服務。

每個服務的登錄資訊：
* `Service GUID`   
    * `ElementName (REG_SZ)` -- 這是服務的易記名稱

若要註冊您自己的服務，請使用您自己的 GUID 和好記名稱建立新的登錄機碼。

好記名稱將會與您的新應用程式相關聯。  它會出現在效能計數器中，以及其他不適用 GUID 之處。

登錄項目將如下所示：
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName    REG_SZ    VM Session Service
    YourGUID\
        ElementName    REG_SZ    Your Service Friendly Name
```

> **提示：**若要在 PowerShell 中產生 GUID，並將其複製到剪貼簿，請執行：  
``` PowerShell
(New-Guid).Guid | clip.exe
```

## 建立 Hyper-V 通訊端

在大部分的基本案例中，定義通訊端都需要位址家族、連線類型和通訊協定。

以下是簡單的[通訊端定義](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
)

``` C
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);
``` 

若是 Hyper-V 通訊端：
* 位址家族 - `AF_HYPERV`
* 類型 - `SOCK_STREAM`
* 通訊協定 - `HV_PROTOCOL_RAW`


以下是範例宣告/實例：  
``` C
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);
```


## 繫結至 Hyper-V 通訊端

繫結可建立通訊端與連線資訊的關聯。

為了方便起見，功能定義複製如下，若想進一步了解繫結，請參閱[這裡](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx)。

``` C
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);
```

有別於標準網際網路通訊協定位址家族 (`AF_INET`) 的通訊端位址 (sockaddr)，其由主機電腦的 IP 位址和該主機的連接埠號碼所組成，`AF_HYPERV` 的通訊端位址則使用虛擬機器的識別碼和上面定義的應用程式識別碼來建立連線。 

由於 Hyper-V 通訊端並未依賴網路堆疊、TCP/IP、DNS 等項目，通訊端的端點必須為可全然明確描述連線的非 IP、非主機名稱格式。

以下是 Hyper-V 通訊端的通訊端位址定義：

``` C
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};
```

AF_HYPERV 端點並不依賴 IP 或主機名稱，而是高度依賴兩個 GUID：  
* VM ID – 這是為每個 VM 指派的唯一 ID。  VM 的 ID 可使用下列 PowerShell 指令碼片段來尋找。  
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* 服務識別碼 – GUID，[如前所述](#RegisterANewApplication)，可供應用程式在 Hyper-V 主機登錄中進行註冊。

此外還有一組在連線到非特定虛擬機器時可使用的 VMID 萬用字元。
 
### VMID 萬用字元

| 名稱 | GUID | 描述 |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | 接聽程式應繫結至此 VmId，才可接受來自所有分割區的連線。 |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | 接聽程式應繫結至此 VmId，才可接受來自所有分割區的連線。 |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |  
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | 子項的萬用字元位址。 接聽程式應繫結至此 VmId，才可接受來自其子項的連線。 |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | 回送位址。 使用此 VmId，可連接到與連接器相同的分割區。 |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | 父項位址。 使用此 VmId，可連接到連接器的父分割。* |


\* `HV_GUID_PARENT`  
虛擬機器的父系是其主機。  容器的父系是容器的主機。  
從執行於虛擬機器中的容器連接，將會連接到主控容器的 VM。  
在此 VmId 上接聽，可接受下列來源的連線：  
(在容器內)：容器主機。  
(在 VM 內：容器主機/無容器)：VM 主機。  
(不在 VM 內：容器主機/無容器)：不支援。

## 支援的通訊端命令

Socket()  
Bind()  
Connect()  
Send()  
Listen()  
Accept()  

## 實用的連結
[完整 WinSock API](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

[Hyper-V 整合服務參考](../reference/integration-services.md)
