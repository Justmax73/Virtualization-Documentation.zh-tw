---
title: 編譯 Kubernetes 二進位檔
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: 從原始碼編譯和交叉編譯 Kubernetes 二進位檔案。
keywords: kubernetes，1.12，linux，編譯
ms.openlocfilehash: 40bf7e65a8910cdab095abb269aa0a92508189cd
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909868"
---
# <a name="compiling-kubernetes-binaries"></a>編譯 Kubernetes 二進位檔 #
編譯 Kubernetes 需要正常運作的 Go 環境。 此頁面討論編譯 Linux 二進位檔和交叉編譯 Windows 二進位檔的幾個方式。
> [!NOTE] 
> 此頁面完全自願，而且只包含給想要試驗最新 & 最佳原始程式碼的感興趣 Kubernetes 開發人員使用。

> [!tip]
> 若要接收最新開發的通知，您可以訂閱[@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce)。

## <a name="installing-go"></a>安裝 Go ##
為了簡單起見，僅討論在暫時、自訂位置中安裝 Go：

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.11.1.linux-amd64.tar.gz -O go1.11.1.tar.gz
tar -vxzf go1.11.1.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!Note]  
> 這些設定工作階段的環境變數。 將 `export` 新增至 `~/.profile` 以進行永久設定。

執行 `go env`，確認已正確設定路徑。 建置 Kubernetes 二進位檔有數個選項：

  - [在本機](#build-locally)建置。
  - 使用 [Vagrant](#build-with-vagrant) 產生二進位檔。
  - 使用 Kubernetes 專案中的[標準容器化組建指令碼](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts)。 針對這點，請依照[在本機建置](#build-locally)的步驟直到 `make` 步驟，然後使用連結的指示。

若要將 Windows 二進位檔複製到各自節點，請使用視覺工具例如 [WinSCP](https://winscp.net/eng/download.php) 或命令列工具例如 [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)，以將其傳輸至 `C:\k` 目錄。


## <a name="building-locally"></a>在本機建立 ##
> [!Tip]  
> 如果您遇到「權限遭拒」錯誤時，可藉由先建置 Linux `kubelet` 避免這些錯誤，如 [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176) 中的注意事項所述：
>  
> _由於 Kubernetes Windows 組建系統中似乎是 bug，因此必須先建立 Linux 二進位檔以產生 `_output/bin/deepcopy-gen`。以 Windows 為基礎，這麼做會產生空的 `deepcopy-gen`。_

首先，擷取 Kubernetes 存放庫：

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about 
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

現在，簽出要建置的分支並建置 Linux `kubelet` 二進位檔。 這是為了避免上文所述的 Windows 建置錯誤。 在此，我們將使用 `v1.12.2`。 在 `git checkout` 之後，您可以套用擱置中 PR、補充程式，或修改自訂二進位檔。

```bash
git checkout tags/v1.12.2
make clean && make WHAT=cmd/kubelet
```

最後，建置所需的 Windows 用戶端二進位檔 (根據稍後應該從何處擷取 Windows 二進位檔，最後一個步驟可能有所不同)：

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

建置 Linux 二進位檔的步驟相同；只要移除命令的 `KUBE_BUILD_PLATFORMS=windows/amd64` 前置詞。 輸出目錄將會是 `_output/.../linux/amd64`。


## <a name="build-with-vagrant"></a>使用 Vagrant 建置 ##
[這裡](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant)有 Vagrant 設定。 使用它來準備 Vagrant VM，然後在其內部執行下列命令：

```bash
DIST_DIR="${HOME}/kube/"
SRC_DIR="${HOME}/src/k8s-main/"
mkdir ${DIST_DIR}
mkdir -p "${SRC_DIR}"

git clone https://github.com/kubernetes/kubernetes.git ${SRC_DIR}

cd ${SRC_DIR}
git checkout tags/v1.12.2
KUBE_BUILD_PLATFORMS=linux/amd64   build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kubelet 
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kube-proxy 
cp _output/dockerized/bin/windows/amd64/kube*.exe ${DIST_DIR}

ls ${DIST_DIR}
```

