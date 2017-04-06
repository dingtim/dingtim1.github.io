---
layout: post
categories: Kubernetes
title: ceph和kubernetes集成
date: 2017-3-27 18:36:51 +0800
description: ceph和kubernetes集成
keywords: Kubernetes
---




## 集成ceph和kubernetes

1、 禁用rbd features
	rbd image有4个 features，layering, exclusive-lock, object-map, fast-diff, deep-flatten
因为目前内核仅支持layering，修改默认配置
每个ceph node的/etc/ceph/ceph.conf 添加一行
rbd_default_features = 1
这样之后创建的image 只有这一个feature
验证方式：
ceph --show-config|grep rbd|grep features
rbd_default_features = 1

2 创建ceph-secret这个k8s secret对象，这个secret对象用于k8s volume插件访问ceph集群：
获取client.admin的keyring值，并用base64编码：

	# ceph auth get-key client.admin
AQBRIaFYqWT8AhAAUtmJgeNFW/o1ylUzssQQhA==

	# echo "AQBRIaFYqWT8AhAAUtmJgeNFW/o1ylUzssQQhA=="|base64
QVFCUklhRllxV1Q4QWhBQVV0bUpnZU5GVy9vMXlsVXpzc1FRaEE9PQo=
创建ceph-secret.yaml文件，data下的key字段值即为上面得到的编码值：

apiVersion: v1
kind: Secret
metadata:
  name: ceph-secret
data:
  key: QVFCUklhRllxV1Q4QWhBQVV0bUpnZU5GVy9vMXlsVXpzc1FRaEE9PQo=
创建ceph-secret:

	# kubectl create -f ceph-secret.yamlsecret "ceph-secret" created
	# kubectl get secret
NAME     TYPE    DATA      AGE
ceph-secret  Opaque   1       2d
default-token-5vt3n  kubernetes.io/service-account-token 3 106d
三、Kubernetes Persistent Volume和Persistent Volume Claim
概念：PV是集群的资源，PVC请求资源并检查资源是否可用
注意：以下操作设计到name的参数，一定要一致

3 创建disk image (以jdk保存到ceph举例)

	# rbd create jdk-image -s 1G
	# rbd info jdk-image
rbd image 'jdk-image':
        size 1024 MB in 256 objects
        order 22 (4096 kB objects)
        block_name_prefix: rbd_data.37642ae8944a
        format: 2
        features: layering
        flags:
        
4 创建pv(仍然使用之前创建的ceph-secret)
创建jdk-pv.yaml:
monitors: 就是ceph的mon，有几个写几个

apiVersion: v1
kind: PersistentVolume
metadata:
  name: jdk-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  rbd:
    monitors:
      - 192.168.200.15:6789
      - 192.168.200.1:6789
      - 192.168.200.2:6789
    pool: rbd
    image: jdk-image
    user: admin
    secretRef:
      name: ceph-secret
    fsType: xfs
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
执行创建操作：

	# kubectl create -f jdk-pv.yamlpersistentvolume "jdk-pv" created
	#kubectl get pv
NAME      CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM      REASON    AGE
ceph-pv   1Gi        RWO           Recycle         Bound       default/ceph-claim   1d
jdk-pv    2Gi        RWO           Recycle         Available            1m
3.3 创建pvc
创建jdk-pvc.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jdk-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
执行创建操作：

	# kubectl create -f jdk-pvc.yamlpersistentvolumeclaim "jdk-claim" created
	# kubectl get pvc
NAME     STATUS    VOLUME    CAPACITY   ACCESSMODES   AGE
ceph-claim   Bound     ceph-pv   1Gi    RWO       2d
jdk-claim    Bound     jdk-pv    2Gi    RWO       39s

5 创建挂载ceph rbd的pod:
创建 ceph-busyboxpod.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: ceph-busybox
spec:
  containers:
  - name: ceph-busybox
    image: busybox
    command: ["sleep", "600000"]
    volumeMounts:
    - name: ceph-vol1
      mountPath: /usr/share/busybox
      readOnly: false
  volumes:
  - name: ceph-vol1
    persistentVolumeClaim:
      claimName: jdk-claim
执行创建操作：

kubectl create -f ceph-busyboxpod.yaml




