# Service (服務)使用說明
### 介紹
#### 透過Deployment部署多個Pod時，不太可能指定Pod IP單獨連線，這時就需要Service功能，在Deployment前面多一層VIP入口，並連線平均分散在每個Pod上。
#### 章節
* Cluster IP
* NodePort
* LoadBalancer
## [ Cluster IP ] (內部服務使用)
![image](https://user-images.githubusercontent.com/39659664/223951242-60974232-ae7b-4b7b-9d4d-3029759f42d8.png)
### 說明: 提供集群內部的服務存取入口，僅可由同一個叢集內的服務呼叫。
> 備註:非此k8s的叢集內的服務，會無法呼叫到放出的VIP。
#### 部署服務
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment             # Deployment 名稱
spec:
  replicas: 2
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
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80          # 容器開放 Port
        resources:
          requests:
            cpu: "200m"
            memory: "128Mi"
          limits:
            cpu: "1000m"
            memory: "256Mi"
```
#### 2.創建Service的Cluster IP模式
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip-service   # Service 名稱，可自訂
spec:
  type: ClusterIP                 # 預設值，可省略；僅供集群內部使用
  selector:
    app: nginx                    # 與 deployment 的 Pod Label 匹配
    env: prod
    dept: it-cni
  ports:
  - port: 8080                    # Service 對外提供的 Port（VIP Port）
    targetPort: 80                # 對應 Pod 中 containerPort
    protocol: TCP
```
#### 3.查看放出的Cluster IP
    kubectl get svc
![image](https://user-images.githubusercontent.com/39659664/223956134-caff6f0b-6fb6-4ccf-bee1-8906539ca1fd.png)
> 可看到cluster ip 有一個VIP，與設定的放出的服務Port
#### 4.驗證方式
    curl http://<cluster ip>:<port>
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
