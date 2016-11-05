---
title: "使用 Hyper-V 管理員管理遠端 Hyper-V 主機"
description: "使用 Hyper-V 管理員管理遠端 Hyper-V 主機"
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 2d34e98c-6134-479b-8000-3eb360b8b8a3
translationtype: Human Translation
ms.sourcegitcommit: 8f08c85921b9d41f10f3b8cff5e4bafe945bd4af
ms.openlocfilehash: c562e1a1370e9286680afa1b498d625195edebb2

---

# 使用 Hyper-V 管理員管理遠端 Hyper-V 主機

Hyper-V 管理員是內建工具，可以診斷及管理本機 Hyper-V 主機和少量的遠端主機。  本文記載使用 Hyper-V 管理員中所有支援的設定，連線到 Hyper-V 主機的設定步驟。

> 在[任何包含 Hyper-V 的 Windows OS](../quick_start/walkthrough_compatibility.md#operating-system-requirements) 上，可透過 **[程式和功能]** 的 **[Hyper-V 管理工具]**，取得 Hyper-V 管理員。  Hyper-V 平台不需要啟用，即能管理遠端主機。

若要連線到 Hyper-V 管理員中的 Hyper-V 主機，請務必在左窗格中選取 [Hyper-V 管理員]，然後再選取右窗格中的 [連線到伺服器]。

![](media/HyperVManager-ConnectToHost.png)

## Hyper-V 管理員支援的 Hyper-V 主機組合
在 Windows 10 中 Hyper-V 管理員可讓您管理下列 Hyper-V 主機：
* Windows 10
* Windows 8.1
* Windows 8
* Windows Server 2016 - 所有版本與安裝選項，包括 Nano 伺服器及對應版本的 HYPER-V 伺服器
* Windows Server 2012 R2 - 所有版本與安裝選項，以及對應版本的 HYPER-V 伺服器
* Windows Server 2012 R2 - 所有版本與安裝選項，以及對應版本的 HYPER-V 伺服器

Windows 8.1 及 Windows Server 2012 R2 中的 Hyper-V 管理員可讓您管理：
* Windows 8.1
* Windows 8
* Windows Server 2012 R2 - 所有版本與安裝選項，以及對應版本的 HYPER-V 伺服器
* Windows Server 2012 - 所有版本與安裝選項，以及對應版本的 HYPER-V 伺服器

Windows 8 及 Windows Server 2012 中的 Hyper-V 管理員可讓您管理：
* Windows 8
* Windows Server 2012 - 所有版本與安裝選項，以及對應版本的 HYPER-V 伺服器

Windows 7 及 Windows Server 2008 R2 中的 Hyper-V 管理員可讓您管理：
* Windows Server 2008 R2 - 所有版本與安裝選項，以及對應版本的 HYPER-V 伺服器

Windows Vista 及 Windows Server 2008 中的 Hyper-V 管理員可讓您管理：
* Windows Server 2008 - 所有版本與安裝選項，以及對應版本的 HYPER-V 伺服器

> **注意：**Hyper-V 管理員的功能與您所管理之版本的可用功能相符。 換言之，若您要從 Windows Server 2012 R2 管理遠端的 Windows Server 2012 主機，就無法使用 Windows Server 2012 R2 的新 HYPER-V 管理員功能。

## 管理本機主機 ##
若要將本機主機新增到 Hyper-V 管理員中作為 Hyper-V 主機，請在 [選取電腦] 對話方塊中，選取 [本機電腦]。

![](media/HyperVManager-ConnectToLocalHost.png)

如果無法建立連線：
*  請確定已啟用「Hyper-V 平台」角色。  
  請參閱[檢查相容性的逐步解說](../quick_start/walkthrough_compatibility.md)一節，以了解是否支援 Hyper-V。
*  請確認您的使用者帳戶為「Hyper-V 系統管理員」群組的成員。


## 管理相同的網域中的另一部 Hyper-V 主機 ##

若要將遠端 Hyper-V 主機新增至 Hyper-V 管理員，請在 **[選取電腦]** 對話方塊中，選取 **[另一台電腦]**，然後在文字欄位中輸入遠端主機的主機名稱、NetBIOS 或 FQDN。

![](media/HyperVManager-ConnectToRemoteHost.png)

為了管理遠端 Hyper-V 主機，在本機電腦和遠端主機上皆必須啟用遠端管理。

您可以透過 `Server Manager -> Remote management` 啟用，或以系統管理員身分執行下列 PowerShell 命令： 

``` PowerShell
Enable-PSRemoting
```

如果您目前的使用者帳戶與遠端主機上的 Hyper-V 系統管理員帳戶相符，請繼續作業，並按 **[確定]** 進行連線。  

> 這是在 Windows 8 或 Windows 8.1 中用 Hyper-V 管理員管理遠端主機的唯一支援方法。


Windows 10 大幅擴充遠端連線類型的可能組合。  
現在您可以使用主機名稱或 IP 位址連線到遠端 Windows 10 或更新版本的主機。  Hyper-V 管理員現在也支援備用的使用者認證。  


### 以不同的使用者身分連線至遠端主機
> 只有在連線到 Windows 10 或 Server 2016 Technical Preview 3 或更新版本的遠端主機時，才能使用此功能

在 Windows 10 中，如果您不是以遠端主機的正確使用者帳戶執行，可以其他使用者身分和備用認證的進行連線。

若要指定遠端 Hyper-V 主機的認證，請在 **[選取電腦]** 對話方塊中，選取 **[以其他使用者身分連接:]**，然後選取 **[設定使用者...]**。

![](media/HyperVManager-ConnectToRemoteHostAltCreds.png)


### 使用 IP 位址連線至遠端主機
> 只有在連線到 Windows 10 或 Server 2016 Technical Preview 3 或更新版本的遠端主機時，才能使用此功能

有時候使用 IP 位址進行連線比用主機名稱更容易。 Windows 10 允許您這麼做。

若要使用 IP 位址連線，請在 **[另一台電腦]** 文字欄位中，輸入 IP 位址。


## 管理網域外 (或沒有網域) 的 Hyper-V 主機 ##
> 只有在連線到 Windows 10 或 Server 2016 Technical Preview 3 或更新版本的遠端主機時，才能使用此功能

以系統管理員身分，在要管理的 Hyper-V 主機上執行下列命令：

1.  [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx)
  * [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx) 會為*私人*網路區域建立必要的防火牆規則。 若要在公用區域上允許這項存取，您必須啟用 CredSSP 和 WinRM 的這條規則。
2.  [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role server

在管理電腦上，以系統管理員身分執行下列命令：

1. Set-Item WSMan:\localhost\Client\TrustedHosts -Value "fqdn-of-hyper-v-host"
2. [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role client -DelegateComputer "fqdn-of-hyper-v-host"
3. 此外，您還需要設定下列群組原則：**電腦設定 |系統管理範本 |系統 |認證委派 |允許使用僅限 NTLM 伺服器驗證的新認證**
    * 按一下 **[啟用]**，並新增 *wsman/fqdn-of-hyper-v-host*



<!--HONumber=Nov16_HO1-->


