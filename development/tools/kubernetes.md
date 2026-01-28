# Kubernetes

## Ubuntu 18 安装kubernetes

### 1. 准备工作

1. 关闭防火墙
    ```shell script
    sudo ufw disable
    ```

1. 关闭系统swap
    ```shell script
    sudo swapoff -a   
    # 修改/etc/fstab，注释掉swap那行，持久化生效
    sudo vi /etc/fstab
    ```
   
1. 安装docker
    ```shell script
    sudo apt install docker.io
    ```


### 2. 准备k8s安装环境

1. 配置k8s的源
    ```shell script
    sudo apt update && sudo apt install -y apt-transport-https curl
    curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
     
    sudo touch /etc/apt/sources.list.d/kubernetes.list 
    sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> /etc/apt/sources.list.d/kubernetes.list
    #配置国内中科大的源 并添加Kubernetes安装的密钥 
    
    curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
    sudo echo "deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
    sudo apt update
    ```

1. 安装kubeadm及kubelet, kubectl等工具
    ```shell script
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    # 保持版本，取消自动更新
    
    sudo apt-mark hold kubelet kubeadm kubectl
    ```

### 3. kubeadm初始化集群

* 执行初始化master操作  只有在 master主机上执行的操作

    注意 主机所在的网段不可以和pod设置的网段冲突否则会导致pod ip 不通
    
    sudo kubeadm init --image-repository=registry.aliyuncs.com/google_containers  --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=xxx.xx.xx.xx --kubernetes-version=v1.15.3
     
* 初始化选项的意义：

    --apiserver-advertise-address：指定用 Master 的哪个IP地址与 Cluster的其他节点通信。
    
    --service-cidr：指定Service网络的范围，即负载均衡VIP使用的IP地址段。
    
    --pod-network-cidr：指定Pod网络的范围，即Pod的IP地址段。
    
    --image-repository：Kubenetes默认Registries地址是 k8s.gcr.io，在国内并不能访问 gcr.io，在1.13版本中我们可以增加-image-repository参数，默认值是 k8s.gcr.io，将其指定为阿里云镜像地址：registry.aliyuncs.com/google_containers。
    
    --kubernetes-version=v1.15.3：指定要安装的版本号。
    
    --ignore-preflight-errors=：忽略运行时的错误，例如上面目前存在[ERROR NumCPU]和[ERROR Swap]，忽略这两个报错就是增加--ignore-preflight-errors=NumCPU 和--ignore-preflight-errors=Swap的配置即可。

    `注：kubernetes的版本要和你安装的一致 cpu 如果小于2的话 也要把NumCPU错误 ignore掉 详细可以查看官方文档 
    记录下来在 node 部署的时候会用到`

* 执行后会显示 成功的字样
    ```shell script
    // ...
    Your Kubernetes control-plane has initialized successfully!
     
    并输出
    kubeadm join 192.168.200.212:6443 --token q1guce.z76o2a2bb65vhd0u \
        --discovery-token-ca-cert-hash sha256:2a57a27853c66d608bc544742b57602a21d47c3d09fe58eef15258946d4341c0 
    
    ```
* 查看k8s集群的pod状态

    kubectl get pods --all-namespaces
    出现The connection to the server localhost:8080 was refused - did you specify the right host or port?
    
    连接到服务器localhost:8080被拒绝我是以root用户执行的不知道为什么也出现这个错误 但是执行以下命令cp 一下config文件就好了这个解决方案 仅仅在 安装时报这个错误时测试过其他场景没有试过，不要病急乱投医，搞清楚为什么，我这里就不再深聊这个问题了 
    
    ```shell script
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
    
* 安装calico插件

    * 从国内的一个源里下载

        kubectl apply -f http://mirror.faasx.com/k8s/calico/v3.3.2/rbac-kdd.yaml
        
        kubectl apply -f http://mirror.faasx.com/k8s/calico/v3.3.2/calico.yaml
    
    * Master隔离解除(主节点也能部署工作任务)可选
    
        `kubectl taint nodes --all node-role.kubernetes.io/master-`
    
    * 成功时会输出类似提示:
    
        `node/xiaomo untainted`

### 4. 加入工作节点(假设有其他的节点需要加入到集群)

登陆到需要加入的服务器上执行

kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>

也就是之前init  master主机时输出的 token

kubeadm join 10.11.10.6:6443 --token d5bsdr.gqtjk98nnkiyb4pq \
    --discovery-token-ca-cert-hash sha256:60f6f6833611dd60ea9449fbd9dcb3b533c81602f032bbb8db8cb91c41ebc3a0
### 5. 查看集群状态

    ```shell script
    kubectl get pods -n kube-system
    ```
