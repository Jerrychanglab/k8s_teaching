# Ingress
## [ Nginx Ingress ] 
![image](https://github.com/user-attachments/assets/b73c3b53-b691-41c8-a919-00282692523a)

### 說明: Ingress 的主要功能是提供 HTTP/HTTPS 的統一入口，允許使用相同網域名稱下的不同路徑，分別導向到不同的 Service，進而存取後端的多個應用服務。
| 功能類別                       | 功能說明                                           |
| -------------------------- | ---------------------------------------------- |
| **路由轉發**                   | 根據 `IP` + `Path` 將流量導向不同的 Service            |
| **HTTPS 與 TLS**            | 使用 `ManagedCertificate` 自動配置 TLS，支援自動續期        |
| **靜態 IP 綁定**               | 將 Ingress 綁定至固定 Global IP，對外 URL 穩定不變          |
| **HTTP → HTTPS 導轉**        | 自動轉導到 HTTPS，透過 annotation 設定                   |

#### 前置準備 (如果用GCP內建的可忽略)
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
##### 步驟三: 驗證
###### 查看SVC
```bash
NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-nodeport-a-service   NodePort    10.14.51.193   <none>        80:30080/TCP   4h24m
nginx-nodeport-b-service   NodePort    10.14.51.90    <none>        80:30081/TCP   4h24m
```
###### 查看Ingress
```bash
[root@gke-teaching service]# kubectl get ingress
NAME               CLASS    HOSTS   ADDRESS      PORTS   AGE
nginx-ingress-ab   <none>   *       34.8.0.132   80      65m

[root@gke-teaching service]# kubectl describe ingress nginx-ingress-ab 
Name:             nginx-ingress-ab
Labels:           <none>
Namespace:        default
Address:          34.8.0.132
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /a   nginx-nodeport-a-service:80 (10.99.64.15:80)
              /b   nginx-nodeport-b-service:80 (10.99.64.16:80,10.99.64.19:80)
```
###### 測試
```bash
curl http://34.8.0.132/a
輸出: Deploymente A -> Pod IP: 10.99.64.15

curl http://34.8.0.132/b
輸出: Deploymente B -> Pod IP: 10.99.64.16
```
