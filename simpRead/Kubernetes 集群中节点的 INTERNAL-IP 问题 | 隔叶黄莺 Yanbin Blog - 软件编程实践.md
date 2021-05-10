> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [yanbin.blog](https://yanbin.blog/kubernetes-cluster-internal-ip-issue/#more-10102)

用自己 [Kubernetes 学习笔记 (一) - 初上手](https://yanbin.blog/kubernetes-learning-1/) 一文中的方法用 Vagrant 虚拟机安装的 Kubernetes 集群，部署应用什么的都没问题，然而却在用

> $ kubectl exec -it <pod-name> -- sh

试图登陆 docker 容器时出问题了，总是报错说

> error: unable to upgrade connection: pod does not exist

kubectl 登陆不了 docker 容器，而且  kubectl logs 也会报一样的错，必须到具体的工作节点上用 docker exec 或 docker logs 才能访问到该节点上的容器信息。

这就不太对头，网上找了下原因，结果是因为节点间通信时选错了 IP 地址。

比如三个 Vagrant 虚拟机分别是

1.  k8s-master (172.28.128.14)
2.  k8s-node1 (172.28.128.10)
3.  k8s-node2 (172.28.128.11)

在 k8s-master 中初始集群时用的命令也是指定的 172.28.128.14 IP 地址

> $ kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address 172.28.128.14

然后 k8s-node1 和  k8s-node2 用上面产生的 token 也正常加入了集群，连后面部署应用都能够到达两个工作节点上。但当时确实未注意 `kubectl get nodes -o wide`  的输出，现在看到了是下面的样子

|  | 

# kubectl get nodes -o wide

NAME         STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP  ...

k8s-master   Ready    master   95m   v1.18.0   10.0.2.15     <none>       ...

k8s-node1    Ready    <none>   93m   v1.17.4   10.0.2.15     <none>       ...

k8s-node2    Ready    <none>   93m   v1.17.4   10.0.2.15     <none>       ...

 |

查看任意一个节点看看

> $ kubectl get nodes k8s-node1 -o yaml  
> status:  
>     addresses:  
>     -  address: 10.0.2.15  
>         type: InternalIP  
>     -  address: k8s-node1

看到的 address 是 10.0.2.15. 那是否能用 `kubectl edit nodes k8s-node1` 修改后保存应用呢？可以改也能保存 (提示  node/k8s-node1 edited)，但再次查看又变化原样。

怎么三个节点的  INTERNA-IP 都是一样的啊，10.0.2.15, 这个 IP 从哪里来的，分别进到 k8s-master, k8s-node1 和  k8s-node2 三个节点，ifconfig 发现第三个位置的网络设备都是 10.0.2.15 这个 IP。以下是 k8s-master 中的 ifconfig 前部分输出, docker0 和 enp0s3 上的 IP 地址在三个节点上是一样的，而 kubeadm 在初始化和加入集群时恰恰就选取了第三个设备 enp0s3 上的 IP 地址，全部是一样的 10.0.2.15。

|  | 

cni0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450    // 在 k8s-node1 和 k8s-node2 中分别为 10.244.1.1, 10.244.2.1

        inet 10.244.0.1  netmask 255.255.255.0  broadcast 0.0.0.0  

......

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500

        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255

......

enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500

        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255

......

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500 // 在 k8s-node1 和 k8s-node2 中分别为 172.28.128.10, 172.28.128.11

        inet 172.28.128.14  netmask 255.255.255.0  broadcast 172.28.128.255

 |

所以造成了 kubectl exec 和 kubectl logs 无法工作。

解决办法

在所有的节点上，包括 master 和  worker 节点，做同样的事情

> # vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf  
> EnvironmentFile=-/etc/default/kubelet  
> Environment="KUBELET_EXTRA_ARGS=--node-ip=<各自 172.28.128.xx 段的 IP>"  
> ExecStart=

只需要加上高亮的那一行，然后再重启两个服务

> # systemctl daemon-reload  
> # systemctl restart kubelet

现在再来看下 `kubectl get nodes -o wide` 显示什么了

|  | 

# kubectl get nodes -o wide

NAME         STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP ...

k8s-master   Ready    master   3h41m   v1.18.0   172.28.128.14   <none>      ...

k8s-node1    Ready    <none>   3h39m   v1.17.4   172.28.128.10   <none>      ...

k8s-node2    Ready    <none>   3h38m   v1.17.4   172.28.128.11   <none>      ...

 |

INTERNAL-IP 都显示正常了，再来试下 `kubectl exec` 命令

> root@k8s-master:# kubectl exec -it python-web-app-68d7bbd7f5-k5mvc -- sh  
> / # hostname  
> python-web-app-68d7bbd7f5-dzjxf

链接：

1.  [How to specify Internal-IP for kubernetes worker node](https://medium.com/@kanrangsan/how-to-specify-internal-ip-for-kubernetes-worker-node-24790b2884fd)
2.  [Playing with kuberadm in Vagrant Machines, Part 2](https://medium.com/@joatmon08/playing-with-kubeadm-in-vagrant-machines-part-2-bac431095706)