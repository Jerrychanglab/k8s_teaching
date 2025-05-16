# Service (服務)使用說明
### 介紹
#### 透過Deployment部署多個Pod時，不太可能指定Pod IP單獨連線，這時就需要Service功能，在Deployment前面多一層VIP入口，並連線平均分散在每個Pod上。
### 重要
* Kubernetes 的 Service 並不會「綁定某個 Deployment或」，它只會根據 selector label 去匹配 Pod。若多個 Deployment 使用相同 label，這些 Pod 都會被這個 Service 同時導向。
#### 章節
* Cluster IP
* NodePort
* LoadBalancer
## [ Cluster IP ] (內部服務使用)
![image](https://user-images.githubusercontent.com/39659664/223951242-60974232-ae7b-4b7b-9d4d-3029759f42d8.png)
### 說明: 提供集群內部的服務存取入口，僅可由同一個叢集內的服務呼叫。
> 備註:非此k8s的叢集內的服務，會無法呼叫到放出的VIP。
#### 部署服務: 呼叫服務時會回應Pod的IP驗證是否有負載均衡
##### 1. 建置ConfigMap
```yaml
kind: ConfigMap
metadata:
  name: nginx-conf-ip
data:
  custom.template: |
    server {
      listen 80;
      location / {
        default_type text/plain;
        return 200 "Hello from Pod IP: ${POD_IP}\n";
      }
    }
```
##### 2. 建置Nginx服務(Deployment)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ip-dynamic
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
      env: test
      dept: it-cni
  template:
    metadata:
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
        command: ["/bin/sh", "-c"]
        args:
          - |
            export POD_IP=$(hostname -i);
            envsubst < /etc/nginx/template/custom.template > /etc/nginx/conf.d/default.conf;
            nginx -g 'daemon off;';
        volumeMounts:
        - name: nginx-template
          mountPath: /etc/nginx/template
      volumes:
      - name: nginx-template
        configMap:
          name: nginx-conf-ip
```
##### 3. 建置SVC (Cluster IP)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip-service   # Service 名稱，可自訂
spec:
  type: ClusterIP                 # 預設值，可省略；僅供集群內部使用
  selector:
    app: nginx                    # 與 deployment 的 Pod Label 匹配
    env: test
    dept: it-cni
  ports:
  - port: 8080                    # Service 對外提供的 Port（VIP Port）
    targetPort: 80                # 對應 Pod 中 containerPort
    protocol: TCP

```
##### 4. 查看 SVC (Cluster)
```bash
kubectl get svc
# 輸出
NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes                ClusterIP   10.144.5.1    <none>        443/TCP    7d
nginx-clusterip-service   ClusterIP   10.144.5.18   <none>        8080/TCP   48m
```
##### 5. 驗證是否有分散需求處理
```bash
# 查看目前 Pod的 IP
kubectl get pod -o wide
## 輸出
nginx-ip-dynamic-cb966cbf6-d9rq8   1/1     Running   0             14m     10.219.0.215   gke-n8n-prod-default-pool-c52506a0-4qzp   <none>           <none>
nginx-ip-dynamic-cb966cbf6-j5xwx   1/1     Running   0             14m     10.219.1.20    gke-n8n-prod-default-pool-d0993e26-tpkw   <none>           <none>

# curl進行呼叫
curl 10.144.5.18:8080
## 輸出
/ # curl 10.144.5.18:8080
Hello from Pod IP: 10.219.0.215
/ # curl 10.144.5.18:8080
Hello from Pod IP: 10.219.1.20
```
##### 6. 驗證 Pod加上一樣的Lable是否也以被svc託管
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ip-pod
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
    command: ["/bin/sh", "-c"]
    args:
      - |
        export POD_IP=$(hostname -i);
        envsubst < /etc/nginx/template/custom.template > /etc/nginx/conf.d/default.conf;
        nginx -g 'daemon off;';
    volumeMounts:
      - name: nginx-template
        mountPath: /etc/nginx/template
  volumes:
    - name: nginx-template
      configMap:
        name: nginx-conf-ip
```
##### 7. 如步驟 5 進行驗證

## [ NodePort ] (內/外部服務使用)
![image](https://user-images.githubusercontent.com/39659664/223967264-5f4b3145-12c0-45ef-bddc-4eabec5d02d5.png)
### 說明: NodePort 會在每個 Node 上開放一個固定的 Port 作為服務入口，只要可以連線到任何一台 Node 的該 Port，就能透過 Service 存取後端的 Pod。
##### 1. 部署SVC (NodePort)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx
    env: test
    dept: it-cni
  ports:
    - port: 8080           # 集群內呼叫的服務 Port
      targetPort: 80       # Pod 的 container Port
      nodePort: 30080      # 外部呼叫用的 Port（必須在 30000-32767 之間）
      protocol: TCP
```
##### 2.查看NodePort
```bash
kubectl get svc
## 輸出
nginx-nodeport-service    NodePort    10.144.5.19   <none>        8080:30080/TCP   7m50s
```
##### 3. 驗證
```bash
# 叢集內呼叫
curl 10.144.5.19:8080
# 叢集外呼叫
curl <Node Ip>:30080
```
## [ LoadBalancer ] (雲上SLB)
