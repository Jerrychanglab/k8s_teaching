
## Pod 是什麼
### Pod 是 Kubernetes 中最小的部署與運行單位，它可以封裝一個或多個container，這些container共享同一個網路（IP、Port）、儲存與生命週期。
* 每個Pod會在Node上運行
* 每個Pod對應一個應用(如:Nginx)  
![image](https://github.com/user-attachments/assets/1d13da9b-1f26-4335-a227-b420e8ffcc1c)
##### Pod的yaml格式
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx             # 容器名稱
    image: nginx:1.14.2     # 容器 Image 來源（預設從 Docker Hub 拉取）
    ports:
    - containerPort: 80     # 容器開放的服務 Port
    resources:
      requests:
        cpu: "200m"         # 最好保證 0.2 CPU 
        memory: "128Mi"     # 最少保證 128MiB 記憶體
      limits:
        cpu: "1000m"         # 最多允許使用 1 CPU
        memory: "256Mi"     # 最多允許使用 256MiB 記憶體
```
##### 部署方式
`kubectl apply -f pod.yaml`
##### 查看方式
`kubectl describe pod nginx`

## Pod 與 Container 的關係
### Pod 就像是一個容器「外殼」，它包裹了一個或多個 container，讓這些 container 在同一個環境中協同運作。
![image](https://github.com/user-attachments/assets/c1de2e7f-6857-450f-a390-0644e0f4d830)

## Sidecar 是什麼
### Sidecar 是一種設計模式，在同一個 Pod 中啟動第二個（或多個）輔助容器，用來提供主容器額外功能，例如：紀錄、代理、監控等。
* 主容器跑 nginx，sidecar 容器跑 fluentbit，收集並上傳 log。
* 共用 Pod 的網路與空間，能直接看到主容器的資料與通訊。

![image](https://github.com/user-attachments/assets/71711225-3fd4-407c-848e-7636e5f0577c)
