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
#### 4.驗證方式
```bash
curl http://<cluster ip>:<port>
```
![image](https://user-images.githubusercontent.com/39659664/223956662-7cf82714-e868-42fa-83ce-a869ac199e4f.png)
> 可看到呼叫VIP時會平均的分配底下的Pod服務。
## [ NodePort ] (內/外部服務使用)
![image](https://user-images.githubusercontent.com/39659664/223967264-5f4b3145-12c0-45ef-bddc-4eabec5d02d5.png)
### 說明: 將每個Node的IP都變成是服務入口，並會配一個Port提供服務使用。
> 此模式，只要服務能與Node IP溝通到，就可以連線此服務。
### 操作介紹
#### 1.使用PodNodePort.yaml建立服務出來。
> 此Pod image採用google-samples的hello來展示。
#### 2.創建Service的NodePort模式
##### 方法一: 透過Yaml建置，可看NodePort.yaml
![image](https://user-images.githubusercontent.com/39659664/223970705-0d6ded9a-50ef-484e-b496-88458aa91457.png)
> 透過yaml創建時，能指定Port 30000-32767之間
##### 方法二: 透過指令綁定Deployment
    kubectl expose deployment <Deployment Name> --type=NodePort --Port=<Internal Service Port> --target-port=<Pod Port>
> 透過指令綁定時，外部呼叫Port會隨機配發，範圍(30000-32767)
#### 3.查看NodePort
    kubectl get svc
![image](https://user-images.githubusercontent.com/39659664/223974411-e30e5a01-4a50-41e9-9d4a-90853c20a097.png)
#### 4.驗證方式
##### 呼叫內部IP:Port 
    curl http://<cluster ip>:<內部Port>
![image](https://user-images.githubusercontent.com/39659664/223974875-80100bb8-d897-4bb9-8d3c-061917aff007.png)
> 內部Port，需呼叫Cluster IP
##### 呼叫外部IP:Port
    curl http://<node ip>:<外部Port>
![image](https://user-images.githubusercontent.com/39659664/223975585-04c966b8-cd54-472a-9437-b273d2e321e6.png)
> 外部Port，需呼叫Node IP
## [ LoadBalancer ] (雲上SLB)
