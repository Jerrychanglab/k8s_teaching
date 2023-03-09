# Service (服務)使用說明
### 介紹
#### 透過Deployment部署多個Pod時，不太可能指定Pod IP單獨連線，這時就需要Service功能，在Deployment前面多一層VIP入口，並連線平均分散在每個Pod上。
#### 章節
* Cluster IP
* NodePort
* SLB
## [ Cluster IP ]
![image](https://user-images.githubusercontent.com/39659664/223951242-60974232-ae7b-4b7b-9d4d-3029759f42d8.png)
### 說明: 提供k8s集群內的服務呼叫。
> 備註:非此k8s的叢集內的服務，會無法呼叫到放出的VIP。
### 操作介紹
#### 1.使用PodClusterIP.yaml建立服務出來。
> 此Pod image採用google-samples的hello來展示。
#### 2.創建Service的Cluster IP模式
> 可透過Yaml建置，也可透過指令。
##### 方法一: 透過Yaml建置，可看ClusterIP.yaml
![image](https://user-images.githubusercontent.com/39659664/223954782-57fa0c41-d5b5-4583-bbb8-4d6bb7c626ad.png)
##### 方法二: 透過指令綁定Deployment
    kubectl expose deployment <Deployment Name> --type=ClusterIP --port=<Service Port> --target-port=<Pod Port>
#### 3.查看放出的Cluster IP
    kubectl get svc
![image](https://user-images.githubusercontent.com/39659664/223956134-caff6f0b-6fb6-4ccf-bee1-8906539ca1fd.png)
> 可看到cluster ip 有一個VIP，與設定的放出的服務Port
#### 4.驗證方式
    curl <cluster ip>:<port>
![image](https://user-images.githubusercontent.com/39659664/223956662-7cf82714-e868-42fa-83ce-a869ac199e4f.png)
> 可看到呼叫VIP時會平均的分配底下的Pod服務。
## [ NodePort ]
![image](https://user-images.githubusercontent.com/39659664/223967264-5f4b3145-12c0-45ef-bddc-4eabec5d02d5.png)
### 說明: 將每個Node的IP都變成是服務入口，並會配一個Port提供服務使用。
> 此模式，只要服務能與Node IP溝通到，就可以連線此服務。
### 操作介紹
#### 1.使用PodNodePort.yaml建立服務出來。
> 此Pod image採用google-samples的hello來展示。
#### 2.創建Service的NodePort模式
##### 方法一: 透過Yaml建置，可看NodePort.yaml
