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
### 說明:如Node上有Taints時，Pod將無法部署在上面，除非Pod有設定Tolerations，key與value需滿足Taints。
##### 操作指令
    kubectl taint node <work node> <key>=<value>:<策略>
> 策略NoSchedule:如此Node已有Pod存在，不會將其趕走 / NoExecute:如此Pod有Node會將Pod趕走，兩種選擇
##### 修改指令
    kubectl taint node <work node> <key>=<value>:<策略> --overwrite
##### 移除指令
    kubectl taint node <work node> <key>-
##### 查看指令
    kubectl describe nodes <work node> | grep Taints
## [ Tolerations ] (反向污點)
![image](https://user-images.githubusercontent.com/39659664/223073507-ccc3346d-80e5-494c-80fa-387712206032.png)
> 上圖描述，Pod設定tolerations，key/value有對應與無對應差別
### 說明:當今天Node上有Taints存在時，但Pod想在此Node上面部署時。
##### 操作指令
![image](https://user-images.githubusercontent.com/39659664/223074198-17a099b9-4938-4012-9d2e-3f4c35a538d8.png)
> 可看目前Work Node01已有配置Taints key=sz value=dep
![image](https://user-images.githubusercontent.com/39659664/223075223-30fc6f97-29f0-4803-9cf4-8d0a51a24d17.png)s
## [ Maintain ] (維護)
### 說明:Cordoning(軟隔離)，Node軟性維護，既有的Pod不會遷移，新的Pod部會部署在此Node。
##### 操作指令
    kubectl cordon <work node>
### 說明:Drain(應隔離)，Node強制維護，會將上面的Pod全數驅離。
##### 操作指令
    kubectl drain <work node> --ignore-daemonsets
##### 查看指令
    kubectl get nodes | grep SchedulingDisabled //有顯示的Node代表維護狀態
##### 解除維護
    kubectl uncordon <work node>
