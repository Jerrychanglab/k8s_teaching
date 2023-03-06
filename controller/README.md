# Controller(控制器)使用說明
## Labels (標籤)
![image](https://user-images.githubusercontent.com/39659664/223018499-9863ebad-2239-424f-9ea1-62ff4983df51.png)
### 說明:定義每個物件的群組，如:Pod/Node，可快速收尋分類。
##### 新增標籤
    kubectl label <object> <pod name> <key>=<value>
##### 修改標籤
    kubectl label <object> <pod name> <key>=<value> --overwrite
##### 移除標籤
    kubectl label <object> <pod name> <key>-
##### 查詢標籤
    kubectl get <object> <pod name> --show-labels    
##### 篩選標籤
    kubectl get <object> --selector <key>=<value>
