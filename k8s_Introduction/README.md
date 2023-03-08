# K8s 元件說明
![image](https://user-images.githubusercontent.com/39659664/223376662-c5933a61-178e-42e6-aa49-99907c86ec92.png)
##  Master Node 
### [ Kube-apiserver ]
#### kube-apiserver是kubernetes集群的控制组件之一，用於處理API請求，並將集群狀態儲存到etcd中。在部署Pod時，用户通過kubectl命令或其他客户端向kube-apiserver發送創建Pod的API請求。
### [ etcd ]
#### etcd是kubernetes集群的可靠數據儲存，用於儲存集群的狀態信息，例如Pod、Service、ConfigMap、Secret等。當kube-apiserver收到用户創建Pod的請求時，會將Pod的配置信息儲存到 etcd中。
### [ Scheduler ]
#### scheduler是kubernetes集群的控制平面组件之一，用于將未指定Node的Pod調度到kubernetes集群中的Node上。當scheduler發現新的Pod创建請求時，會根據Pod的調度要求選擇最優的節點，並將該信息存储到etcd中。
### [ Controller Manager ]
## Work Node
### [ kubelet ]
#### kubelet是kubernetes的一個重要組件，負責管理每個Node上的生命週期，確保Pod在Node上的運行狀態保持一致，當kubelet接收到kube-apiserver發送的建置Pod請求時，會負責建置Pod。
### [ kube Proxy ]
### [ Container Runtime ]
