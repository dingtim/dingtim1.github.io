---
layout: post
categories: Kubernetes
title: 使用kubeadm安装Kubernetes 1.6
date: 2017-4-12 11:36:51 +0800
description: 使用kubeadm安装Kubernetes 1.6
keywords: Kubernetes
---


使用kubeadm安装Kubernetes 1.6

环境准备
1.	192.168.253.200 iov-253-200.supower.tech
2.	192.168.253.202 iov-253-202.supower.tech
3.	192.168.253.203 iov-253-203.supower.tech
安装Docker 1.12
Kubernetes 1.6还没有针对docker 1.13和最新的docker 17.03上做测试和验证，所以这里安装Kubernetes官方推荐的Docker 1.12版本。
1.	yum install -y yum-utils
2.	
3.	yum-config-manager \\
4.	    --add-repo \\
5.	    https://docs.docker.com/v1.13/engine/installation/linux/repo_files/centos/docker.repo
6.	
7.	yum makecache fast
查看版本：
1.	yum list docker-engine.x86_64  --showduplicates |sort -r
2.	docker-engine.x86_64             1.13.1-1.el7.centos                 docker-main
3.	docker-engine.x86_64             1.12.6-1.el7.centos                 docker-main
4.	docker-engine.x86_64             1.11.2-1.el7.centos                 docker-main
安装1.12.6：
1.	yum install -y docker-engine-1.12.6
2.	
3.	systemctl start docker
4.	systemctl enable docker
系统配置
根据官方文档Installing Kubernetes on Linux with kubeadm中的Limitations小节中的内容，对各节点系统做如下设置:
创建/etc/sysctl.d/k8s.conf文件，添加如下内容：
1.	net.bridge.bridge-nf-call-ip6tables = 1
2.	net.bridge.bridge-nf-call-iptables = 1
执行sysctl -p /etc/sysctl.d/k8s.conf使修改生效。
在/etc/hostname中修改各节点的hostname，在/etc/hosts中设置hostname对应非lo回环网卡ip:
1.	192.168.253.200 iov-253-200.supower.tech
2.	192.168.253.202 iov-253-202.supower.tech
3.	192.168.253.203 iov-253-203.supower.tech
安装kubeadm和kubelet
下面在各节点安装kubeadm和kubelet：
1.	cat \<\<EOF \> /etc/yum.repos.d/kubernetes.repo
2.	[kubernetes](#)
3.	name=Kubernetes
4.	baseurl=http://yum.kubernetes.io/repos/kubernetes-el7-x86_64
5.	enabled=1
6.	gpgcheck=1
7.	repo_gpgcheck=1
8.	gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
9.	        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
10.	EOF
测试地址http://yum.kubernetes.io/repos/kubernetes-el7-x86_64是否可用，如果不可用需要科学上网。
1.	 curl http://yum.kubernetes.io/repos/kubernetes-el7-x86_64
查看kubeadm, kubelet, kubectl, kubernets-cni的最新版本：
1.	yum list kubeadm  --showduplicates |sort -r
2.	kubeadm.x86_64                        1.6.1-0                        kubernetes
3.	kubeadm.x86_64                        1.6.0-0                        kubernetes
4.	
5.	yum list kubelet  --showduplicates |sort -r
6.	kubelet.x86_64                        1.6.1-0                        kubernetes
7.	kubelet.x86_64                        1.6.0-0                         kubernetes
8.	kubelet.x86_64                        1.5.4-0                         kubernetes
9.	
10.	
11.	yum list kubectl  --showduplicates |sort -r
12.	kubectl.x86_64                        1.6.1-0                        kubernetes
13.	kubectl.x86_64                        1.6.0-0                         kubernetes
14.	kubectl.x86_64                        1.5.4-0                         kubernetes
15.	
16.	
17.	yum list kubernets-cni  --showduplicates |sort -r
18.	kubernetes-cni              x86_64              0.5.1-0                    kubernetes
kubeadm和kubelet已经是1.6.1，就是我们要安装的版本，直接安装即可：
1.	setenforce 0
2.	
3.	yum install -y kubelet kubeadm kubectl kubernetes-cni
4.	...
5.	Installed:
6.	  kubeadm.x86_64 0:1.6.1-0               kubectl.x86_64 0:1.6.1-0        kubelet.x86_64 0:1.6.1-0
7.	  kubernetes-cni.x86_64 0:0.5.1-0
8.	
9.	Dependency Installed:
10.	  ebtables.x86_64 0:2.0.10-15.el7                       socat.x86_64 0:1.7.2.2-5.el7
11.	
12.	Complete!
1.	systemctl enable kubelet.service
初始化集群
接下来使用kubeadm初始化集群，选择iov-253-200.supower.tech作为Master Node，在iov-253-200.supower.tech上执行下面的命令：
kubeadm init --apiserver-advertise-address 192.168.253.253.200 --service-cidr 10.20.0.0/23 --kubernetes-version stable注意到kubeadm init的--kubernetes-version 默认 1.6.0 stable（1.6.1）
--apiserver-advertise-address发生了变化，之前kuebadm 1.6.0-0.alpha是--use-kubernetes-version和--api-advertise-addresses。
在集群初始化遇到问题，可以使用下面的命令进行清理后重新再初始化：
1.	kubeadm reset
kubeadm init执行成功后输出下面的信息：
[kubeadm](#) WARNING: kubeadm is in beta, please do not use it for production clusters.
[init](#) Using Kubernetes version: v1.6.1
[init](#) Using Authorization mode: RBAC
[preflight](#) Running pre-flight checks
[preflight](#) Starting the kubelet service
[certificates](#) Generated CA certificate and key.
[certificates](#) Generated API server certificate and key.
[certificates](#) API Server serving cert is signed for DNS names [iov-253-200.supower.tech kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local](#) and IPs [10.20.0.1 192.168.253.200](#)
[certificates](#) Generated API server kubelet client certificate and key.
[certificates](#) Generated service account token signing key and public key.
[certificates](#) Generated front-proxy CA certificate and key.
[certificates](#) Generated front-proxy client certificate and key.
[certificates](#) Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig](#) Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig](#) Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig](#) Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig](#) Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[apiclient](#) Created API client, waiting for the control plane to become ready
[apiclient](#) All control plane components are healthy after 224.271821 seconds
[apiclient](#) Waiting for at least one node to register
[apiclient](#) First node has registered after 6.664066 seconds
[token](#) Using token: 282f36.4d66e2cd9cf99611
[apiconfig](#) Created RBAC rules
[addons](#) Created essential addon: kube-proxy
[addons](#) Created essential addon: kube-dns

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  sudo cp /etc/kubernetes/admin.conf $HOME/
  sudo chown $(id -u):$(id -g) $HOME/admin.conf
  export KUBECONFIG=$HOME/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork](#).yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 282f36.4d66e2cd9cf99611 192.168.253.200:6443
Master Node初始化完成，使用kubeadm初始化的Kubernetes集群在Master节点上的核心组件：kube-apiserver,kube-scheduler, kube-controller-manager是以静态Pod的形式运行的。
1.	ls /etc/kubernetes/manifests/
2.	etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
在/etc/kubernetes/manifests/目录里可以看到kube-apiserver,kube-scheduler, kube-controller-manager的定义文件。另外集群持久化存储etcd也是以单点静态Pod的形式运行的，对于etcd后边我们会把它切换成etcd集群，这里暂且不表。
查看一下kube-apiserver.yaml的内容：
1.	apiVersion: v1
2.	kind: Pod
3.	metadata:
4.	  creationTimestamp: null
5.	  labels:
6.	    component: kube-apiserver
7.	    tier: control-plane
8.	  name: kube-apiserver
9.	  namespace: kube-system
10.	spec:
11.	  containers:
12.	  - command:
13.	    - kube-apiserver
14.	    .......
15.	    - --insecure-port=0
注意到kube-apiserver的选项--insecure-port=0，也就是说kubeadm 1.6.0初始化的集群，kube-apiserver没有监听默认的http 8080端口。所以我们使用kubectl get nodes会报The connection to the server localhost:8080 was refused - did you specify the right host or port?。
查看kube-apiserver的监听端口可以看到只监听了https的6443端口，
1.	netstat -nltp | grep apiserver
2.	tcp6       0      0 :6443                 :*                    LISTEN      9831/kube-apiserver
为了使用kubectl访问apiserver，在/.bashprofile中追加下面的环境变量：
export KUBECONFIG=/etc/kubernetes/admin.conf
source /.bashprofile
此时kubectl命令在master node上就好用了，查看一下当前机器中的Node：
1.	kubectl get nodes
2.	NAME      STATUS     AGE       VERSION
3.	node0     NotReady   3m        v1.6.1
安装Pod Network
接下来安装flannel network add-on：
1.	kubectl create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml
2.	kubectl apply -f  https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
3.	serviceaccount "flannel" created
4.	configmap "kube-flannel-cfg" created
5.	daemonset "kube-flannel-ds" created
如果Node有多个网卡的话，参考flannel issues 39701，目前需要在kube-flannel.yml中使用--iface参数指定集群主机内网网卡的名称，否则可能会出现dns无法解析。需要将kube-flannel.yml下载到本地，flanneld启动参数加上--iface=\<iface-name\>
1.	......
2.	apiVersion: extensions/v1beta1
3.	kind: DaemonSet
4.	metadata:
5.	  name: kube-flannel-ds
6.	......
7.	containers:
8.	      - name: kube-flannel
9.	        image: quay.io/coreos/flannel:v0.7.0-amd64
10.	        command: [ "/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr", "--iface=eth1" ](#)
11.	......
使用kubectl get pod --all-namespaces -o wide确保所有的Pod都处于Running状态。
1.	kubectl get pod --all-namespaces -o wide
使master node参与工作负载(可选)
使用kubeadm初始化的集群，出于安全考虑Pod不会被调度到Master Node上，也就是说Master Node不参与工作负载。
这里搭建的是测试环境可以使用下面的命令使Master Node参与工作负载：
1.	kubectl taint nodes --all  node-role.kubernetes.io/master-
测试DNS
1.	kubectl run curl --image=radial/busyboxplus:curl -i --tty
2.	Waiting for pod default/curl-2421989462-vldmp to be running, status is Pending, pod ready: false
3.	Waiting for pod default/curl-2421989462-vldmp to be running, status is Pending, pod ready: false
4.	If you don't see a command prompt, try pressing enter.
5.	[ root@curl-2421989462-vldmp:/ ](#)$
进入后执行nslookup kubernetes.default确认解析正常。
1.	[ root@curl-2421989462-vldmp:/ ](#)$ nslookup kubernetes.default
2.	Server:    10.96.0.10
3.	Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
4.	
5.	Name:      kubernetes.default
6.	Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
测试OK后，删除掉curl这个Pod。
1.	kubectl delete deploy curl
向集群中添加节点
下面将iov-253-202.supower.tech和iov-253-203.supower.tech加入集群，分别在iov-253-202.supower.tech和iov-253-203.supower.tech上执行：
1.	kubeadm join --token e7986d.e440de5882342711 192.168.253.200:6443
2.	[kubeadm](#) WARNING: kubeadm is in beta, please do not use it for production clusters.
3.	[preflight](#) Running pre-flight checks
4.	[discovery](#) Trying to connect to API Server "192.168.253.200:6443"
5.	[discovery](#) Created cluster-info discovery client, requesting info from "https://192.168.253.200:6443"
6.	[discovery](#) Cluster info signature and contents are valid, will use API Server "https://192.168.253.200:6443"
7.	[discovery](#) Successfully established connection with API Server "192.168.253.200:6443"
8.	[bootstrap](#) Detected server version: v1.6.1
9.	[bootstrap](#) The server supports the Certificates API (certificates.k8s.io/v1beta1)
10.	[csr](#) Created API client to obtain unique certificate for this node, generating keys and certificate signing request
11.	[csr](#) Received signed certificate from the API server, generating KubeConfig...
12.	[kubeconfig](#) Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
13.	
14.	Node join complete:
15.	* Certificate signing request sent to master and response
16.	  received.
17.	* Kubelet informed of new secure connection details.
18.	
19.	Run 'kubectl get nodes' on the master to see this machine join.
查看集群中节点：
1.	kubectl get nodes
2.	NAME      STATUS    AGE       VERSION
3.	iov-253-200.supower.tech     Ready     12m       v1.6.1
4.	iov-253-202.supower.tech     Ready     4m        v1.6.1
5.	iov-253-203.supower.tech     Ready     2m        v1.6.1


