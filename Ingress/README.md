# Ingress
**章節**
## [ Nginx Ingress ]
![image](https://user-images.githubusercontent.com/39659664/225511954-b2cc0554-7a4a-46f1-b003-fa448cdac74e.png)
### 說明: Ingress主要功能用於一個網址透過網址路徑導向到不同的Service並且每個Service底下都是不同的服務。
#### 建置流程
##### 步驟一:建置兩個服務。
> 部署PodAIngress.yaml與PodBIngress.yaml，兩個Nginx服務。
##### 步驟二:建立兩個Serice。
> 部署ServiceAIngress.yaml與ServiceBIngress.yaml，內容採用Default Cluster IP。
##### 步驟三:建置Ingress。
> 部署Ingress.yaml，講兩個Service綁定。
