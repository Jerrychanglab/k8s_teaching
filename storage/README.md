# Storage(儲存)使用説明
### [ EmptyDir ]
![image](https://user-images.githubusercontent.com/39659664/223010027-1f7aa4a8-e881-45d9-870b-f185e85bc448.png)
#### 說明:讓Pod內的Containers使用共用資料空間。(範例:Pod內的containers，分別是:nginx與busybox)
### [ HostPath ]
![image](https://user-images.githubusercontent.com/39659664/223010500-437057b0-c669-439a-80ff-045cdf429e1d.png)
#### 說明:讓Pod能使用Work Node(本機)空間。
### [ Persistent ]
![image](https://user-images.githubusercontent.com/39659664/223010972-6128aaf6-19a0-4a14-9e64-1fb0d55e47cb.png)
#### 說明:讓Pod能使用NFS空間，需採用PersistentVolumes + PersistentVolumesClaims達成此目標。
##### PersistentVolumes: 簡稱PV。
##### PersistentVolumesClaims: 簡稱PVC。 
#### 建置流程:
##### 1.準備NFS Server
##### 2.PV建置，並向NFS Server挖取空間
##### 3.PVC與PV邏輯層綁定
##### 4.Pod內的Container與PVC綁定。
