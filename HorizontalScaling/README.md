# HorizontalScaling(水平縮放)
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
```bash
kubectl describe nodes <Node Name> | grep -A11 Capacity
```
> 查看cpu欄位，如顯示4就代表您有4000個mCPU
## [ Metrics採集 ]
### 說明: 在一般 K8s 環境中，若要查看每個 Node 或 Pod 的資源使用率（CPU / Memory），需安裝 Metrics Server 作為資料採集器。（在 GCP 的 GKE 環境中，Metrics Server 通常已預先安裝）
#### Metrics Server建置
##### 步驟一:抓取Metrics yaml
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml
```
> 部署metrics server會存在kube-system的namespaces內
> 必須要有兩個Node節點才能部署
##### 步驟二:修改yaml新增tls參數，不驗證CA證書
```bash
kubectl edit deployments.apps -n kube-system metrics-server
```
![image](https://user-images.githubusercontent.com/39659664/224663352-adcc7054-66db-48a1-b847-93d9a191101e.png)
> 新增 --kubelet-insecure-tls
##### 步驟三: 重新啟動
```bash
kubectl rollout restart deployment -n kube-system metrics-server
```
##### 步驟四: 驗證
```bash
kubectl get deployments.apps -n kube-system metrics-server
```
![image](https://user-images.githubusercontent.com/39659664/224880162-902da9d7-efd4-4726-b7be-fa9a7a60efd1.png)

    kubectl get service -n kube-system metrics-server
![image](https://user-images.githubusercontent.com/39659664/224880505-b425e900-ddc9-4ee8-a78e-9b8faa4aa253.png)
> 確認是否READY
#### 使用率查看
##### Node
    kubectl top node
##### Pod
    kubectl top pods
## [ Pod AutoScaling (HPA) ]
![image](https://user-images.githubusercontent.com/39659664/224660612-84ec2739-4c5a-4a15-96fb-0c31ec69250d.png)
### 說明: 透過AutoScaling有效的控管Pod使用數量，讓資源妥善運用，並且當規則配置完成後，當遇到大量需求量時，會透過閥值偵測(Metrice Server)，自動擴充Pod的數量，後續需求量減少，會自己將Pod回收。
> 備註:HPA機制，仰賴Metrice Server，如無此服務，會倒置HPA功能失效。
#### HPA建置
##### 步驟一: Deployment創建服務(定義cpu/memory的requests/limits)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment            # Deployment明稱給HPA對應
spec:
  replicas: 2                       # 想要運行的 Pod 數量
  selector:
    matchLabels:
      app: nginx
      env: prod
      dept: it-cni
  template:
    metadata: 
      labels:
        app: nginx
        env: prod
        dept: it-cni
    spec:
      containers:
      - name: nginx                  # 容器名稱
        image: nginx:latest          # 容器鏡像
        ports:
        - containerPort: 80          # 容器開放 Port
        resources:
          requests:
            cpu: "200m"              # 最少保留 0.2 顆 CPU
            memory: "128Mi"          # 最少保留 128MiB 記憶體
          limits:
            cpu: "1000m"             # 最多使用 1 顆 CPU
            memory: "256Mi"          # 最少保留 256 MiB 記憶體
```
##### 步驟二: 部署HPA Yaml。
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment           # 目標資源類型，這裡是 Deployment
    name: nginx-deployment     # 要套用 HPA 的 Deployment 名稱
  minReplicas: 1               # 最少維持的副本數
  maxReplicas: 2               # 最多擴展到的副本數
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # 當 CPU 使用率超過 70% 時會啟動擴展
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80  # 當 RAM 使用率超過 80% 時會啟動擴展
```
## [ Vertical Pod Autoscaler (VPA) ]
### 說明: VPA 是 Kubernetes 提供的資源管理元件，能為 Pod 建議或自動調整 requests 和 limits 的值。其用途是「調整單一 Pod 所需資源」，而不是改變 Pod 數量。VPA 搭配 Metrics Server 收集歷史資源使用資料後提出建議，特別適合資源需求不穩定或長時間執行的應用場景。
#### 模式說明：
* Off: 只給建議，不做任何調整
* Initial: 初次啟動 Pod 套用建議值
* Auto: 動態調整資源，會導致 Pod 重啟
#### 建議搭配
* VPA Off 搭配 HPA（避免衝突）
### VPA搭建
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  updatePolicy:
    updateMode: "Off"  # 只建議不實作
```
