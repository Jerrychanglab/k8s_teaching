### [ EmptyDir ]
![image](https://user-images.githubusercontent.com/39659664/223010027-1f7aa4a8-e881-45d9-870b-f185e85bc448.png)
#### 說明:
讓Pod內的Containers使用共用資料空間。(範例:Pod內的containers，分別是:nginx與busybox)
### [ HostPath ]
![image](https://user-images.githubusercontent.com/39659664/223010500-437057b0-c669-439a-80ff-045cdf429e1d.png)
#### 說明:
讓Pod能使用Work Node(本機)空間。
### [ Persistent ]
#### 說明:
Pod如要掛載NFS Server需採用PersistentVolumes + PersistentVolumesClaims達成此目標。
##### PersistentVolumes: 簡稱PV，在NFS Server挖空間出來。
##### PersistentVolumesClaims: 簡稱PVC，將PV挖出的空間透過PVC邏輯層，綁定放給Pod掛載。
