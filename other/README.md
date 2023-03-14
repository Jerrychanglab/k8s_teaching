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
##### 步驟一:抓取HPA yaml
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml
> 部署metrics server會存在kube-system的namespaces內
> 必須要有兩個Node節點才能部署
##### 步驟二:修改yaml新增tls參數，不驗證CA證書
    kubectl edit deployments.apps -n kube-system metrics-server
![image](https://user-images.githubusercontent.com/39659664/224663352-adcc7054-66db-48a1-b847-93d9a191101e.png)
> 新增 --kubelet-insecure-tls
##### 步驟三: 重新啟動
    kubectl rollout restart deployment -n kube-system metrics-server
##### 步驟四: 驗證
    kubectl get deployments.apps -n kube-system metrics-server
![image](https://user-images.githubusercontent.com/39659664/224880162-902da9d7-efd4-4726-b7be-fa9a7a60efd1.png)

    kubectl get service -n kube-system metrics-server
![image](https://user-images.githubusercontent.com/39659664/224880505-b425e900-ddc9-4ee8-a78e-9b8faa4aa253.png)
> 確認是否READY
#### 使用率查看
##### Node
    kubectl top node
##### Pod
    kubectl top pods
## [ Pod AutoScaling ]
![image](https://user-images.githubusercontent.com/39659664/224660612-84ec2739-4c5a-4a15-96fb-0c31ec69250d.png)
### 說明: 透過AutoScaling有效的控管Pod使用數量，讓資源妥善運用，並且當規則配置完成後，當遇到大量需求量時，會透過閥值偵測(Metrice Server)，自動擴充Pod的數量，後續需求量減少，會自己將Pod回收。
> 備註:HPA機制，仰賴Metrice Server，如無此服務，會倒置HPA功能失效。
#### HPA建置
##### 步驟一: 透過Deployment創建服務(定義cpu/memory的requests/limits)
> 開啟PodAutoScaling.yaml，進行修改。(注意:limits數值不能小於requests)

![image](https://user-images.githubusercontent.com/39659664/224886976-160e13c8-a673-404b-8941-6ce0564a91d5.png)

    kubectl apply -f PodAutoScaling.yaml
##### 步驟二: 建置Service，讓Pod能進入負載平衡。
    kubectl apply -f ServiceAutoScaling.yaml
##### 步驟三: 建置AutoScaling
> 開啟AutoScaling.yaml進行修改
