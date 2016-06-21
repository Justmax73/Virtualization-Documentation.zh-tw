---
title: &1042028547 在 Windows 10 Hyper-V 中部署 Windows 虛擬機器
description: 在 Windows 10 Hyper-V 中部署 Windows 虛擬機器
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &797369065 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 66723f33-b12c-49d1-82cf-71ba9d6087e9
---

# 在 Windows 10 Hyper-V 中部署 Windows 虛擬機器

您可以建立虛擬機器，然後以許多不同方式部署其作業系統，例如使用 Windows 部署服務、附加已備妥的虛擬硬碟、或手動使用安裝媒體。 本文逐步解說如何建立虛擬機器，並使用作業系統安裝媒體將作業系統部署到虛擬機器。

開始這個練習之前，您需要欲部署之作業系統的 .iso 檔案。 如有需要，從 <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">TechNet 評估中心</g><g id="2CapsExtId3" ctype="x-title"></g></g>取得試用版的 Windows 8.1 或 Windows 10。

## 使用 Hyper-V 管理員建立虛擬機器

這些步驟逐步解說如何手動建立虛擬機器，並將作業系統部署到此虛擬機器。

1. 在 Hyper-V 管理員中，按一下 <g id="2" ctype="x-strong">[動作]</g> > <g id="4" ctype="x-strong">[新增]</g> > <g id="6" ctype="x-strong">[虛擬機器]</g> 即可啟動 [新增虛擬機器精靈]。

2. 閱讀 [在您開始前] 的內容，然後按 <g id="2" ctype="x-strong">[下一步]</g>。

3. 為虛擬機器命名。
> <g id="1" ctype="x-strong">注意︰</g>這是 Hyper-V 用於虛擬機器的名稱，而不是指定給將在虛擬機器內部署之客體作業系統的電腦名稱。

4. 選擇將儲存虛擬機器檔案的位置，例如 <g id="2" ctype="x-strong">c:\virtualmachine</g>。 您也可以使用預設位置。 完成時按 <g id="2" ctype="x-strong">[下一步]</g>。

  <g id="1" ctype="x-linkText"></g>

5. 選取機器的世代，然後按 <g id="2" ctype="x-strong">[下一步]</g>。

  Windows Server 2012 R2 引進第 2 代虛擬機器，可提供簡化的虛擬硬體模型和一些額外的功能。 64 位元作業系統只能安裝在第 2 代虛擬機器上。 如需第 2 代虛擬機器的詳細資訊，請參閱<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">第 2 代虛擬機器概觀</g><g id="2CapsExtId3" ctype="x-title"></g></g>。

> 如果新的虛擬機器設定為第 2 代，並將執行 Linux 散發套件，則必須停用安全開機。 如需安全開機的詳細資訊，請參閱<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">安全開機</g><g id="2CapsExtId3" ctype="x-title"></g></g>。

6. 選取 <g id="2" ctype="x-strong">2048</g> MB 作為 <g id="4" ctype="x-strong">[啟動記憶體]</g> 的值，並維持選取 <g id="6" ctype="x-strong">[Use Dynamic Memory] (使用動態記憶體)</g>。 按 <g id="2" ctype="x-strong">[下一步]</g> 按鈕。

  Hyper-V 主機與在主機上執行的虛擬機器之間會共用記憶體。 可以在單一主機上執行的虛擬機器數目，有一部分取決於可用的記憶體。 也可以將虛擬機器設定為使用動態記憶體。 若啟用，動態記憶體會從執行中的虛擬機器回收未使用的記憶體。 這可讓更多個虛擬機器在主機上執行。 如需動態記憶體的詳細資訊，請參閱 <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Hyper-V 動態記憶體概觀</g><g id="2CapsExtId3" ctype="x-title"></g></g>。

7. 在 [設定網路] 精靈中，選取虛擬機器的虛擬交換器，然後按 <g id="2" ctype="x-strong">[下一步]</g>。 如需詳細資訊，請參閱<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">建立虛擬交換器</g><g id="2CapsExtId3" ctype="x-title"></g></g>。

