---
author: neilpeterson
---

# Windows 容器需求

**這是初版內容，後續可能會變更。**

本指南列出 Windows 容器主機的需求。

## 實體系統上的 Windows 容器

- Windows 容器角色只在 Windows Server 2016 TP4 (完整版及核心版) 和 Nano Server 上供使用。
- 必須安裝 Hyper-V 角色才可執行 Hyper-V 容器。

## 虛擬系統上的 Windows 容器

如果 Windows 容器主機將會在 Hyper-V 虛擬機器上執行，而且也會主控 Hyper-V 容器，就必須啟用巢狀虛擬化。 巢狀的虛擬化的需求如下：

- 至少有 4 GB RAM 可供虛擬化的 HYPER-V 主機使用。
- Windows Server 2016 Technical Preview 4 或主機系統上的 Windows 10 組建 10565 ，以及 Windows Server Technical Preview 4 (完整版、核心版) 或是虛擬機器上的 Nano Server。
- Intel VT-x 的處理器 (這項功能目前只適用於 Intel 處理器)。
- 容器主機 VM 也將會需要至少 2 個虛擬處理器。


## 支援的作業系統映像

Windows Server Technical Preview 4 提供兩種容器作業系統映像 (Windows Server Core 和 Nano Server)。 並非所有的設定都支援這兩種作業系統映像。 此表詳加說明所支援的設定。

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>主機作業系統</center></th>
<th><center>Windows Server 容器</center></th>
<th><center>Hyper-V 容器</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 完整 UI</center></td>
<td><center>Core 作業系統映像</center></td>
<td><center>Nano 作業系統映像</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Core 作業系統映像</center></td>
<td><center> Nano 作業系統映像</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Nano 作業系統映像</center></td>
<td><center>Nano 作業系統映像</center></td>
</tr>
</tbody>
</table>






<!--HONumber=Mar16_HO1-->


