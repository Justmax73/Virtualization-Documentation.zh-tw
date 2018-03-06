---
title: "從頭建立 Kubernetes 主機"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "從頭開始建立 Kubernetes 叢集主機。"
keywords: "kubernetes, 1.9, 主機, linux"
ms.openlocfilehash: 3ea338f7af3dd921731fce0ec5a8b2cf8c4fef0c
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/01/2018
---
# <a name="kubernetes-master--from-scratch"></a>從頭建立 Kubernetes 主機 #
此頁面從開始到結束逐步解說 Kubernetes 主機的手動部署。

需要有最近更新、類似 Ubuntu 的 Linux 電腦，才能進行示範。 Windows 不在此情境中；二進位檔案是從 Linux 交叉編譯的。


> [!Warning]  
> 因為 Kubernetes 各版本間的波動性，此指南中所做的假設在未來可能不成立。


## <a name="preparing-the-master"></a>準備主機 ##
首先，安裝所有的必要條件：

```bash
sudo apt-get install curl git build-essential docker.io conntrack python2.7
```

如果您位於 Proxy 伺服器後方，必須為目前工作階段定義環境變數：
```bash
HTTP_PROXY=http://proxy.example.com:80/
HTTPS_PROXY=http://proxy.example.com:443/
http_proxy=http://proxy.example.com:80/
https_proxy=http://proxy.example.com:443/
```
或如果您想要讓此設定具有永久性，請將變數新增至 /etc/environment (登出然後再次登入，才能套用變更)。

[這個存放庫](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux)上的指令碼集合，有助於設定程序。 將這些指令碼簽出至 `~/kube/`；在未來步驟中，這整個目錄將裝載至許多 Docker 容器，因此請將其結構保持和本指南中所述的相同。

```bash
mkdir ~/kube
mkdir ~/kube/bin
git clone https://github.com/Microsoft/SDN /tmp/k8s 
cd /tmp/k8s/Kubernetes/linux
chmod -R +x *.sh
chmod +x manifest/generate.py
mv * ~/kube/
```


### <a name="installing-the-linux-binaries"></a>安裝 Linux 二進位檔 ###

> [!Note]  
> 若要包含修補程式或使用 Bleeding-Edge Kubernetes 程式碼，而不下載預先建置的二進位檔，請參閱[此頁面](./compiling-kubernetes-binaries.md)。

從 [Kubernetes Mainline](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1) 下載並安裝官方 Linux 二進位檔案，並進行安裝，如下所示：

```bash
wget -O kubernetes.tar.gz https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
tar -vxzf kubernetes.tar.gz 
cd kubernetes/cluster 
# follow the prompts from this command, the defaults are generally fine:
./get-kube-binaries.sh
cd ../server
tar -vxzf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin
cp hyperkube kubectl ~/kube/bin/
```

將二進位檔案新增至 `$PATH`，以便我們可以從任何地方執行它們。 請注意，此僅會針對此工作階段設定路徑；將這一行新增至 `~/.profile` 以進行永久設定。

```bash
$ PATH="$HOME/kube/bin:$PATH"
```

