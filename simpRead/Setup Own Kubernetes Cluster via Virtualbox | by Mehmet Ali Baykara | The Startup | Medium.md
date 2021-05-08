> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [medium.com](https://medium.com/swlh/setup-own-kubernetes-cluster-via-virtualbox-99a82605bfcc)

Setup Own Kubernetes Cluster via Virtualbox
===========================================



The objective of this post will set up three nodes Kubernetes(K8S) cluster on VirtualBox and launch an application/nginx.

What you will learn

*   Using vagrant spin up multiple virtual machines
*   Install Kubernetes and set up Controller(master) and Workers
*   Launch an application scale up/down and expose it.
*   containerd as the container runtime. **_we will not use Docker_**


Tools
=====

*   [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
*   [containerd](https://containerd.io/) _will be installed via_ [_script_](https://github.com/mbaykara/k8s-cluster/blob/main/k8s-setup-master.sh)
*   [Kubernetes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime) _will be installed via_ [_script_](https://github.com/mbaykara/k8s-cluster/blob/main/k8s-setup-master.sh)
*   [Vagrant](https://www.vagrantup.com/)

please make sure on your host machine Vagrant and VirtualBox are installed.

For sure you can play around with Kubernetes via **minikube** as well, but to be honest it does not feel like a real use case for K8S. I mean minikube is easy, we love complicated things:)All resources are accessible on my GitHub repo [here](https://github.com/mbaykara/k8s-cluster.git). I have prepared some script which makes easy repeated tasks.

*   control node at least ~ 2GB RAM
*   Workers ca ~ 1GB RAM

```
$ git clone https://github.com/mbaykara/k8s-cluster.git
$ ls
k8s-setup-master.sh  README.md  Vagrantfile
$ vagrant up
```

I have defined one control node and two workers, have look Vagrantfile for further details. `vagrant up`` command will create `master` and `worker-1`, `worker-2`

![](https://miro.medium.com/max/900/1*TQlVqfeFejDzpfEE40KDXg.png)

The master node is ready we can ssh into the machine via `vagrant ssh master`

![](https://miro.medium.com/max/954/1*vRtXZkfKrwmaIIbNgBE-Qg.png)

From now on can initialize **kubeadm** on the master node to manage the workers as root.

run the following command as root, it will take a few mins

```
root@master:~# kubeadm init --apiserver-advertise-address 192.168.33.13 --pod-network-cidr=10.244.0.0/16
```

![](https://miro.medium.com/max/941/1*SJOVEo4UDGysKChAiS-Uyg.png)

When it is done, should looks as:

![](https://miro.medium.com/max/1017/1*PjBuctPApQKBoZGP7J68zg.png)
As we see that it’s relatively straightforward and explain what should be done in the next step.

Exit from the root and run the following command as an arbitrary user

```
$ mkdir -p $HOME/.kube
 $ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 $ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

If you now list nodes till now it is the only one which is master by `kubectl get nodes`` you will see, the `STATUS`` is not ready. Simply because we did not apply any network plugin.

![](https://miro.medium.com/max/705/1*JN49znuoZqacvD1MxMQchQ.png)
I will use **flannel** `net` plugin for the cluster network.

```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

![](https://miro.medium.com/max/1030/1*i3lRa-x2v-pFHHLGctTOHQ.png)
At this point the `Scheduler`and `Controller-panel` might be unhealthy.

```
$ kubectl get cs
```

![](https://miro.medium.com/max/949/1*uA7Am_nDhPREVzA1lFU30A.png)
Modify the following files on all master nodes:

```
$ sudo vim /etc/kubernetes/manifests/kube-scheduler.yaml
Clear the line (spec->containers->command) containing this phrase:  -- port=0
$ sudo vim /etc/kubernetes/manifests/kube-controller-manager.yaml
Clear the line (spec->containers->command) containing this phrase: --- port=0
$ sudo systemctl restart kubelet.service
```

![](https://miro.medium.com/max/940/1*8BOzFo6mU1cuCzHPa0uzbg.png)
Now master/controller node is ready, we can simply add workers.

```
step 1. vagrant ssh worker-1
step 2. run the command below as root
 kubeadm join 192.168.33.13:6443 --token bx86lo.agyszwr53ow5y53u \
    --discovery-token-ca-cert-hash sha256:536b10417f411de9ff9f11cb83d286f9217f5031845df93355b3a6a5ed96c066
```

Repeat both steps for each worker. Then go to the master node

![](https://miro.medium.com/max/1477/1*8MSXIcxP1QWaXrZt4z4QIg.png)

No, finally we are ready to deploy nginx! :D

I will deploy one Nginx service and 5 instances of the webserver

```
$ kubectl create deployment nginx --image=nginx --port 80
$ kubectl create deployment webserver --image=nginx --port 80 --replicas=5
```

To access from the internet we have to expose it as follow

```
vagrant@master:~$ kubectl expose deployment webserver --port 80 --type=NodePort
```

![](https://miro.medium.com/max/1376/1*3WSwnH8TbWtTVBIzVgfGww.png)

or via curl

![](https://miro.medium.com/max/902/1*QFLup4O8zJ9Sslgsaq2Vvg.png)

Note that webserver launched on worker-2 that's why you have to access it via worker-2 IP Address.

Troubleshooting
===============

1.  [By](https://v1-17.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) default, your cluster will not schedule Pods on the control-plane/master node for security reasons. If you want to be able to schedule Pods on the control-plane node, run:

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

2. For further issues using `kubectl describe nodes [nodeName]` will let you know where eventually the problem is.

```
Resources:
* https://v1-17.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
*https://github.com/coreos/flannel/blob/master/Documentation/troubleshooting.md
* https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
* https://github.com/mmumshad/kubernetes-the-hard-way
```

PS: In case you face any issue through this tutorial, leave a comment please, I will respond as soon as possible.
