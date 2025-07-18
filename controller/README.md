# Controller(控制器)使用說明
**章節**
* Labels(標籤)
* Deployment(部署策略)
* DaemonSet(進程守護)
* Job(一次性任務)
* CronJob(排程任務)
## [ Labels ] (標籤)
![image](https://user-images.githubusercontent.com/39659664/223018499-9863ebad-2239-424f-9ea1-62ff4983df51.png)
### 說明:Kubernetes 中的 Label（標籤） 是一組 key=value 格式的附加資訊，可以附加在 Pod、Node、Service 等資源上，用來分類與選取資源群組。
##### 新增標籤
```bash
kubectl label <object> <pod name> <key>=<value>
```
##### 修改標籤
```bash
kubectl label <object> <pod name> <key>=<value> --overwrite
```
##### 移除標籤
```bash
kubectl label <object> <pod name> <key>-
```
##### 查詢標籤
```bash
kubectl get <object> <pod name> --show-labels
```
##### 篩選標籤
```bash
kubectl get <object> --selector <key>=<value>
```
## [ Deployment ] (部署策略)
![image](https://user-images.githubusercontent.com/39659664/223021971-28a61756-c518-4702-acd1-10fa7ae686e7.png)
### 說明:自訂快速部署策略，並包括ReplicaSet(副本功能)。ReplicaSet功能是Pod的保證存活數量，如Pod異常時，會自動將Pod移除重新生成，滿足ReplicaSet的數值。
##### Yaml格式講解
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment             # Deployment 名稱（會影響產生的 Pod 名稱）
spec:
  replicas: 2                        # 想要運行的 Pod 數量
  selector:
    matchLabels:  # 與下方 template.metadata.labels 必須一致
      app: nginx
      env: prod
      dept: it-cni
  template:
    metadata: 
      labels: # 與上方 matchLables 必須一致
        app: nginx
        env: prod
        dept: it-cni
    spec:
      containers:
      - name: nginx                  # 容器名稱
        image: nginx:1.14.2          # 容器鏡像
        ports:
        - containerPort: 80          # 容器開放 Port
        resources:
          requests:
            cpu: "500m"              # 最少保留 0.5 顆 CPU
            memory: "128Mi"          # 最少保留 128MiB 記憶體
          limits:
            cpu: "1000m"             # 最多使用 1 顆 CPU
            memory: "256Mi"          # 最多使用 256MiB 記憶體
```
> 備註:Deployment / ReplicaSet / Pod都是有Lables的對應關係，如將其中一個Lables更改或移除，會導致異常。
##### 新增方式 (透過dry-run=Client)
```bash
kubectl create deployment <deploymet name> --image=<套件>:<版本> --replicas=<副本數> --dry-run=client -o yaml > <deployment name>.yaml
kubectl apply -f <deployment name>.yaml
```
##### 副本數調整
```bash
kubectl scale deployment <deployment name> --replicas <數量>
```
##### Image升級
```bash
kubectl set image deployment/<deployment name> <套件>=<套件>:<版本>
```
##### 退版方式 (下方流程步驟)
```bash
kubectl rollout history deployment <deployment name> //查看版本歷程
kubectl rollout history deployment <deployment name> --revision=<號碼> //查看要退版的版本號碼
kubectl rollout undo deployment <deployment name> --to-revision=<號碼> //退版到要的版本號碼
```
> 備註:產生新的版本，並同步新增一個ReplicaSet，此ReplicaSet紀錄Pod的組態，如被移除，會無法退版。
##### 移除方式
```bash
kubectl delete deployments <deployment name>
```
##### 重啟方式
```bash
kubectl rollout restart deployment <deployment name>
```
## [ DaemonSet ] (進程守護)
![image](https://user-images.githubusercontent.com/39659664/223023844-79d31c33-fb8c-429d-a335-58c087171f9d.png)
### 說明:確保所有Node上都運行一個Pod，當有新的Node加入時，Pod也會自動生成在上面。
##### Yaml格式講解
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset              # DaemonSet 名稱
spec:
  selector:
    matchLabels:
      app: nginx                     # 與下方 template.metadata.labels 一致
      env: prod
      dept: it-cni
  template:
    metadata:
      labels:                        # 與上方selector.matchLabels 一致
        app: nginx
        env: prod
        dept: it-cni
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "200m"
            memory: "128Mi"
          limits:
            cpu: "1000m"
            memory: "256Mi"
```
##### 查看方式
```bash
kubectl get daemonsets -o wide
kubectl get pod <pod name> -o wide //可查看是否pod都在每個Node上
```
## [ Job ] (一次性任務)
### 說明:執行「一次性完成的任務」，不像 Deployment 那樣長期運行
* 批次轉檔
* 備份任務
* 單次初始化腳本
#### 新增方式Yaml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  ttlSecondsAfterFinished: 10   # 完成執行後，幾秒內自動刪除
  template:
    metadata:
    spec:
      containers:
      - name: sleep-task
        image: busybox
        command: ["sh", "-c", "sleep 5"]
      restartPolicy: Never        # 必須設定為 Never或 OnFailure
```
* restartPolicy: OnFailure   # 失敗時會嘗試重新啟
* restartPolicy: Never       # container 結束後不管成功或失敗都不會重啟
#### 驗證方式
```
### 持續查看job執行與完成後是否有自動移除
kubectl get jobs.batch
```
## [ CronJob ] (排程任務)
### 指定週期性執行任務時間，可透過CronJob達成
#### 新增方式Yaml
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob
spec:
  schedule: "*/1 * * * *"                # 每分鐘執行一次
  jobTemplate:
    spec:
      ttlSecondsAfterFinished: 10        # 執行完成後10秒自動移除
      template:
        spec:
          containers:
          - name: cronjob
            image: busybox:1.28
            command: ["sh", "-c", "echo Hello Jarry"]
          restartPolicy: OnFailure       # 任務失敗才會重新執行
```
> schedule 在更改要執行的時間，標準Linux CronJob用法
##### 驗證方式
```bash
### 查看cronjobs
kubectl get cronjobs
###移除方式
kubectl delete cronjobs <CronJob Name>
```
