# Kubernetes資源
* 資源概念(CPU/RAM)
* Metrics採集
* Pod AutoScaling
## [ 資源概論(CPU/RAM) ]
![image](https://user-images.githubusercontent.com/39659664/224649798-1442e483-7f14-49a0-8692-be66d845cd00.png)
### CPU說明: Kubernetes中的CPU資源是用核心的單位來計算的，每顆CPU會被切割1000個mCPU，因此在Pod內的Containers要分配資源，需設定兩個欄位，requests和limit
### Memory說明: 與傳統記憶體認知無差別，配置資源時可用(k、Ki、M、Mi、G、Gi)等單位，也是要配置requests和limits欄位。
#### Requests說明: Requests指的是Containers最小資源量，當Pod被部署時，會檢查Node(s)是否有足夠的資源能取得，如不滿足時會處於Pending狀態。
#### Limits說明: Limits指的是，Containers最大資源量，如Pod被大量呼叫時，不會超過設定的Limit，確保Node(s)的穩定。
##### 查看Node CPU/Memory資源
    kubectl describe nodes <Node Name> | grep -A11 Capacity
> 查看cpu欄位，如顯示4就代表您有4000個mCPU
## [ Metrics採集 ]
### 說明: K8s環境建置後，想暸解每個Node或pod資源使用率多少，必須安裝採集器，來收集此資訊。
#### Metrics Server建置
##### 步驟一:抓取yaml
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
> 部署metrics server會存在kube-system的namespaces內
##### 步驟二:修改yaml新增tls by pass參數
    kubectl edit deployments.apps -n kube-system metrics-server
![image](https://user-images.githubusercontent.com/39659664/224663352-adcc7054-66db-48a1-b847-93d9a191101e.png)
> 新增 --kubelet-insecure-tls
##### 步驟三: 驗證
    kubectl get deployments.apps -n kube-system metrics-server
![image](https://user-images.githubusercontent.com/39659664/224665264-505165bf-61a1-4b4e-9122-39fa88ed98c1.png)
> 確認是否READY
#### 使用率查看
##### Node
    kubectl top node
##### Pod
    kubectl top pods
## [ Pod AutoScaling ]
![image](https://user-images.githubusercontent.com/39659664/224660612-84ec2739-4c5a-4a15-96fb-0c31ec69250d.png)
### 說明: 透過AutoScaling有效的控管Pod使用數量，讓資源妥善運用，並且當規則配置完成後，當遇到大量需求量時，會透過閥值偵測，自動擴充Pod的數量，後續需求量減少，會自己將Pod回收。
> 備註:必須要在Deployment佈建出來的Pod才能採用此功能。
