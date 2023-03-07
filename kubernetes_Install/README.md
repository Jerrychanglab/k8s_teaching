# K8S Insatall (Centos 8)
撰寫人員:Jerry_chang
##此LAB架構，需部三台Node，並指定一台Master，與一台NFS Server做後續的掛載運用。
![image](https://user-images.githubusercontent.com/39659664/223370763-2a83de9d-6ba0-4ae0-af0c-3c79caca4058.png)

## 流程一: 建置三台 Centos8 並定義名稱與IP
### [ 下載Centos8 ]
    http://mirror01.idc.hinet.net/centos/8-stream/isos/x86_64/CentOS-Stream-8-x86_64-20230125-boot.iso
    
## 流程二: 前置環境準備
### [ Hostname修改 ]
    hostnamectl set-hostname <hostname>
### [ 關閉SELinux ]
    setenforce 0
    sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
### [ 關閉Swap ]
    sed -ri s'/.*swap.*/#&/' /etc/fstab
    
### [ 啟動偽裝，讓K8s Pod通信 ]
    cat << EOF > /etc/modules-load.d/containerd.conf
    br_netfilter
    EOF
    
    modprobe br_netfilter

### [ 調整橋接規則 ]
    cat << EOF > /etc/sysctl.d/kubernetes.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables  = 1
    EOF
    
    sysctl --system
### [ 關閉防火牆 ]
    systemctl stop firewalld.service
    systemctl disable firewalld.service

## 流程三: 安裝Docker CE
    dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo    
    dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
    dnf install -y docker-ce
    systemctl enable docker
    systemctl start docker

## 流程四: 安裝Kubernetes
### [ 新增 Kubernetes repo ]
    cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    EOF

### [ 安裝 kubeadm ]
    dnf install -y kubeadm
    dnf install -y iproute-tc                                       
    systemctl enable kubelet               
    systemctl start kubelet
    systemctl enable containerd
    systemctl restart containerd

## 流程五: 每個Node都按照流程2~4進行。
## 流程六: K8S叢集建置
### [ 初始化Master節點 ]
    mv /etc/containerd/config.toml /root/config.toml.bak
    systemctl restart containerd
#### 注意在init時，需指定apiserver ip，範例是用master ip，cidr IP屬於K8s內部配發的IP，如init失敗，進行kubeadm reset
#### token & ca 代碼請記得

    sudo kubeadm init --apiserver-advertise-address=指定master端口IP(自己) --pod-network-cidr=10.244.0.0/16
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

#### 查看Master Node狀態
    kubectl get node -o wide
### [ Worker Node Join ]
#### Join IP屬於Master APIserver IP
    kubeadm join 192.168.56.30:6443 --token m91fdb.kfxtp0ppis9aktii --discovery-token-ca-cert-hash sha256:84a9f8bc70d0ab97502800594e1502130f0011e5f83a891c5574db0a85bc2de5
#### 切換回Master查看Work Node Join狀態
    kubectl get node -o wide
#### 除錯:如遇到join跳出以下訊息
[ERROR CRI]: container runtime is not running: output: time="2023-02-05T09:28:41-05:00" level=fatal msg="validate service connection: CRI v1 runtime API is not implemented for endpoint \"unix:///var/run/containerd/containerd.sock\": rpc error: code = Unimplemented desc = unknown service runtime.v1.RuntimeService"

#### 請進行以下指令:
     
     rm -rf /etc/containerd/config.toml
     systemctl restart containerd
#### 如token 或 ca 遺忘可透過以下方法
token name請透過以下指令

    kubeadm token list
ca name請透過以下指令

    openssl x509 -in /etc/kubernetes/pki/ca.crt -pubkey -noout | openssl pkey -pubin -outform DER | openssl dgst -sha256
### [ Network overlay方案 ]
#### 透過curl抓取flannel.yaml。
    curl https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml > kube-flannel.yaml
#### 查看net-conf.json，Network是否為配置的CIDR網段
#### 新增iface參數，尋找
    command:
    - /opt/bin/flanneld
    args:
    - --ip-masq
    - --kube-subnet-mgr
    - --iface=enp0s8  <------新增通訊端口網卡名稱
    
#### 部署flannel
    kubectl apply -f flannel.yaml
#### 查看flannel狀態，是否在每個Node都自動建立。
    kubectl get pods -A -o wide
### [ 配置k8s自動補字 ]
    source <(kubectl completion bash) 
## 流程七:驗證

### [ 快速建置Pod ]
    kubectl run <Pod Name> --image=nginx:<版本> --dry-run=client -o yaml > nginx.yaml    
    kubectl apply -f nginx.yaml
### [ 查看啟動是否正常 ]

    kubectl get pods -o wide
#### 取得Pod IP並且Curl，查看回傳內容是否正常。
    curl http://<pod ip>

## END
