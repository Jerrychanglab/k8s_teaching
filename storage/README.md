# Storage(儲存)使用説明
### 章節
* EmptyDir
* HostPath
* Persistent
* ConfgMap
* Secrets
### [ EmptyDir ]
#### 說明:效果是讓同一個 Pod 內的多個容器可以共用同一塊臨時儲存空間（例如 /var/log），Pod 結束時，資料也會被移除。
![image](https://user-images.githubusercontent.com/39659664/223010027-1f7aa4a8-e881-45d9-870b-f185e85bc448.png)
##### yaml說明
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sidecar-pod                 
spec:
  containers:
  - name: nginx                           
    image: nginx:1.14.2                   
    ports:
    - containerPort: 80                   
    volumeMounts:
    - name: shared-logs                   # 掛載名為 shared-logs 的 Volume
      mountPath: /var/log/nginx           # 將 Volume 掛載到 nginx 的 log 資料夾路徑
    resources:
      requests:
        cpu: "500m"                       
        memory: "128Mi"                   
      limits:
        cpu: "1000m"                     
        memory: "256Mi"                   

  - name: log-agent                       
    image: busybox                       
    command: ["/bin/sh", "-c", "tail -n+1 -F /input/nginx/access.log"]
                                         
    volumeMounts:
    - name: shared-logs                  # 與主容器共用同一個 Volume
      mountPath: /input/nginx            # 掛載在 log-agent 內部為 /input/nginx

  volumes:
  - name: shared-logs                   # 定義一個名為 shared-logs 的共享 Volume
    emptyDir: {}                        # 使用 emptyDir 類型：當 Pod 被建立時產生，刪除後資料會消失
```
### [ HostPath ]
#### 說明:允許 Pod 直接掛載所在 Node（工作節點）上的本機路徑，如Pod移除，work node上的內容會保留。
![image](https://user-images.githubusercontent.com/39659664/223010500-437057b0-c669-439a-80ff-045cdf429e1d.png)
##### Yaml說明
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo                    
spec:
  containers:
  - name: nginx                          
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: host-logs                    # 掛載名為 host-logs 的 Volume
      mountPath: /mnt/host-logs          # 容器內路徑：可看到 Node 的 /var/log 內容
    resources:
      requests:
        cpu: "200m"
        memory: "128Mi"
      limits:
        cpu: "1000m"
        memory: "256Mi"            

  volumes:
  - name: host-logs                      # 定義 Volume 名稱
    hostPath:
      path: /var/log/jarry               # Node 上實際要掛載的路徑
      type: DirectoryOrCreate            # 若 Node 上此路徑不存在，則自動建立資料夾
```
### [ Persistent ]
![image](https://user-images.githubusercontent.com/39659664/223010972-6128aaf6-19a0-4a14-9e64-1fb0d55e47cb.png)
#### 說明:讓Containers能使用NFS空間，需採用PersistentVolumes + PersistentVolumesClaims達成此目標。
##### PersistentVolumes: 簡稱PV。
##### PersistentVolumesClaims: 簡稱PVC。 
#### 建置流程:
##### 1.準備NFS Server
##### 2.PV建置，並向NFS Server挖取空間
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 5Gi                     # 假設限制為 5Gi，可自行調整
  accessModes:
    - ReadWriteMany                  # NFS 支援多個 Pod 同時掛載
  nfs:
    server: 10.104.6.63              # NFS 伺服器 IP
    path: /gcp_backup_volume         # 掛載的實際路徑
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-sc           # 自定義的 StorageClass 名稱
```
##### 3.PVC與PV邏輯層綁定
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: nfs-sc   # 需要與PV的storageClassName完全一致
```
##### 4.Containers綁定pvc。
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
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "200m"
            memory: "128Mi"
          limits:
            cpu: "1000m"
            memory: "256Mi"
        volumeMounts:
        - name: nfs-volume                     # 輸入下方volumes的name
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nfs-volume                       # 提供給上方 volumeMounts的 name使用
        persistentVolumeClaim:
          claimName: nfs-pvc                   # 這要跟 PVC 名稱一致
```
### [ ConfigMap ]
![image](https://user-images.githubusercontent.com/39659664/223612473-5118e1c3-9ebc-4d84-9c97-b64f8dfbc0a9.png)
#### 說明:建立一份設定檔（ConfigMap），可作為環境變數或掛載成檔案，供容器於啟動時讀取使用。
#### 指令說明:
##### 查看目前的 configmap
```bash
kubectl get configmap
```
#### 建置流程:
##### 1.創建configMap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf-config    # configmap名稱
data:                        # 文件內容
  custom.conf: |
    server {
      listen 80;
      location / {
        default_type text/plain;
        return 200 'Hello from ConfigMap NGINX config';
      }
    }
```
##### 2.服務掛載此環境參數或文件。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    env: test
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
    volumeMounts:
    - name: nginx-config
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: nginx-config
    configMap:
      name: nginx-conf-config
```
> 使用PodConfigMap.yaml，達成Containers呼叫configmap的參數或文件使用。
#### 注意事項:
##### 更新 ConfigMap 後，Pod 不會自動載入新內容，若為 Deployment 管理，需執行 rollout restart 重新部署以套用變更。
### [ Secrets ]
使用說明: 透過加密機制，將加密內容傳遞給Containers的環境變數使用。
#### 指令說明:
```data
kubectl get secrets
```
#### 建置流程:
##### 創建Secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  rootpw: bXlwYXNzd29yZA==  # mypassword
```
> Secret加密是透過base64 (echo -n '<passwd>' | base64) 
##### 2.建置Mysql，使用Secret加密機制
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
spec:
  containers:
  - name: mysql
    image: mysql:5.7
    env:
    - name: MYSQL_ROOT_PASSWORD # MySQL 容器內建的環境變數，用於設定 root 密碼
      valueFrom:
        secretKeyRef:
          name: mysql-secret    # 這裡引用的是事先建立的 Secret 資源名稱
          key: rootpw           # 對應 Secret.data 中的欄位名稱（即密碼的 key）
    ports:
    - containerPort: 3306
```
##### 3.部署完成後，請進入Mysql查看
```bash
kubectl exec -it <pod name> -- sh
mysql -u root -p
```
