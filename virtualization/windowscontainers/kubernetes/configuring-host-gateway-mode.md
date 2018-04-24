# <a name="host-gateway-mode"></a>主機閘道模式 #
Kubernetes 網路功能的其中一個可用選項是*主機閘道模式*，這涉及所有節點上 Pod 子網路之間靜態路徑的設定。


## <a name="configuring-static-routes--linux"></a>設定靜態路由 | Linux ##
針對這點，我們會使用 `iptables`。 將 `$CLUSTER_PREFIX` 變數取代 (或設定) 為所有 Pod 都將使用的簡寫式子網路：

```bash
CLUSTER_PREFIX="192.168"
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING ! -d $CLUSTER_PREFIX.0.0/16 \
              -m addrtype ! --dst-type LOCAL -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1
```

這只會為 Pod 設定基本 NATing。 現在，我們需要讓前往 Pod 的所有流量都通過主要介面。 如有需要，再次取代 `$CLUSTER_PREFIX` 變數和 `eth0` (如果適用)：

```bash
sudo route add -net $CLUSTER_PREFIX.0.0 netmask 255.255.0.0 dev eth0
```

最後，我們需要在**每個節點**新增下一個躍點閘道。 例如，如果第一個節點是 `192.168.1.0/16` 上的 Windows 節點，則：

```bash
sudo route add -net $CLUSTER_PREFIX.1.0 netmask 255.255.255.0 gw $CLUSTER_PREFIX.1.2 dev eth0
```

*針對*叢集中的每個節點，*在*叢集中的每個節點，必須新增類似路由。


<a name="explanation-2-suffix"></a>
> [!Important]  
> **只**適用於 Windows 節點，閘道是子網路的 `.2`。 對於 Linux，這可能一律是 `.1`。 此異常是因為 `.1` 位址已保留供橋接主機網路和虛擬 Pod 網路之網路介面卡的閘道使用。


## <a name="configuring-static-routes--windows"></a>設定靜態路由 | Windows ##
針對這點，我們會使用 `New-NetRoute`。 [這個存放庫](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/AddRoutes.ps1)中有可用的自動指令碼 `AddRoutes.ps1`。 您必須知道 *Linux 主機*的 IP 位址：

```powershell
$url = "https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/windows/AddRoutes.ps1"
Invoke-WebRequest $url -o AddRoutes.ps1
./AddRoutes.ps1 -MasterIp 10.1.2.3
```
