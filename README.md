# Kubernetes教學
## 大綱
### Kubernetes Install (安裝)
* Linux準備3台
> 此安裝使用Centos 8
* Node環境安裝與配置
> 配置完後，在指定一台做Master Node
* Master Node Assign
* Work Node Join
* NetWork Overlay部署
* 驗證
### K8S元件說明
![image](https://user-images.githubusercontent.com/39659664/223367119-31500a4d-eb9e-45cb-9f45-43941792d6eb.png)
* api server
* etcd
* scheduler
* controller manager
* kubelet
* kube Proxy
* Container Runtime
### Controller (控制)
* Labels(標籤)
* Deployment(部署策略)
* DaemonSet(進程守護)
* Job(一次性任務)
* CronJob(排程任務)
### Scheduling (調度)
* NodeSelector(節點選擇)
* Taints(污點)
* Tolerations(反污點)
* Maintain(維護)
### Storage (儲存)
* EmptyDir(暫時儲存空間)
* HostPath(本機掛載空間)
* PV + PVC(NFS掛載)
### Network (網路)
