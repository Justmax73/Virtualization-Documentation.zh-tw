# [Windows 上的容器文件](index.md) 

# 概觀
## [關於 Windows 容器](about/index.md)
## [容器與VM](about/containers-vs-vm.md)
## [系統需求](deploy-containers/system-requirements.md)
## [常見問題集](about/faq.md)

# 開始使用
## [設定您的環境](quick-start/set-up-environment.md)
## [執行您的第一個容器](quick-start/run-your-first-container.md)
## [容器化範例應用程式](quick-start/building-sample-app.md)

# 教學課程
## 建立 Windows 容器
### [撰寫 Dockerfile](manage-docker/manage-windows-dockerfile.md)
### [最佳化 Dockerfile](manage-docker/optimize-windows-dockerfile.md)
## 在 Azure Kubernetes Service 上執行
### [在 AKS 上建立 Windows 容器叢集](/azure/aks/windows-container-cli)
### [目前的限制](/azure/aks/windows-node-limitations)
## 在 Service Fabric 上執行
### [部署您的第一個容器](/azure/service-fabric/service-fabric-quickstart-containers)
### [在 Windows 容器中部署 .NET 應用程式](/azure/service-fabric/service-fabric-host-app-in-a-container)
## 在 Azure App Service 上執行
### [Azure App Service 快速入門](/azure/app-service/app-service-web-get-started-windows-container)
### [遷移具有 Windows 容器和 Azure App Service 的 ASP.NET 應用程式](/azure/app-service/app-service-web-tutorial-windows-containers-custom-fonts)
## Windows 上的 Linux 容器
### [概觀](deploy-containers/linux-containers.md)
### [執行您的第一個 LCOW 容器](quick-start/quick-start-windows-10-linux.md)
## 透過 Windows 測試人員計畫使用容器
### [概觀](deploy-containers/insider-overview.md)

# 概念
## Windows 容器基本資訊
### [容器基底映像](manage-containers/container-base-images.md)
### [隔離模式](manage-containers/hyperv-container.md)
### [版本相容性](deploy-containers/version-compatibility.md)
### [資源控制項](manage-containers/resource-controls.md)
## Docker
### [Windows 上的 Docker 引擎](manage-docker/configure-docker-daemon.md)
### [Windows Docker 主機的遠端管理](management/manage_remotehost.md)
## 容器協調流程
### [概觀](about/overview-container-orchestrators.md)
### Windows 上的 Kubernetes
#### [Windows 上的 Kubernetes](kubernetes/getting-started-kubernetes-windows.md)
#### [建立 Kubernetes 主檔](kubernetes/creating-a-linux-master.md)
#### [選擇網路解決方案](kubernetes/network-topologies.md)
#### [加入 Windows 背景工作角色](kubernetes/joining-windows-workers.md)
#### [加入 Linux 背景工作角色](kubernetes/joining-linux-workers.md)
#### [部署 Kubernetes 資源](kubernetes/deploying-resources.md)
#### [疑難排解](kubernetes/common-problems.md)
#### [Windows 服務形式的 Kubernetes](kubernetes/kube-windows-services.md)
#### [編譯 Kubernetes 二進位檔](kubernetes/compiling-kubernetes-binaries.md)
### Service Fabric
#### [Service Fabric 和容器](/azure/service-fabric/service-fabric-containers-overview)
#### [資源管理](/azure/service-fabric/service-fabric-resource-governance)
### Docker Swarm
#### [Swarm 模式](manage-containers/swarm-mode.md)
## 工作負載
### 群組受管理的服務帳戶
#### [建立群組 gMSA](manage-containers/manage-serviceaccounts.md)
#### [設定應用程式以使用 gMSA](manage-containers/gmsa-configure-app.md)
#### [使用 gMSA 執行容器](manage-containers/gmsa-run-container.md)
#### [使用 gMSA 協調容器](manage-containers/gmsa-orchestrate-containers.md)
#### [針對 gMSA 進行疑難排解](manage-containers/gmsa-troubleshooting.md)
### [印表機服務](deploy-containers/print-spooler.md)
## 網路功能
### [概觀](container-networking/architecture.md)
### [網路拓撲和驅動程式](container-networking/network-drivers-topologies.md)
### [網路隔離和安全性](container-networking/network-isolation-security.md)
### [設定進階網路選項](container-networking/advanced.md)
## 儲存體
### [概觀](manage-containers/container-storage.md)
### [永續性儲存體](manage-containers/persistent-storage.md)
## 裝置
### [硬體裝置](deploy-containers/hardware-devices-in-containers.md)
### [GPU 加速](deploy-containers/gpu-acceleration.md)

# 參考
## [基底映像服務生命週期](deploy-containers/base-image-lifecycle.md)
## [防毒程式最佳化](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [容器平台工具](deploy-containers/containerd.md)
## [容器 OS 映像 EULA](Images_EULA.md)

# 資源
## [容器範例](samples.md)
## [疑難排解](troubleshooting.md)
## [容器論壇](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [社群影片與部落格](communitylinks.md)
