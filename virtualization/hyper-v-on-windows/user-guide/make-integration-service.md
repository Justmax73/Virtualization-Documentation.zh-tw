---
title: 製作您自己的整合服務
description: Windows 10 整合服務。
keywords: Windows 10, Hyper-V, HVSocket, AF_HYPERV
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
ms.openlocfilehash: 89a36ee87bce1da18852f0ebff248e239165eb7d
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911028"
---
# <a name="make-your-own-integration-services"></a>製作您自己的整合服務

從 Windows 10 年度更新版開始，任何人都可以讓在 Hyper-V 主機之間通訊，並使用虛擬電腦的應用程式使用 Hyper-V 通訊端 -- 此通訊端為使用新位址家族，且具備以虛擬電腦為目標之特殊端點的 Windows 通訊端。  所有透過 Hyper-V 通訊端的通訊，都不使用網路功能，且所有的資料會留在相同的實體記憶體上。 使用 Hyper-V 通訊端的應用程式，類似於 Hyper-V 的整合服務。

本文件會逐步介紹如何用 Hyper-V 通訊端為基礎來建立簡單的程式。

**支援的主機 OS**
* Windows 10 和更新版本
* Windows Server 2016 和更新版本

**支援的客體 OS**
* Windows 10 和更新版本
* Windows Server 2016 和更新版本
* 使用 Linux 整合服務的 Linux 客體 (請參閱 [Windows 上 Hyper-V 支援的 Linux 及 FreeBSD 虛擬機器](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows)，英文內容)
> **注意：** 支援的 Linux 客體必須對下列項目提供核心支援：
> ```bash
> CONFIG_VSOCKET=y
> CONFIG_HYPERV_VSOCKETS=y
> ```

**功能和限制**
* 支援核心模式或使用者模式動作
* 僅限資料流
* 沒有區塊記憶體 (並非備份/視訊的最佳選擇)

--------------

## <a name="getting-started"></a>開始使用

需求：
* C/C++ 編譯器。  如果您沒有的話，請查看 [Visual Studio 社群](https://aka.ms/vs)
* [Windows 10 SDK](https://developer.microsoft.com/windows/downloads/windows-10-sdk) -- 已預先安裝於 Visual Studio 2015 內，並包含 Update 3 及以上版本。
* 搭配至少一個虛擬電腦來執行其中一個上述主機作業系統的電腦。 -- 這是用以測試您的應用程式。

> **注意：** 適用于 Hyper-v 通訊端的 API 在 Windows 10 年度更新版中公開提供。 使用 Hvsocket.h 的應用程式將會在任何 Windows 10 主機和來賓上執行，但只能使用比組建14290更晚的 Windows SDK 來開發。

## <a name="register-a-new-application"></a>註冊新的應用程式
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

易記名稱將會與您的新應用程式相關聯。  它會出現在效能計數器中，以及其他不適用 GUID 之處。

登錄項目將如下所示：
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName    REG_SZ    VM Session Service
    YourGUID\
        ElementName    REG_SZ    Your Service Friendly Name
```

> **注意：** Linux 客體的服務 GUID 使用 VSOCK 通訊協定，透過 `svm_cid` 和 `svm_port` (而非 guid) 定址。 為了彌補與 Windows 之間的這種不一致情形，使用知名的 GUID 做為主機上的服務範本，這會在客體中轉譯為連接埠。 若要自訂您的服務 GUID，只需將第一個「00000000」變更為所需的連接埠號碼。 例如：「00000ac9」是連接埠 2761。
> ```C++
> // Hyper-V Socket Linux guest VSOCK template GUID
> struct __declspec(uuid("00000000-facb-11e6-bd58-64006a7986d3")) VSockTemplate{};
>
>  /*
>   * GUID example = __uuidof(VSockTemplate);
>   * example.Data1 = 2761; // 0x00000AC9
>   */
> ```
>

> **提示：** 若要在 PowerShell 中產生 GUID，並將其複製到剪貼簿，請執行：
>``` PowerShell
>(New-Guid).Guid | clip.exe
>```

## <a name="create-a-hyper-v-socket"></a>建立 Hyper-V 通訊端

在大部分的基本案例中，定義通訊端都需要位址家族、連線類型和通訊協定。

以下是簡單的[通訊端定義](https://docs.microsoft.com/windows/desktop/api/winsock2/nf-winsock2-socket)

``` C
// Windows
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);

// Linux guest
int socket(int domain, int type, int protocol);
```

若是 Hyper-V 通訊端：
* 位址家族 - `AF_HYPERV` (Windows) 或 `AF_VSOCK` (Linux 客體)
* 類型 - `SOCK_STREAM`
* 通訊協定 - `HV_PROTOCOL_RAW` (Windows) 或 `0` (Linux 客體)


以下是範例宣告/實例：
``` C
// Windows
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);

