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
![image](https://user-images.githubusercontent.com/39659664/223010500-437057b0-c669-439a-80ff-045cdf429e1d.png)
#### 說明:讓Containers能使用Work Node(本機)空間。
![image](https://user-images.githubusercontent.com/39659664/223604858-7112fc3c-2441-4fe1-b72b-7ce675c8b037.png)
> 使用HostPath.yaml，效果是Containers能使用work node上的路徑空間。如Pod移除，work node上的內容會保留。
### [ Persistent ]
![image](https://user-images.githubusercontent.com/39659664/223010972-6128aaf6-19a0-4a14-9e64-1fb0d55e47cb.png)
#### 說明:讓Containers能使用NFS空間，需採用PersistentVolumes + PersistentVolumesClaims達成此目標。
##### PersistentVolumes: 簡稱PV。
##### PersistentVolumesClaims: 簡稱PVC。 
#### 建置流程:
##### 1.準備NFS Server
##### 2.PV建置，並向NFS Server挖取空間
> 使用PersistentVolumes.yaml並指定work node能溝通到的nfs ip與路徑，並且pv的labels key:value需與後續pvc一致。
##### 3.PVC與PV邏輯層綁定
> 使用PersistentVolumesClaims.yaml，pvc的labels key:value需要與pv一致，並且與PV拿儲存空間。
##### 4.Pod內的Container與PVC綁定。
> 使用PodMountPVC.yaml，claimName需填寫pvc的name
### [ ConfigMap ]
![image](https://user-images.githubusercontent.com/39659664/223612473-5118e1c3-9ebc-4d84-9c97-b64f8dfbc0a9.png)
#### 說明:創建一個環境參數或文件，讓有需要使用此內容的Container自行帶入使用。
#### 建置流程:
##### 1.創建configMap.yaml
![image](https://user-images.githubusercontent.com/39659664/223613431-dae72a3d-2b78-4ac5-94a1-b10001784d7f.png)
> 使用ConfigMap.yamls，範例用nginx的default.conf說明
##### 2.掛載此環境參數或文件。
![image](https://user-images.githubusercontent.com/39659664/223614311-985f364f-9eb9-411d-9471-63a424d38363.png)
> 使用PodConfigMap.yaml，達成Containers呼叫configmap的參數或文件使用。
### [ Secrets ]
使用說明: 透過加密機制，將加密內容傳遞給Containers的環境變數使用。
> 此範例透過Secrets加密，更改Mysql的登入密碼。
#### 建置流程:
##### 1.創建Secret.yaml
    echo -n '<passwd>' | base64
![image](https://user-images.githubusercontent.com/39659664/223894085-6540614e-c03f-418a-a319-66416fa071b9.png)
> 使用Secret.yaml，文件內password後面的密碼需填寫base64的字元
##### 2.建置Mysql，使用Secret加密機制
![image](https://user-images.githubusercontent.com/39659664/223894740-6d94579d-e330-4905-9b90-c5cc800ceb6b.png)
> 使用PodSecrets.yaml
##### 3.部署完成後，請進入Mysql查看
    kubectl exec -it <pod name> -- sh
    mysql -u root -p
![image](https://user-images.githubusercontent.com/39659664/223895723-140bfdd2-a542-4292-bb46-2888a67bf051.png)
