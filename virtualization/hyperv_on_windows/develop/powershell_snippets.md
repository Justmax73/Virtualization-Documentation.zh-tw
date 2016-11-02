---
title: "PowerShell 程式碼片段"
description: "PowerShell 程式碼片段"
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: dc33c703-c5bc-434e-893b-0c0976b7cb88
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: ff356c8dad96b5705c6d698480c6f435cedd7bd9

---

# PowerShell 程式碼片段

PowerShell 是用於 Hyper-V 的實用指令碼處理、自動化及管理工具。  以下的工具箱用於探索一些 PowerShell 能做的酷事情！

所有的 Hyper-V 管理皆需要以系統管理員的身分執行，我們假設所有的指令碼和程式碼片段是以系統管理員身分從 Hyper-V 系統管理員帳戶執行。

若不確定自己是否有正確的權限，可輸入 `Get-VM`；如果順利執行且未發生錯誤，表示您已準備就緒。


## PowerShell Direct 工具
本節所有的指令碼和程式碼片段皆仰賴下列的基本概念。

**需求**：  
*  PowerShell Direct。  Windows 10 客體和主機作業系統。

**常用變數**：  
`$VMName` -- 這是包含 VMName 的字串。  使用下列項目查看可用的 VM 清單： `Get-VM`  
`$cred` -- 客體 OS 的認證。  可以使用下列項目來填入： `$cred = Get-Credential`  

### 確認客體是否已開機

Hyper-V 管理員看不到客體作業系統，常讓人很難知道客體 OS 是否已開機。

以下相同功能的兩個檢視，第一個是程式碼片段，然後是 PowerShell 功能。

程式碼片段：  
``` PowerShell
if((Invoke-Command -VMName $VMName -Credential $cred {"Test"}) -ne "Test"){Write-Host "Not Booted"} else {Write-Host "Booted"}
```  

Function:  
``` PowerShell
function waitForPSDirect([string]$VMName, $cred){
   Write-Output "[$($VMName)]:: Waiting for PowerShell Direct (using $($cred.username))"
   while ((Invoke-Command -VMName $VMName -Credential $cred {"Test"} -ea SilentlyContinue) -ne "Test") {Sleep -Seconds 1}}
```

**結果**  
列印易讀訊息，並鎖定直到成功連接至 VM。  
成功時不會顯示訊息。

### 鎖定指令碼直到客體有網路
使用 PowerShell Direct 就可以在虛擬機器收到 IP 位址之前，連接到虛擬機器內的 PowerShell 工作階段。

``` PowerShell
# Wait for DHCP
while ((Get-NetIPAddress | ? AddressFamily -eq IPv4 | ? IPAddress -ne 127.0.0.1).SuffixOrigin -ne "Dhcp") {sleep -Milliseconds 10}
```

** 結果 ** 鎖定，直到收到 DHCP 租用為止。  由於此指令碼不會尋求特定的子網路或 IP 位址，無論您使用何種網路設定它都能運作。  
成功時不會顯示訊息。

## 使用 PowerShell 管理認證
Hyper-V 指令碼經常需要處理一或多個虛擬機器、Hyper-V 主機、或兩者的認證。

使用 PowerShell Direct 或標準 PowerShell 遠端時，有多種方法可達到此目標：

1. 第一個 (且最簡單的) 方法是讓相同的使用者認證在主機和客體上有效，或在本機和遠端主機上有效。  
  如果您用您的 Microsoft 帳戶登入，或者如果您是在網域環境中，這相當簡單。  
  在這個案例中，您可以只執行 `Invoke-Command -VMName "test" {get-process}`。

2. 讓 PowerShell 提示您輸入認證  
  如果您的認證不符，系統會自動顯示認證提示，可讓您為虛擬機器提供適當的認證。

3. 將認證儲存在變數中以便重複使用。
  執行簡單的命令，像這樣：  
  ``` PowerShell
  $localCred = Get-Credential
   ```
  然後執行這個：
  ``` PowerShell
  Invoke-Command -VMName "test" -Credential $localCred  {get-process} 
  ```
  這表示對於每個指令碼/PowerShell 工作階段，您只會看到一次提供認證的提示。

4. 將您的認證寫入指令碼。  **千萬不要在任何真實的工作負載或系統中這麼做**
 > 警告：_請不要在生產系統中這麼做。請不要對真正的密碼這麼做。_
  
  您可以用一些程式碼編寫 PSCredential 物件，像這樣：  
  ``` PowerShell
  $localCred = New-Object -typename System.Management.Automation.PSCredential -argumentlist "Administrator", (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) 
  ```
  大致上不安全，但可用來進行測試。  現在，您在這個工作階段中不會再看到任何提示。 




<!--HONumber=Oct16_HO4-->


