### EmptyDir
#### 說明:
使用Deployment建置，每個Pod內有兩個containers，分別是:nginx與busybox，透過EmptyDir做法，讓containers在Pod內共用資料夾。
### HostPath
#### 說明:
使用Deployment建置，讓Pod能使用到Work Node的空間共享。
### Persistent
#### 說明:
如要掛載NFS需採用PersistentVolumes + PersistentVolumesClaims達成此目標。
##### PersistentVolumes: 簡稱PV，在NFS Server那挖空間出來。
##### PersistentVolumesClaims: 簡稱PVC，將PV挖出的空間透過PVC綁定放給Pod掛載。
