### [ EmptyDir ]
#### 說明:
讓Pod內的Containers使用共用資料空間。(範例:Pod內的containers，分別是:nginx與busybox)
### [ HostPath ]
#### 說明:
讓Pod能使用Work Node(本機)空間。
### [ Persistent ]
#### 說明:
Pod如要掛載NFS Server需採用PersistentVolumes + PersistentVolumesClaims達成此目標。
##### PersistentVolumes: 簡稱PV，在NFS Server挖空間出來。
##### PersistentVolumesClaims: 簡稱PVC，將PV挖出的空間透過PVC邏輯層，綁定放給Pod掛載。
