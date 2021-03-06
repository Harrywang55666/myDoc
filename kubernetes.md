

#  场景：使用官方文档中的yaml创建statefulset的时候出错

https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

错误：pod has unbound PersistentVolumeClaims

原因：As the cluster used in this tutorial is configured to dynamically provision PersistentVolumes, the PersistentVolumes were created and bound automatically.

解决方法：

去掉yaml中pv的创建部分,k8s会自动分配pv：

```
 volumeMounts:
        - name: www
         mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

#  kubectl delete pod <pod-name> 删除不了

需要启动 pod 所在的 node

# 挂载 nfs 提供的 volume 出错

`kubectl describe po [podname]`可看到如下输出：

```
...
  Warning  FailedMount  2m    kubelet, ubuntu3   MountVolume.SetUp failed for volume "pvc-09cf7a8a-c237-11e8-bbbf-0050568693f8" : mount failed: exit status 32
Mounting command: systemd-run
Mounting arguments: --description=Kubernetes transient mount for /var/lib/kubelet/pods/09db4e8e-c237-11e8-bbbf-0050568693f8/volumes/kubernetes.io~nfs/pvc-09cf7a8a-c237-11e8-bbbf-0050568693f8 --scope -- mount -t nfs 10.10.12.27:/nfs/default-mysql01-pv-claim-pvc-09cf7a8a-c237-11e8-bbbf-0050568693f8 /var/lib/kubelet/pods/09db4e8e-c237-11e8-bbbf-0050568693f8/volumes/kubernetes.io~nfs/pvc-09cf7a8a-c237-11e8-bbbf-0050568693f8
Output: Running scope as unit run-rbc74e570849941fe9a9355219e82ba5d.scope.
mount: wrong fs type, bad option, bad superblock on 10.10.12.27:/nfs/default-mysql01-pv-claim-pvc-09cf7a8a-c237-11e8-bbbf-0050568693f8,
       missing codepage or helper program, or other error
       (for several filesystems (e.g. nfs, cifs) you might
       need a /sbin/mount.<type> helper program)

       In some cases useful info is found in syslog - try
       dmesg | tail or so.
```

到该 pod 所在的 node 机器上（这里是 ubuntu3）上安装 nfs-common：

`apt install nfs-common`

# 开启 apiserver http 代理

在 master 上：

`kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$' --port=8080`

在其他机器上：

`kubectl get node -s http://[master_ip]:8080`

# `kubectl delete pvc [pvc-name]`出错

`kubectl get pvc` 

```
mysql01-pv-claim                           Terminating   pvc-c32e14a7-f8f9-11e8-b1c0-0050568693f8   5Gi        RWO            managed-nfs-storage        1d
```

一直阻塞在`Terminating`状态

`kubectl edit pvc mysql01-pv-claim`

删除`finalizers`字段

# Kubernetes Components

https://kubernetes.io/docs/concepts/overview/components/

Master Components 控制平面:

```
kube-apiserver: 暴露 Kubernetes API，控制平面的前端
etcd：持久、高可用的key-value 存储，存储所有的集群数据
kube-scheduler：关注新创建的pod，并负责把pod调度到node上
kube-controller-manager：控制器的集合，包括Node Controller，Replication Controller，Endpoints Controller，Service Account & Token Controllers
cloud-controller-manager：由云供应商实现
```

Node Components 工作节点:

```
kubelet：确保container在pod中运行
kube-proxy：维护容器网络和转发
Container Runtime：
```

Addons 插件，实现了集群功能的pods和services:

```
DNS
Web UI (Dashboard)
Container Resource Monitoring
Cluster-level Logging
```

# 容器开放接口规范（CRI OCI CNI）

CRI - Container Runtime Interface(容器运行时接口)

CNI - Container Network Interface(容器网络接口)

CSI - Container Storage Interface(容器存储接口)

OCI - Open Container Initiative

# 使用 `kubectl port-forward`的时候失败

背景：

https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/installation.md

通过端口转发访问 nginx ingress controllor

命令：

`$ kubectl port-forward <nginx-ingress-pod> 8080:8080 --namespace=nginx-ingress`

错误：

```
E1217 20:46:55.531317   18752 portforward.go:331] an error occurred forwarding 8080 -> 8080: error forwarding port 8080 to pod 0e20166936ce1d0988aa01a3928c264813579f324b5ff939c057557b7236d177, uid : unable to do port forwarding: socat not found.
```

解决方法：

宿主机上安装 socat

`apt install socat`

