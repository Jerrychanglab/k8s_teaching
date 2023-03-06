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
### 說明:自訂快速部署策略，並包括ReplicaSet(副本功能)。ReplicaSet功能是Pod的存活數量。
##### 新增方式 (透過dry-run=Client)
    kubectl create deployment <deploymet name> --image=<套件>:<版本> --replicas=<副本數> --dry-run=client -o yaml > <deployment name>.yaml
    kubectl apply -f <deployment name>.yaml
##### 副本數調整
    kubectl scale deployment <deployment name> --replicas <數量>
##### Image升級
    kubectl set image deployment/<deployment name> <套件>=<套件>:<版本>
##### 退版方式 (下方流程步驟)
    kubectl rollout history deployment <deployment name> //查看版本歷程
    kubectl rollout history deployment <deployment name> --revision=1 //查看要退版的版本號碼
    kubectl rollout undo deployment <deployment name> --to-revision=1 //退版到要的版本號碼
> 備註:產生新的版本，在同步新增一個ReplicaSet，此ReplicaSet紀錄Pod的組態，如被移除，會無法退版。
##### 移除方式
    kubectl delete deployments.apps <deployment name>
