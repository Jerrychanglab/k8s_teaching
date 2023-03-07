# Scheduling (調度)使用說明
**章節** 
- NodeSelector(節點選擇) 
- Taints(污點)
- Tolerations(反向污點)
- Maintain(維護)

## [ NodeSelector ] (節點選擇)
![image](https://user-images.githubusercontent.com/39659664/223045866-6c756acc-0685-4c37-a041-79a631971308.png)
### 說明:Pod要在指定的Node上運行，須透過此方式。
##### 建置說明
    kubectl label nodes <work node> <key>=<vale> //透過前面章節Labels設定與查看
    kubectl create deployment nodeselector --image=nignx:1.14 --dry-run=client -o yaml > NodeSelector.yaml //創建部署策略
> 打開yaml,再template底下的spec新增一個nodeSelector，並指定key與value
![image](https://user-images.githubusercontent.com/39659664/223050664-0792fa78-bcb2-4459-81ef-9354a7ecb786.png)
## [ Taints ] (污點)
![image](https://user-images.githubusercontent.com/39659664/223072610-9031e728-d73e-4dbd-a279-b3744eeabf9c.png)
### 說明:如Node上有Taints時，Pod將無法部署在上面，除非Pod有設定Tolerations，key與value需滿足Taints。
##### 操作指令
    kubectl taint node <work node> <key>=<value>:<策略>
> 策略NoSchedule:如此Node已有Pod存在，不會將其趕走 / NoExecute:如此Pod有Node會將Pod趕走，兩種選擇
##### 修改指令
    kubectl taint node <work node> <key>=<value>:<策略> --overwrite
##### 移除指令
    kubectl taint node <work node> <key>-
##### 查看指令
    kubectl describe nodes <work node> | grep Taints
## [ Tolerations ] (反向污點)
![image](https://user-images.githubusercontent.com/39659664/223073507-ccc3346d-80e5-494c-80fa-387712206032.png)
> 上圖描述，Pod設定tolerations，key/value有對應與無對應差別
### 說明:當今天Node上有Taints存在時，但Pod想在此Node上面部署時。
##### 操作指令
![image](https://user-images.githubusercontent.com/39659664/223074198-17a099b9-4938-4012-9d2e-3f4c35a538d8.png)
> 可看目前Work Node01已有配置Taints key=sz value=dep
![image](https://user-images.githubusercontent.com/39659664/223075223-30fc6f97-29f0-4803-9cf4-8d0a51a24d17.png)s
## [ Maintain ] (維護)
### 說明:Cordoning(軟隔離)，Node軟性維護，既有的Pod不會遷移，新的Pod部會部署在此Node。
##### 操作指令
    kubectl cordon <work node>
### 說明:Drain(應隔離)，Node強制維護，會將上面的Pod全數驅離。
##### 操作指令
    kubectl drain <work node> --ingore-daemonsets
##### 查看指令
    kubectl get nodes | grep SchedulingDisabled //有顯示的Node代表維護狀態
##### 解除維護
    kubectl uncordon <work node>
