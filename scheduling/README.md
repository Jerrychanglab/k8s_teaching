# Scheduling (調度)使用說明
## NodeSelector (節點選擇)
![image](https://user-images.githubusercontent.com/39659664/223045866-6c756acc-0685-4c37-a041-79a631971308.png)
### 說明:Pod要在指定的Node上運行，須透過此方式。
##### 建置說明
    kubectl label nodes <work node> <key>=<vale> //透過前面章節Labels設定與查看
    kubectl create deployment nodeselector --image=nignx:1.14 --dry-run=client -o yaml > NodeSelector.yaml //創建部署策略
> 打開yaml,再template底下的spec新增一個nodeSelector，並指定key與value
![image](https://user-images.githubusercontent.com/39659664/223050664-0792fa78-bcb2-4459-81ef-9354a7ecb786.png)
## Taints (污點)
![image](https://user-images.githubusercontent.com/39659664/223071892-ac5c95fe-c608-4fcd-908a-9d670c75bb0f.png)
### 說明:如Node上有Taints時，Pod將無法部署在上面，除非有對應的Tolerations。
#### 操作指令
    kubectl taint node <work node> <key>=<value>:<策略>
> 策略NoSchedule:如此Node已有Pod存在，不會將其趕走 / NoExecute:如此Pod有Node會將Pod趕走，兩種選擇
#### 修改指令
    kubectl taint node <work node> <key>=<value>:<策略> --overwrite
#### 移除指令
    kubectl taint node <work node> <key>-
#### 查看指令
    kubectl describe nodes <work node> | grep Taints
## Tolerations (反向污點)
### 說明:當今天Node上有Taints存在時，但Pod想在此Node上面部署時。

