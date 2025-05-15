# Scheduling (調度)使用說明
**章節** 
- NodeSelector(節點選擇) 
- Taints(污點)
- Tolerations(反向污點)
- Maintain(維護)

## [ NodeSelector ] (節點選擇)
![image](https://user-images.githubusercontent.com/39659664/223045866-6c756acc-0685-4c37-a041-79a631971308.png)
### 說明:想讓 Pod 部署到特定 Node 上時。
##### 成立條件: 需先為 Node 貼上Lable，並在 Pod 中指定對應的 nodeSelector。
##### 建置說明
`kubectl label nodes <work node> <key>=<vale> //透過前面章節Labels設定與查看`
> 打開yaml,再template底下的spec新增一個nodeSelector，並指定key與value
##### Yaml格式說明
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment             # Deployment 名稱
spec:
  replicas: 2                       
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
      nodeSelector:
        disk: ssd                    # 限定要部署到有 disk=ssd 的 Node
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
## [ Taints ] (污點) 
![image](https://user-images.githubusercontent.com/39659664/223072610-9031e728-d73e-4dbd-a279-b3744eeabf9c.png)
### 說明: Taints 是設在 Node 上的一種「限制」，用來阻止某些 Pod 被排到這台機器上，Pod 若沒有寫 Toleration 就會被排除，只有有容忍力（toleration）的 Pod 才能通過。
##### 操作指令
`kubectl taint node <work node> <key>=<value>:<策略>`
##### 屬性說明
* NoSchedule: 沒有 Toleration 的 Pod 無法被「排程」到該 Node，但已在該 Node 上的 Pod 不會被驅離。
* NoExecute: 不僅禁止新 Pod 排程，連已在此 Node 上的 Pod，如果沒有對應 Toleration，也會被強制驅離。可搭配 tolerationSeconds 設定延遲時間。
##### 修改指令
`kubectl taint node <work node> <key>=<value>:<策略> --overwrite`
##### 移除指令
`kubectl taint node <work node> <key>-`
##### 查看指令 key的地方需要更改
`kubectl get nodes -o jsonpath='{range .items[?(@.spec.taints[0].key=="env")]}{.metadata.name}{"\n"}{end}'`
## [ Tolerations ] (忽略污點)
![image](https://user-images.githubusercontent.com/39659664/223073507-ccc3346d-80e5-494c-80fa-387712206032.png)
> 上圖描述，Pod設定tolerations，key/value有對應與無對應差別
### 說明: 設定在 Pod 上的屬性，目的是讓該 Pod 可以容忍某些 Node 上的 taint，進而允許被排程到那些有「限制」的節點。
### 注意: 設定了與 Taints 相符的 Toleration，僅表示 Pod 有資格被部署到該 Node，並不代表它一定會被排程到那台 Node 上。
##### Yaml說明
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
      tolerations:             # Pod 可容忍哪些 Taints，需與 Node 上的 key / value / effect 完全對應
      - operator: "Equal"      # 比對方式：Equal 需 key + value 完全匹配
        key: "env"             # 對應 Node 上的 Taint key
        value: "prod"          # 對應 Node 上的 Taint value
        effect: "NoSchedule"   # 對應 Node 上的 Taint effect
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
##### 總結
![image](https://github.com/user-attachments/assets/922fdcf2-a92a-48bc-b63e-11803891a093)
## [ Maintain ] (維護)
### 說明:在 Kubernetes 中，當某個 Node 需要進行維護（如升級、調整資源、重啟）時，可以透過以下兩種方式控制排程器與 Pod 行為。
##### 操作指令
    kubectl cordon <work node>
### 說明:Drain(應隔離)，Node強制維護，會將上面的Pod全數驅離。
##### 操作指令
    kubectl drain <work node> --ignore-daemonsets
##### 查看指令
    kubectl get nodes | grep SchedulingDisabled //有顯示的Node代表維護狀態
##### 解除維護
    kubectl uncordon <work node>