// Linux guest
int sock = socket(AF_VSOCK, SOCK_STREAM, 0);
```

## <a name="bind-to-a-hyper-v-socket"></a>繫結至 Hyper-V 通訊端

繫結可建立通訊端與連線資訊的關聯。

為了方便起見，功能定義複製如下，若想進一步了解繫結，請參閱[這裡](https://docs.microsoft.com/windows/desktop/api/winsock/nf-winsock-bind)。

``` C
// Windows
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);

// Linux guest
int bind(int sockfd, const struct sockaddr *addr,
         socklen_t addrlen);
```

有別於標準網際網路通訊協定位址家族 (`AF_INET`) 的通訊端位址 (sockaddr)，其由主機電腦的 IP 位址和該主機的連接埠號碼所組成，`AF_HYPERV` 的通訊端位址則使用虛擬機器的識別碼和上面定義的應用程式識別碼來建立連線。 如果繫結來自 Linux 客體，`AF_VSOCK` 使用 `svm_cid` 和 `svm_port`。

由於 Hyper-V 通訊端並未依賴網路堆疊、TCP/IP、DNS 等項目，因此通訊端的端點需要仍可明確說明連線的非 IP (而不是主機名稱) 格式。

以下是 Hyper-V 通訊端的通訊端位址定義：

``` C
// Windows
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};

// Linux guest
// See include/uapi/linux/vm_sockets.h for more information.
struct sockaddr_vm {
    __kernel_sa_family_t svm_family;
    unsigned short svm_reserved1;
    unsigned int svm_port;
    unsigned int svm_cid;
    unsigned char svm_zero[sizeof(struct sockaddr) -
                   sizeof(sa_family_t) -
                   sizeof(unsigned short) -
                   sizeof(unsigned int) - sizeof(unsigned int)];
};
```

AF_HYPERV 端點並不依賴 IP 或主機名稱，而是高度依賴兩個 GUID：
* VM ID – 這是為每個 VM 指派的唯一 ID。  VM 的 ID 可使用下列 PowerShell 指令碼片段來尋找。
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* 服務識別碼 – GUID，[如前所述](#register-a-new-application)，可供應用程式在 Hyper-V 主機登錄中進行註冊。

此外還有一組在連線到非特定虛擬機器時可使用的 VMID 萬用字元。

### <a name="vmid-wildcards"></a>VMID 萬用字元

| 名稱 | GUID | 說明 |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | 接聽程式應繫結至此 VmId，才可接受來自所有分割區的連線。 |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | 接聽程式應繫結至此 VmId，才可接受來自所有分割區的連線。 |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | 子項的萬用字元位址。 接聽程式應繫結至此 VmId，才可接受來自其子項的連線。 |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | 回送位址。 使用此 VmId，可連接到與連接器相同的分割區。 |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | 父項位址。 使用此 VmId，可連接到連接器的父分割。* |


\* `HV_GUID_PARENT` 虛擬機器的父系是其主機。  容器的父項是容器的主機。
從執行於虛擬機器中的容器連接，將會連接到主控容器的 VM。
在此 VmId 上接聽，可接受下列來源的連線：(在容器內)：容器主機。
(在 VM 內：容器主機/無容器)：VM 主機。
(不在 VM 內：容器主機/無容器)：不支援。

## <a name="supported-socket-commands"></a>支援的通訊端命令

Socket() Bind() Connect() Send() Listen() Accept()

## <a name="useful-links"></a>實用的連結
[完整 WinSock API](https://docs.microsoft.com/windows/desktop/WinSock/winsock-functions)

[Hyper-v Integration Services 參考](../reference/integration-services.md)
