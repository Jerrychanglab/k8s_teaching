# Ingress
## [ Nginx Ingress ] 
![image](https://user-images.githubusercontent.com/39659664/225511954-b2cc0554-7a4a-46f1-b003-fa448cdac74e.png)
### 說明: Ingress主要功能用於一個網址透過網址路徑導向到不同的Service並且每個Service底下都是不同的服務。
#### 前置準備
##### 步驟一:部署Nginx Ingress Contriller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml
```
> 參考文獻:https://kubernetes.github.io/ingress-nginx/deploy/
##### 步驟二: 元件檢查
```bash
kubectl get namespaces | grep ingress-nginx
```
> 檢查是否有多出一個NameSpaces名稱是ingress-nginx
```bash
kubectl get all -n ingress-nginx
```
![image](https://user-images.githubusercontent.com/39659664/225531521-a73b1957-7268-4a20-8ac5-d1d9e0a20054.png)
> 檢查個元件是否有部署上去
#### 建置流程
##### 步驟一: 建置兩個服務
###### configmap部署
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf-a-ip
data:
  custom.template: |
    server {
      listen 80;
      location / {
        default_type text/plain;
        return 200 " Deploymente A -> Pod IP: ${POD_IP}\n";
      }
    }
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf-b-ip
data:
  custom.template: |
    server {
      listen 80;
      location / {
        default_type text/plain;
        return 200 " Deploymente B -> Pod IP: ${POD_IP}\n";
      }
    }
```
###### Deployment部署
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ip-a-dynamic
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-a
      env: test
      dept: it-cni
  template:
    metadata:
      labels:
        app: nginx-a
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
          name: nginx-conf-a-ip
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ip-b-dynamic
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-b
      env: test
      dept: it-cni
  template:
    metadata:
      labels:
        app: nginx-b
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
          name: nginx-conf-b-ip
```
###### Service部署
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-a-service
spec:
  type: NodePort
  selector:
    app: nginx-a
    env: test
    dept: it-cni
  ports:
    - port: 80             # 集群內的服務 Port
      targetPort: 80     # Pod 的 containerPort
      nodePort: 30080      # 外部訪問用的 Port（必須在 30000-32767 之間）
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-b-service
spec:
  type: NodePort
  selector:
    app: nginx-b
    env: test
    dept: it-cni
  ports:
    - port: 80             # 集群內的服務 Port
      targetPort: 80       # Pod 的 containerPort
      nodePort: 30081      # 外部訪問用的 Port（必須在 30000-32767 之間）
      protocol: TCP
```
##### 步驟二: 部署Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress-ab
spec:
  rules:
  - http:
      paths:
      - path: /a                           # 使用者訪問http//:<IP>/a 路徑
        pathType: Prefix
        backend:
          service:
            name: nginx-nodeport-a-service # 將流量導向此 A Service
            port:
              number: 80
      - path: /b                           # 使用者訪問http//:<IP>/b 路徑
        pathType: Prefix
        backend:
          service:
            name: nginx-nodeport-b-service # 將流量導向此 B Service
            port:
              number: 80
```