### <a name="install-cni-plugins"></a>安裝 CNI 外掛程式 ###
Kubernetes 網路需要基本 CNI 外掛程式。 可從[這裡](https://github.com/containernetworking/plugins/releases)下載這些外掛程式並解壓縮至 `/opt/cni/bin/`：

```bash
DOWNLOAD_DIR="${HOME}/kube/cni-plugins"
CNI_BIN="/opt/cni/bin/"
mkdir ${DOWNLOAD_DIR}
cd $DOWNLOAD_DIR
curl -L $(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep browser_download_url | grep 'amd64.*tgz' | head -n 1 | cut -d '"' -f 4) -o cni-plugins-amd64.tgz
tar -xvzf cni-plugins-amd64.tgz
sudo mkdir -p ${CNI_BIN}
sudo cp -r !(*.tgz) ${CNI_BIN}
ls ${CNI_BIN}
```


### <a name="certificates"></a>憑證 ###
首先，取得本機 IP 位址，請透過 `ifconfig` 或：

```bash
$ ip addr show dev eth0
```

如果介面名稱是已知的。 在此程序中將會經常參考此位址，將其設定為環境變數會輕鬆許多。 下列程式碼片段暫時設定它；如果工作階段結束或殼層關閉時，需要再次設定。

```bash
$ MASTER_IP=10.123.45.67   # example! replace
```

準備憑證供叢集節點通訊之用：

```bash
cd ~/kube/certs
chmod u+x generate-certs.sh
./generate-certs.sh $MASTER_IP
```

### <a name="prepare-manifests--addons"></a>準備資訊清單與附加元件 ###
產生一組指定 Kubernetes 系統 Pod 的 YAML 檔案，方式是將主機 IP 位址和*完整*的叢集 CIDR 傳遞給 `manifest` 資料夾中的 Python 指令碼：

```bash
cd ~/kube/manifest
./generate.py $MASTER_IP --cluster-cidr 192.168.0.0/16
```

移除/移動 Python 指令碼，以便 Kubernetes 不將它誤認為資訊清單。如果未執行此動作，稍後這會造成問題。

> [!Important]  
> 如果 Kubernetes 版本與本指南有分歧，請在指令碼上使用各種不同的版本旗標 (例如 `--api-version`) 以[自訂 Pod 部署的映像](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/hyperkube-amd64)。 並非所有的資訊清單都使用相同的映像，而且有不同的版本結構描述 (尤其是，`etcd` 和附加元件管理員)。


#### <a name="manifest-customization"></a>資訊清單自訂 ####
此時，可能需要安裝特定的變更。 例如，可能有需要將子網路手動指派至節點，而不任由 Kubernetes 自動管理。 此特定組態在指令碼中有個選項 (請參閱 `--help` 以取得 `--im-sure` 參數的說明)：

```bash
./generate.py $MASTER_IP --im-sure
```

任何其他自訂的組態選項，都將會需要手動修改產生的資訊清單。


### <a name="configure--run-kubernetes"></a>設定及執行 Kubernetes ###
設定 Kubernetes 以使用產生的憑證。 這將會在 `~/.kube/config` 建立組態：

```bash
cd ~/kube
./configure-kubectl.sh $MASTER_IP
```

現在，將檔案複製到 Pod 稍後所預期的位置：

```bash
mkdir ~/kube/kubelet
sudo cp ~/.kube/config ~/kube/kubelet/
```

Kubernetes「用戶端」`kubelet` 已準備啟動。 下列兩個指令碼都會無限期地執行；在每個之後開啟另一個終端工作階段以持續運作：

```bash
cd ~/kube
sudo ./start-kubelet.sh
```

執行 Kubeproxy 指令碼，傳遞部分叢集 CIDR：

```bash
cd ~/kube
sudo ./start-kubeproxy.sh 192.168
```


> [!Important]  
> 這將是*完整*預期的 /16 CIDR，其下的節點將會失敗，*即使該 CIDR 上有非 Kubernetes 流量也一樣*。 Kubeproxy *只*適用於前往*服務*子網路的 Kubernetes 流量，所以它應該不會影響其他主機的流量。

> [!Note]  
> 這些指令碼可以精靈化。 本指南只涵蓋手動執行這些指令碼，因為這在設定期間攔截錯誤時最實用。


## <a name="verifying-the-master"></a>驗證主機 ##
幾分鐘之後，系統應該處於下列狀態：

  - 在 `docker ps` 下，將會有 ~23 個背景工作角色和 Pod 容器。
  - 呼叫 `kubectl cluster-info`，將會顯示 Kubernetes 主機 API 伺服器以及 DNS 及 Heapster 附加元件的相關資訊。
  - `ifconfig` 將會顯示新的介面 `cbr0` 與選擇的叢集 CIDR。

