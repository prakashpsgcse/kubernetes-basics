#K8S default namespaces
-> In k8s cluster  4 NS will be there by default 
````shell
kubectl get ns 
----------------------------------------
[root@localhost local_cluster_setup]# kubectl get ns
NAME                 STATUS   AGE
default              Active   21m
kube-node-lease      Active   21m
kube-public          Active   21m
kube-system          Active   21m
local-path-storage   Active   21m <-------------from kind 

````
##kube-system
-> not for user  
-> for system Objects  
##kube-public 
-> opened for all  
-> Readable by all users
-> not for user
##kube-node-lease 
-> node availability  
-> lease objects associated with each node
-> node performance via heardbeat 
#default
->all users objects are created   


#Sample objects in NS 
-> created cluster with one master 3 worker in kind 
```shell
[root@localhost local_cluster_setup]# kubectl get node -o wide
NAME                   STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION           CONTAINER-RUNTIME
sample-control-plane   Ready    control-plane,master   60m   v1.20.2   172.18.0.4    <none>        Ubuntu 20.10   3.10.0-1160.el7.x86_64   containerd://1.4.0-106-gce4439a8
sample-worker          Ready    <none>                 59m   v1.20.2   172.18.0.5    <none>        Ubuntu 20.10   3.10.0-1160.el7.x86_64   containerd://1.4.0-106-gce4439a8
sample-worker2         Ready    <none>                 59m   v1.20.2   172.18.0.2    <none>        Ubuntu 20.10   3.10.0-1160.el7.x86_64   containerd://1.4.0-106-gce4439a8
sample-worker3         Ready    <none>                 59m   v1.20.2   172.18.0.3    <none>        Ubuntu 20.10   3.10.0-1160.el7.x86_64   containerd://1.4.0-106-gce4439a8

```
-> public/node-lease is empty 
#kube-system NS 

###Pods in kube system 
```shell
[root@localhost local_cluster_setup]# kubectl get pod -o wide --namespace=kube-system
NAME                                           READY   STATUS    RESTARTS   AGE   IP           NODE                   NOMINATED NODE   READINESS GATES
coredns-74ff55c5b-cb682                        1/1     Running   0          72m   10.244.0.4   sample-control-plane   <none>           <none>
coredns-74ff55c5b-dspsw                        1/1     Running   0          72m   10.244.0.3   sample-control-plane   <none>           <none>
etcd-sample-control-plane                      1/1     Running   0          72m   172.18.0.4   sample-control-plane   <none>           <none>
kindnet-5nqq4                                  1/1     Running   0          72m   172.18.0.3   sample-worker3         <none>           <none>
kindnet-8j4kw                                  1/1     Running   0          72m   172.18.0.2   sample-worker2         <none>           <none>
kindnet-r5bfj                                  1/1     Running   0          72m   172.18.0.5   sample-worker          <none>           <none>
kindnet-t5fcg                                  1/1     Running   0          72m   172.18.0.4   sample-control-plane   <none>           <none>
kube-apiserver-sample-control-plane            1/1     Running   0          72m   172.18.0.4   sample-control-plane   <none>           <none>
kube-controller-manager-sample-control-plane   1/1     Running   0          72m   172.18.0.4   sample-control-plane   <none>           <none>
kube-proxy-4rqrk                               1/1     Running   0          72m   172.18.0.5   sample-worker          <none>           <none>
kube-proxy-hgv8j                               1/1     Running   0          72m   172.18.0.4   sample-control-plane   <none>           <none>
kube-proxy-qmzmb                               1/1     Running   0          72m   172.18.0.2   sample-worker2         <none>           <none>
kube-proxy-scxx8                               1/1     Running   0          72m   172.18.0.3   sample-worker3         <none>           <none>
kube-scheduler-sample-control-plane            1/1     Running   0          72m   172.18.0.4   sample-control-plane   <none>           <none>
```
###Service in kube sys
```shell
[root@localhost local_cluster_setup]# kubectl get service  -o wide --namespace=kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   82m   k8s-app=kube-dns
```
### daemonset in kube sys
```[root@localhost local_cluster_setup]# kubectl get daemonset  -o wide --namespace=kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE   CONTAINERS    IMAGES                                SELECTOR
kindnet      4         4         4       4            4           <none>                   84m   kindnet-cni   kindest/kindnetd:v20210119-d5ef916d   app=kindnet
kube-proxy   4         4         4       4            4           kubernetes.io/os=linux   84m   kube-proxy    k8s.gcr.io/kube-proxy:v1.20.2         k8s-app=kube-proxy
```

#PODS 
####