# Kubernetes(k8s)初階課程
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
### K8S Introduction
* API Server
* etcd
* Scheduler
* Controller Manager
* kubelet
* kube Proxy
* Container Runtime
### Pod基礎介紹
* Pod是什麼
* Pod與Container的關係
* Sidecar是什麼
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
* ConfigMap(環境變數-非機密)
* Secret(環境變數-機密)
### Service (服務負載平衡) 
* Cluster IP
* NodePort
* LoadBalancer
### Ingress
* Nginx Ingress
### HorizontalScaling(水平縮放)
* 資源概念(CPU/RAM)
* Metrics採集
* Pod AutoScaling (HPA)
