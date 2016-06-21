---
title: &1042028547 建立虛擬交換器
description: 建立虛擬交換器
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1216525102 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 532195c6-564b-4954-97c2-5a5795368c09
---

# 建立虛擬交換器

在 Hyper-V 中建立虛擬機器之前，您可能要提供一個方法讓此虛擬機器連線到實體網路。 Hyper-V 包含以軟體為基礎的網路技術，可讓虛擬機器的網路卡連接至虛擬交換器，提供網路連線。 在 Hyper-V 中建立的每個虛擬交換器可以設定為三種連線類型的其中一種：

- <g id="1" ctype="x-strong">外部網路</g> – 虛擬交換器連接至實體網路介面卡，後者提供實體網路、Hyper-V 主機和虛擬機器之間的連線。 在此設定中，您也可以啟用或停用主機透過實體連線的網路卡進行通訊的能力。 這很實用，可以用來隔離 VM 到特定實體網路卡的流量。

- <g id="1" ctype="x-strong">內部網路</g> – 虛擬交換器並未連接至實體網路介面卡。 不過，Hyper-V 主機和連接到此交換器的任何虛擬機器之間有網路連線存在。

- <g id="1" ctype="x-strong">私人網路</g> – 虛擬交換器未連接至實體網路介面卡，且 Hyper-V 主機和連接到此交換器的任何虛擬機器之間沒有連線存在。

## 手動建立虛擬交換器

這個練習逐步解說如何使用 Hyper-V 管理員來建立外部虛擬交換器。 完成時，您的 Hyper-V 主機會包含可用來連接虛擬機器和實體網路的虛擬交換器。

1. 開啟 [Hyper-V 管理員]。

2. 以滑鼠右鍵按一下 Hyper-V 主機的名稱，選取 <g id="2" ctype="x-strong">[虛擬交換器管理員...]</g>。

3. 在 [虛擬交換器] 下選取 <g id="2" ctype="x-strong">[新虛擬網路交換器]</g>。

4. 在 [您要建立哪種類型的虛擬交換器?] 底下，選取 <g id="2" ctype="x-strong">[外部]</g>。

5. 選取 <g id="2" ctype="x-strong">[建立虛擬交換器]</g> 按鈕。

6. 在 [虛擬交換器內容] 下，指定新交換器的名稱，例如 <g id="2" ctype="x-strong">External VM Switch</g>。

7. 在 [連線類型] 下，確定已選取 <g id="2" ctype="x-strong">[外部網路]</g>。

8. 選取要搭配新虛擬交換器的實體網路卡。 這是實際連線到網路的網路卡。

    <g id="1" ctype="x-linkText"></g>

9. 選取 <g id="2" ctype="x-strong">[套用]</g> 建立虛擬交換器。 此時，您會看到下列訊息。 按一下 [是]<g id="2" ctype="x-strong"></g> 繼續作業。

    <g id="1" ctype="x-linkText"></g>

10. 選取 <g id="2" ctype="x-strong">[確定]</g> 關閉 [虛擬交換器管理員] 視窗。

## 使用 PowerShell 建立虛擬交換器

使用下列步驟可以 PowerShell 建立外部連線類型的虛擬交換器。

1. 使用 <g id="2" ctype="x-strong">Get-NetAdapter</g> 傳回連線到 Windows 10 系統的網路介面卡清單。

    ```powershell
    PS C:\> Get-NetAdapter

    Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
    ----                      --------------------                    ------- ------       ----------             ---------
    Ethernet 2                Broadcom NetXtreme 57xx Gigabit Cont...       5 Up           BC-30-5B-A8-C1-7F         1 Gbps
    Ethernet                  Intel(R) PRO/100 M Desktop Adapter            3 Up           00-0E-0C-A8-DC-31        10 Mbps  
    ```

2. 選取要與 Hyper-V 交換器搭配的網路介面卡，並在名為 <g id="2" ctype="x-strong">$net</g> 的變數中放個執行個體。

    ```
    $net = Get-NetAdapter -Name 'Ethernet'
    ```

3. 執行下列命令以建立新的 Hyper-V 虛擬交換器。

    ```
    New-VMSwitch -Name "External VM Switch" -AllowManagementOS $True -NetAdapterName $net.Name
    ```

## 虛擬交換器與膝上型電腦

如果要在膝上型電腦上執行 Windows 10 Hyper-V，您可能想同時為乙太網路和無線網路卡建立虛擬交換器。 有此設定時，您可以根據膝上型電腦連線到網路的方式，在這些交換器之間變更您的虛擬機器。 虛擬機器不會自動在有線和無線之間切換。

## 後續步驟 - 建立虛擬機器

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">建立 Windows 虛擬機器</g><g id="1CapsExtId3" ctype="x-title"></g></g>






<!--HONumber=May16_HO2-->


