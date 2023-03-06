# Scheduling (調度)使用說明
## NodeSelector
![image](https://user-images.githubusercontent.com/39659664/223045866-6c756acc-0685-4c37-a041-79a631971308.png)
### 說明:Pod要在指定的Node上運行，須透過此方式。
##### 建置說明
    kubectl label nodes <work node> <key>=<vale> //透過前面章節Labels設定與查看
    kubectl create deployment nodeselector --image=nignx:1.14 --dry-run=client -o yaml > NodeSelector.yaml //創建部署策略
> 打開yaml,再template底下的spec新增一個nodeSelector，並指定key與value
![image](https://user-images.githubusercontent.com/39659664/223049369-50c07bd3-ea17-4e41-bbc4-0a0acb5e063b.png)
