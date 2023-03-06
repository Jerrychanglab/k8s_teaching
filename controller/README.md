# Controller(控制器)使用說明
## Labels (標籤)
![image](https://user-images.githubusercontent.com/39659664/223018499-9863ebad-2239-424f-9ea1-62ff4983df51.png)
### 說明:定義每個物件的群組，如:Pod/Node，可快速收尋分類。
##### 新增標籤
    kubectl label <object> <pod name> <key>=<value>
##### 修改標籤
    kubectl label <object> <pod name> <key>=<value> --overwrite
##### 移除標籤
    kubectl label <object> <pod name> <key>-
##### 查詢標籤
    kubectl get <object> <pod name> --show-labels    
##### 篩選標籤
    kubectl get <object> --selector <key>=<value>
## Deployment (部署策略)
![image](https://user-images.githubusercontent.com/39659664/223021971-28a61756-c518-4702-acd1-10fa7ae686e7.png)
### 說明:自訂快速部署策略，並包括ReplicaSet(副本功能)。ReplicaSet功能是Pod的保證存活數量，如Pod異常時，會自動將Pod移除重新生成，滿足ReplicaSet的數值。
> 備註:Deployment / ReplicaSet / Pod都是有Lables的對應關係，如將其中一個Lables更改或移除，會導致異常。
##### 新增方式 (透過dry-run=Client)
    kubectl create deployment <deploymet name> --image=<套件>:<版本> --replicas=<副本數> --dry-run=client -o yaml > <deployment name>.yaml
    kubectl apply -f <deployment name>.yaml
##### 副本數調整
    kubectl scale deployment <deployment name> --replicas <數量>
##### Image升級
    kubectl set image deployment/<deployment name> <套件>=<套件>:<版本>
##### 退版方式 (下方流程步驟)
    kubectl rollout history deployment <deployment name> //查看版本歷程
    kubectl rollout history deployment <deployment name> --revision=<號碼> //查看要退版的版本號碼
    kubectl rollout undo deployment <deployment name> --to-revision=<號碼> //退版到要的版本號碼
> 備註:產生新的版本，並同步新增一個ReplicaSet，此ReplicaSet紀錄Pod的組態，如被移除，會無法退版。
##### 移除方式
    kubectl delete deployments.apps <deployment name>
## DaemonSet (進程守護)
![image](https://user-images.githubusercontent.com/39659664/223023844-79d31c33-fb8c-429d-a335-58c087171f9d.png)
### 說明:確保所有Node上都運行一個Pod，當有新的Node加入時，Pod也會自動生成在上面。
#### 新增方式
    kubectl create deployment <daemonset name> --image=<套件> --dry-run -o yaml | sed '/null\|{}\|replicas/d;/status/,$d;s/Deployment/DaemonSet/g' > <daemonset name>.yaml
    kubectl apply -f <daemonset name>.yaml
> 透過Deployment的dry-run來進行修改。
#### 查看方式
    kubectl get daemonsets -o wide
    kubectl get pod <pod name> -o wide //可查看是否pod都在每個Node上
