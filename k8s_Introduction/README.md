# K8s 元件說明
![image](https://user-images.githubusercontent.com/39659664/223376662-c5933a61-178e-42e6-aa49-99907c86ec92.png)
##  Master Node 
### [ Kube-apiserver ]
#### kube-apiserver是kubernetes集群的组件之一，用於處理API請求，並將集群狀態儲存到etcd中。在部署Pod時，用户通過kubectl命令或其他客户端向kube-apiserver發送創建Pod的API請求。
### [ etcd ]
#### etcd是kubernetes集群的可靠數據儲存，用於儲存集群的狀態信息，例如Pod、Service、ConfigMap、Secret等。當kube-apiserver收到用户創建Pod的請求時，會將Pod的配置信息儲存到 etcd中。
### [ Scheduler ]
#### scheduler是kubernetes集群的组件之一，用于將未指定Node的Pod調度到kubernetes集群中的Node上。當scheduler發現新的Pod创建請求時，會根據Pod的調度要求選擇最優的節點，並將該信息存储到etcd中。
### [ Controller Manager ]
#### controller manager是kubernetes集群的組件之一，用於運行一組控制器，以確保集群的狀態與期望狀態一致。當controller manager檢測到新的Pod創建請求時，會啟動相應的控制器，如:ReplicaSet、Deployment...等，以確保Pod被正確的創建與管理。
## Work Node
### [ kubelet ]
#### kubelet是kubernetes集群中每個Node的主要組件之一，負責管理該Node上的容器生命周期，確保Pod在節點上運行的狀態與期望狀態一致。當kubelet接收到從kube-apiserver發送的創建Pod 的請求時，會負責在該Node上創建與運行對應的Pod。
### [ kube Proxy ]
#### kube proxy是kubernetes集群中每個Node的重要的组件之一，其主要作用是實現Service的負載均衡和服務發現，並支持Kubernetes内部組件的訪問。通过監聽kube-apiserver中Service的變化化，並更新節點上的iptables規則，以保證請求能够正確轉發到Service後端的Pod上。
### [ Container Runtime ]