8. 指定虛擬硬碟的名稱，選取位置或保留預設值，最後，指定大小。 一切就緒後按 <g id="2" ctype="x-strong">[下一步]</g>。

  虛擬硬碟提供給虛擬機器的儲存體，類似於實體硬碟。 需有虛擬硬碟，才能在虛擬機器上安裝作業系統。

  <g id="1" ctype="x-linkText"></g>

9. 在 [安裝選項] 精靈中，選取 <g id="2" ctype="x-strong">[從可開機映像檔安裝作業系統]</g>，然後選取作業系統 .iso 檔。 完成之後按 <g id="2" ctype="x-strong">[下一步]</g>。

  建立虛擬機器時，您可以設定某些作業系統安裝選項。 三個選項如下：

  - <g id="1" ctype="x-strong">稍後安裝作業系統</g> – 此選項對虛擬機器不會做任何其他修改。

  - <g id="1" ctype="x-strong">從可開機映像檔安裝作業系統</g> – 這類似將 CD 插入實體電腦的 CD-ROM 光碟機。 若要設定此選項，請選取 .iso 映像。 此映像將會掛接到虛擬機器的虛擬 CD-ROM 光碟機。 虛擬機器的開機順序會變更為先從 CD-ROM 光碟機開機。

  - <g id="1" ctype="x-strong">從網路安裝伺服器安裝作業系統</g> – 除非您已將虛擬機器連接至網路交換器，否則無法使用此選項。 在此設定中，虛擬機器會嘗試從網路開機。

10. 檢閱虛擬機器詳細資料，然後按一下 <g id="2" ctype="x-strong">[完成]</g>，完成虛擬機器的建立。

## 使用 PowerShell 建立虛擬機器

1. 以系統管理員身分開啟 PowerShell ISE。

2. 請執行下列指令碼。

  ```powershell
  # Set VM Name, Switch Name, and Installation Media Path.
  $VMName = 'TESTVM'
  $Switch = 'External VM Switch'
  $InstallMedia = 'C:\Users\Administrator\Desktop\en_windows_10_enterprise_x64_dvd_6851151.iso'

  # Create New Virtual Machine
  New-VM -Name $VMName -MemoryStartupBytes 2147483648 -Generation 2 -NewVHDPath "D:\Virtual Machines\$VMName\$VMName.vhdx" -NewVHDSizeBytes 53687091200 -Path "D:\Virtual Machines\$VMName" -SwitchName $Switch

  # Add DVD Drive to Virtual Machine
  Add-VMScsiController -VMName $VMName
  Add-VMDvdDrive -VMName $VMName -ControllerNumber 1 -ControllerLocation 0 -Path $InstallMedia

  # Mount Installation Media
  $DVDDrive = Get-VMDvdDrive -VMName $VMName

  # Configure Virtual Machine to Boot from DVD
  Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive
  ```

## 完成作業系統部署

若要完成虛擬機器的建置，您必須啟動虛擬機器，然後逐步完成作業系統安裝。

1. 在 [Hyper-V 管理員] 中，按兩下該虛擬機器。 這會啟動 VMConnect 工具。

2. 在 VMConnect 中，按一下綠色的 [開始] 按鈕。 這就相當於按下實體電腦上的電源按鈕。 系統可能會提示您「按任意鍵從 CD 或 DVD 光碟開機」。 請按照提示執行這項操作。
> <g id="1" ctype="x-strong">注意︰</g>您可能需要在 VMConnect 視窗內按一下，以確保您的按鍵輸入會傳送至虛擬機器。

3. 虛擬機器會開機進入安裝程式，然後您可以逐步完成安裝，就像在實體電腦上的做法一樣。

  <g id="1" ctype="x-linkText"></g>

> <g id="1" ctype="x-strong">注意：</g>除非您是執行大量授權版本的 Windows，否則在虛擬機器內執行的 Windows 需要個別授權。 虛擬機器的作業系統與主機作業系統無關。

## 下一個步驟 - 使用 PowerShell 和 Hyper-V

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Hyper-V 和 Windows PowerShell</g><g id="1CapsExtId3" ctype="x-title"></g></g>





<!--HONumber=May16_HO2-->


