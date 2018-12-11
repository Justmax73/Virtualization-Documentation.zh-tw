---
title: 設定直接與 Azure 的虛擬網路中的資源通訊的巢狀虛擬機器
description: 巢狀虛擬化
keywords: windows 10 的 hyper-v Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: abe6f0da68ff90af0b2b5e675f70f106d42ca81c
ms.sourcegitcommit: 8db42caaace760b7eeb1367b631b38e7904a9f26
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2018
ms.locfileid: "8962306"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>設定 Azure 的虛擬網路中的資源與通訊的巢狀虛擬機器

部署及設定 Azure 內的巢狀的虛擬機器上的原始指導方針必須要在您透過 NAT 交換器存取這些 Vm。 這會顯示數個限制：

1. 巢狀虛擬機器無法存取資源內部部署或 Azure 虛擬網路內。
2. 內部部署資源或 Azure 中的資源，才能存取巢狀虛擬機器透過 NAT，這表示多個來賓不能共用相同的連接埠。

本文件會逐步引導部署，因此我們利用 RRAS，某些使用者定義路由和 「 浮動 」 的位址空間，以允許巢狀行為及通訊像直接將 VNet Azure 內部署任何其他虛擬機器的虛擬機器。

請開始本指南中，： 之前

1. 讀取[此處提供的指導方針](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization)上巢狀虛擬化、 建立能夠您巢狀虛擬機器，以及安裝 HYPER-V 角色，這些虛擬機器內。 請勿繼續超過設定 HYPER-V 角色。
2. 閱讀本文整個之前的實作。

本指南假設目標環境相關下列幾點：

1. 我們會作業[集線器及網幅拓撲](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)中，與我們 hub 連線到 ExpressRoute。
1. 我們支點網路指派 10.0.0.0/23，劃分成兩個 /24 子網路的位址空間。
    * 10.0.0.0/24 – 我們的 HYPER-V 主機的所在位置的子網路。
    * 10.0.1.0/24 – 這會是 「 浮動 」 的子網路。 此種位址空間將會由我們的巢狀虛擬機器，且存在來處理路由回到內部部署的廣告。
    * 支點 VNet 巧妙名為 「 支點 」。

1. 我們 hub 網路 IP 範圍是不相關，但知道它的名稱是 「 中樞 」。
1. 我們 HYPER-V 會指派 10.0.0.4/24 的位址。
1. 我們在 10.0.0.10/24 有 DNS 伺服器，但這並非必要，但我們逐步解說的假設。

## <a name="high-level-overview-of-what-were-doing-and-why"></a>高的層級概觀的我們的正在進行的動作和原因

* 背景： 巢狀虛擬機器將不會收到 DHCP 從其主機連接到，即使您設定的內部或外部交換器 VNet。 
  * 這表示在 HYPER-V 主機必須提供 DHCP。
* 我們將會使用透過 HYPER-V 主機只需配置 Ip 的區塊。  HYPER-V 主機並不會察覺的目前受指派的租約上 VNet，因此我們必須為避免在其中主機會指派一個 IP 已經存在的情況下配置的 Ip 區塊以只供 HYPER-V 主機。 這可讓我們可以避免重複的 IP 案例。
  * 我們選擇的 Ip 的區塊將會對應到的子網路中，在您的 HYPER-V 主機位於 VNet。
  * 我們希望這會對應到現有的子網路的原因是為了處理回透過 ExpressRoute BGP 廣告。 如果我們只是組成以使用 HYPER-V 主機的 IP 範圍，則我們會建立一系列的靜態路由，讓用戶端有內部部署與巢狀虛擬機器進行通訊。 這表示這不硬碟需求，因為您可能構成 IP 範圍的巢狀虛擬機器，然後建立直接存取該範圍的 HYPER-V 主機的用戶端所需的所有路由。
* 我們將會建立內部交換器 HYPER-V 內，然後我們將會指派新建立的介面的 IP 位址，我們將保留 dhcp 範圍內。 此 IP 位址會成為我們巢狀虛擬機器的預設閘道，且可用於內部交換器和主機連接到我們 VNet 的 NIC 之間的路線。
* 我們將會在主機上，這會將我們的主機，變成路由器安裝路由及遠端存取的角色。  這是必要允許外部主機的資源和我們的巢狀虛擬機器之間的通訊。
* 我們將會告訴其他資源如何存取這些巢狀虛擬機器。 這樣會使用我們建立使用者定義路由表包含的 IP 範圍中的巢狀虛擬機器的靜態路由。 這個靜態路由會指向 HYPER-V 的 IP 位址。
* 您則會放置這個 UDR 閘道子網路上，因此來自內部部署的用戶端知道如何到達我們巢狀虛擬機器。
* 您也會將此 UDR 放在需要能連線到巢狀虛擬機器的 Azure 內的任何其他子網路上。
* 針對多個 HYPER-V 主機，您會建立其他 「 浮動 」 的子網路，並將額外的靜態路由新增到 UDR。
* 當您解除委任 HYPER-V 主機您將會刪除/重新規劃我們 「 浮動 」 的子網路和移除該靜態路由從我們 UDR，或如果這是最後一個的 HYPER-V 主機，移除 UDR 完全。

