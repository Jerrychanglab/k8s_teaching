# Service (服務)使用說明
### 介紹
##### 透過Deployment部署多個Pod時，不太可能指定Pod IP單獨連線，這時就需要Service功能，在Deployment前面多一層VIP，讓流量能平均分配。
#### 章節
* Cluster IP
* NodePort
* SLB
## [ Cluster IP ]
![image](https://user-images.githubusercontent.com/39659664/223943902-2ebc2704-8a6a-44f8-b40c-3218e63da1a1.png)
### 說明:今天透過Deployment部署多個Pod時，可透過Cluster IP負載提供給內部服務呼
