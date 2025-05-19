# K8s 元件說明
![image](https://github.com/user-attachments/assets/0e975b65-0ea9-4502-9aa6-24b39d02246e)

##  Master Node
### [ Kube-apiserver ]
* Kubernetes 的核心 API 入口，負責接收、驗證並處理來自用戶或元件的 API 請求。
* 所有元件透過它進行溝通。
* 寫入或查詢狀態資訊時，會與 etcd 交互。
### [ etcd ]
* 分散式 Key-Value 儲存系統，用來保存整個集群狀態（如 Pod、Service、ConfigMap 等）。
* 被視為 Kubernetes 的「資料庫」。
### [ Scheduler ]
* 將未指定 Node 的 Pod，根據資源條件與策略排程至合適的 Node。
* 排程結果會寫入 etcd，由 kubelet 執行。
### [ Controller Manager ]
* 負責執行多種控制器（如 ReplicaSet、Deployment 等），持續監控資源狀態。
* 自動調整實際狀態，使其符合期望狀態。
## Work Node
### [ Kubelet ]
* 每個 Node 上的代理，負責與 API Server 溝通。
* 接收 Pod 部署指令後，調用 Container Runtime 啟動容器，並回報狀態。
### [ Kube Proxy ]
* 實現服務的「負載平衡」與「網路轉發」。
* 動態維護 iptables 規則，將流量導向對應的 Pod。
### [ CRI]
* 一個標準化介面，用來讓kubelet可以與不同的容器執行環境（Container Runtime）進行溝通。
### [ Container Runtime ]
* 負責實際啟動與管理容器。
* 如 containerd 、 CRI-O ... 等
