### EmptyDir.yaml
#### 說明:
使用Deployment建置，每個Pod內有兩個containers，分別是:nginx與busybox，透過EmptyDir做法，讓containers在Pod內共用資料夾。
### HostPath.yaml
#### 說明:
使用Deployment建置，讓Pod能使用到Work Node的空間共享。
