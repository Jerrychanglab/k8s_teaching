# K8s 元件說明
![image](https://user-images.githubusercontent.com/39659664/223376662-c5933a61-178e-42e6-aa49-99907c86ec92.png)
## [ Master Node ]
### Kube-apiserver
#### Kube-apiserver是kubernetes控制面的核心組件之一，作用是提供API服務，讓其他使用者或K8s組件(如:kubelet、kube-proxy、controller...等)，可通過Restful API與k8s集群進行互動。
#### 主要用途
* 提供API: 提供Restful API，讓用戶可使用Kubectl、Dashboard來管理集群的應用程序與服務對像等等。
* 權限驗證: 權限驗證，確保授權只有在此用戶能進行訪問操作集群。
* 管理etcd儲存: 與etcd儲存通信，將所有k8s集群訊息的狀態訊息儲存在etcd中，並提供對etcd儲存的管理監控功能。
* 控制器管理: 提供內置的控制器(如:ReplicaSet、Deploument...等)來管理應用程序狀態，並按照用戶的配置維持期望狀態。
* 擴展性和插件: 提供擴展API插件機制，可以方便位k8s添加自訂義的功能。
### etcd
#### etcd在kubernetes是用於儲存集群整個狀態，包括運行中的節點、容器、服務、儲存...等)，每當集群狀態發生變化時，etcd會自動更新訊息，並將訊息發送到所有的節點。
### Scheduler
### Controller Manager
## [ Work Node ]
### kubelet
### kube Proxy
### Container Runtime
