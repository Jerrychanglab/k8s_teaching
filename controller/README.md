# Controller(控制器)使用說明
### Labels (標籤)
#### 說明:定義每個物件的群組，如:Pod/Node。備註:如想要指定Pod部署在哪些Node上，可透過Labels實踐。
操作指令(Imperatively):
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
