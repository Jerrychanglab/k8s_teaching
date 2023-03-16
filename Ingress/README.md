# Ingress
## [ Nginx Ingress ]
![image](https://user-images.githubusercontent.com/39659664/225511954-b2cc0554-7a4a-46f1-b003-fa448cdac74e.png)
### 說明: Ingress主要功能用於一個網址透過網址路徑導向到不同的Service並且每個Service底下都是不同的服務。
#### 前置準備
##### 步驟一:部署Nginx Ingress Contriller
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml
> 參考文獻:https://kubernetes.github.io/ingress-nginx/deploy/
##### 步驟二: 元件檢查
     kubectl get namespaces | grep ingress-nginx
> 檢查是否有多出一個NameSpaces名稱是ingress-nginx

     kubectl get all -n ingress-nginx
![image](https://user-images.githubusercontent.com/39659664/225531521-a73b1957-7268-4a20-8ac5-d1d9e0a20054.png)

> 檢查個元件是否有部署上去

#### 建置流程
##### 步驟一:準備一個Domain
> 因屬於測試環境，故在Master node寫/etc/hosts，綁定IP部分需拿service的ingress-nginx-controller的Cluster IP

    kubectl get svc -n ingress-nginx

![image](https://user-images.githubusercontent.com/39659664/225532573-511311ae-99dc-42cb-a44d-bbf9cff611dd.png)

> 例:網址:k8s.ingress.com IP:ingress-nginx-controller的Cluster IP

![image](https://user-images.githubusercontent.com/39659664/225533083-d8e58dad-7430-40a2-84fc-7afe4d3e678e.png)
##### 步驟二:建置兩個服務。
> 部署PodAIngress.yaml與PodBIngress.yaml，兩個Nginx服務。
##### 步驟三:建立兩個Service。
> 部署ServiceAIngress.yaml與ServiceBIngress.yaml，內容採用Default Cluster IP 
![image](https://user-images.githubusercontent.com/39659664/225514278-fa4ad363-5244-438a-a0da-4938adbf62bd.png)

##### 步驟四:建置Ingress。
> 部署Ingress.yaml，兩個Service綁定。

![image](https://user-images.githubusercontent.com/39659664/225515231-3e182fde-6eb7-4daf-9016-9b7c1e350713.png)
##### 步驟五:驗證
    curl <Domain>:80/con
    curl <Domain>:80/age
  
###### 結論
此實驗可以觀察到，透過單一網址的路徑，來達成轉到不同Service底下的服務。
