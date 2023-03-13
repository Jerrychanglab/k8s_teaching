# Kubernetes資源
* 資源概念(CPU/RAM)
* Metrics採集
* Pod AutoScaling
## 資源概論(CPU/RAM)
![image](https://user-images.githubusercontent.com/39659664/224649798-1442e483-7f14-49a0-8692-be66d845cd00.png)
### CPU說明: Kubernetes中的CPU資源是用核心的單位來計算的，每顆CPU會被切割1000個mCPU，因此在Pod內的Containers要分配資源，需設定兩個欄位，requests和limit
### Memory說明: 與傳統記憶體認知無差別，配置資源時可用(k、Ki、M、Mi、G、Gi)等單位，也是要配置requests和limits欄位。
#### Requests說明: Requests指的是Containers最小資源量，當Pod被部署時，會檢查Node(s)是否有足夠的資源能取得，如不滿足時會處於Pending狀態。
#### Limits說明: Limits指的是，Containers最大資源量，如Pod被大量呼叫時，不會超過設定的Limit，確保Node(s)的穩定。
##### 查看Node CPU/Memory資源
    kubectl describe nodes <Node Name> | grep -A11 Capacity
> 查看cpu欄位，如顯示4就代表您有4000個mCPU
## Pod AutoScaling
![image](https://user-images.githubusercontent.com/39659664/224660612-84ec2739-4c5a-4a15-96fb-0c31ec69250d.png)
### 說明: 透過AutoScaling有效的控管Pod使用數量，讓資源妥善運用，並且當規則配置完成後，當遇到大量需求量時，會透過閥值偵測，自動擴充Pod的數量，後續需求量減少，會自己將Pod回收。
> 備註:必須要在Deployment佈建出來的Pod才能採用此功能。