## <a name="creating-our-virtual-switch"></a>建立我們的虛擬交換器

1. 在 \ [系統管理模式中開啟 PowerShell。
2. 建立內部交換器： `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. IP 指派新建立的介面： `New-NetIPAddress –IPAddress 10.0.1.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>安裝與設定 DHCP

*許多人會錯過這個元件時要先嘗試取得巢狀虛擬化工作。 不同於內部其中您客體虛擬機器將會接收 DHCP 所在網路，您的主機，在 Azure 中的巢狀虛擬機器必須提供 DHCP 透過它們所執行的主機。 或您需要將靜態 IP 位址指派給每個巢狀的 VM，不是可調整。*

1. 安裝 DHCP 角色： `Install-WindowsFeature DHCP -IncludeManagementTools`
2. 建立 DHCP 範圍： `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.1.2 -EndRange 10.0.1.254 -SubnetMask 255.255.255.0`
3. 設定範圍的 DNS 與預設閘道選項： `Set-DhcpServerV4OptionValue -DnsServer 10.0.0.10 -Router 10.0.1.1`
    * 請務必輸入有效的 DNS 伺服器。 在此情況下，我發生，向上 Windows DNS 服務 10.0.0.0/24 網路上有伺服器。

## <a name="installing-remote-access"></a>安裝遠端存取

* 開啟伺服器管理員並選取 「 新增角色及功能 」。
* 直到您取得 「 伺服器角色 」，請選取 「 下一步 」。
* 檢查 「 遠端存取 」，然後按一下 [下一步 」 直到您取得 」 角色服務 」。
* 檢查 「 路由 」，選取 [新增功能 \]，，然後選取 [下一頁]，然後再 「 安裝 」。 完成精靈，並等待完成安裝。

## <a name="configuring-remote-access"></a>設定遠端存取

* 開啟 \ [伺服器管理員並選取 「 工具 」，然後選取 「 路由及遠端存取 」。
* 路由及遠端存取管理面板的右邊上您會看到與您的伺服器名稱旁邊的圖示，以滑鼠右鍵按一下這並選取 「 設定和啟用路由及遠端存取 」。
* 在精靈選取 「 下一步 」、 檢查 」 兩個私人網路之間的安全連線 」，放射狀的按鈕，然後選取 「 下一步 」。
* 選取放射狀按鈕"No"詢問您是否要使用指定撥號連線，然後選取 [下一頁]，然後選取 [」 完成 」 時。

## <a name="creating-a-route-table-within-azure"></a>建立 Azure 內的路由表

請參閱[這篇文章](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)，以便更深入了解讀取上建立和管理 Azure 的路由。

* 瀏覽至https://portal.azure.com。
* 左上角中選取 [建立資源 \]。
* 在 [搜尋] 欄位中輸入 「 路由表 」，然後點擊輸入。
* 最佳的結果將會路由表，選取此選項，，然後選取 「 建立 」
* 路由表的名稱，請在這個案例中我將它命名為 「 路由-適用於-巢狀-Vm 」。
* 請確定您選取您的 HYPER-V 主機位於相同訂用帳戶。
* 建立新的資源群組或選取現有，並確定您建立的路由表中的區域是您的 HYPER-V 主機位於相同區域。
* 選取 [建立 \]。

## <a name="configuring-the-route-table"></a>設定路由表

* 瀏覽到我們剛建立的路由表。 您可以搜尋從入口網站的中央頂端搜尋列路由表的名稱。
* 選取之後路由表從瀏覽到 「 路由 」 刀鋒視窗內。
* 選取 「 新增 」。
* 為您的路線提供名稱，採用 「 巢狀虛擬機器 」。
* 地址的前置詞輸入我們 「 浮動 」 的子網路的 IP 範圍。 在此情況下，便會是 10.0.1.0/24。
* 「 下一個躍點類型 「 選取 」 虛擬設備 」，然後輸入 HYPER-V 主機，這可能會 10.0.0.4，而且然後選取 「 確定 」 的 IP 位址。
* 現在內刀鋒視窗中選取 「 子網路 」，這將能從 「 路由 」 的正下方。
* 選取 「 產生關聯 」，然後選取我們 」 Hub 「 虛擬網路，然後選取 「 GatewaySubnet 」，，然後選取 「 確定 」。
* 對我們的 HYPER-V 主機位於以及與其他需要存取巢狀虛擬機器的子網路的子網路中執行這個相同程序。

## <a name="conclusion"></a>總結

您現在應該能夠部署到 HYPER-V 主機的虛擬機器 (甚至是 32 位元 VM ！)，並讓它可以存取從內部部署和 Azure 內。