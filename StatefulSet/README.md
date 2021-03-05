# StatefulSet 
-> for stateful apps  
-> Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods  
-> maintains a sticky identity for each pod 
-> These pods are created from the same spec, but are not interchangeable
-> each has a persistent identifier that it maintains across any rescheduling  
-> use storage volumes to provide persistence
-> Stable, unique network identifiers & persistent storage
-> Ordered, graceful deployment and scaling
-> Ordered, automated rolling updates
## Limitations
-> PV or PVC with storage class should be there
-> deleteing will not delete volume 
-> requires Headless Service
-> graceful termination of the pods in the StatefulSet scale to zero and delete
->
# Rolling Updates 
-> default strategy in stateful set 
-> RollingUpdate update strategy implements automated, rolling update for the Pods in a StatefulSet  
-> Same order as termination (higher to lower)
-> wait for new pod to become healthy & running before next

# Partitions
-> RollingUpdate update strategy can be partitioned, by specifying a .spec.updateStrategy.rollingUpdate.partition  
-> partition is specified ordinal that is greater than or equal to the partition will be updated
-> All Pods with an ordinal that is less than the partition will not be updated, and, even if they are deleted, they will be recreated at the previous version

# Forced Rollback 
-> when you update pod with (error in new version) sts will stop (broken state)
-> This needs manual intervention 
-> even if you update old image in template it will not work 
-> delete errored pod after updating image 

# Pod Management Policies

## OrderedReady 
-> default 
-> kill last node and update with new version , wait for it to healthy & running
-> it if its not then brokern state 

## Parallel
-> StatefulSet controller to launch or terminate all Pods in parallel
-> not wait for Pods to become Running and Ready 
-> This option only affects the behavior for scaling operations. Updates are not affected.

# using kind 

-> POD created order 0,1,2 
```shell
[root@localhost prakash]# kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
runit-0   1/1     Running   0          3m11s
runit-1   1/1     Running   0          3m9s
runit-2   1/1     Running   0          3m4s
```
-> Automatically created PV from PVC 
```shell
[root@localhost prakash]# kubectl get pv -o wide
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE   VOLUMEMODE
pvc-404f3af3-12c9-458b-9466-5c0714ae7cfd   1Gi        RWO            Delete           Bound    default/k8s-prakash-runit-0   standard                25m   Filesystem
pvc-d401ae2b-50d3-4a1d-991a-f1dce7a0d9b9   1Gi        RWO            Delete           Bound    default/k8s-prakash-runit-1   standard                22m   Filesystem
pvc-fe977b6b-15ae-464e-8490-97ce54a960d6   1Gi        RWO            Delete           Bound    default/k8s-prakash-runit-2   standard                22m   Filesystem
[root@localhost prakash]# kubectl get pvc -o wide
NAME                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
k8s-prakash-runit-0   Bound    pvc-404f3af3-12c9-458b-9466-5c0714ae7cfd   1Gi        RWO            standard       25m   Filesystem
k8s-prakash-runit-1   Bound    pvc-d401ae2b-50d3-4a1d-991a-f1dce7a0d9b9   1Gi        RWO            standard       22m   Filesystem
k8s-prakash-runit-2   Bound    pvc-fe977b6b-15ae-464e-8490-97ce54a960d6   1Gi        RWO            standard       22m   Filesystem
```

-> look at the same of PVC 
```shell
k8s-prakash-runit-0 , 1, 2
```
-> when we decribe the pod to check volume its is actually a CLAIM  
```shell
[root@localhost prakash]# kubectl describe pod runit-0
Name:         runit-0
Namespace:    default
Priority:     0
Node:         sample-worker3/172.18.0.5
Start Time:   Fri, 05 Mar 2021 17:04:45 +0530
Labels:       app=runit
              controller-revision-hash=runit-575cbb44cd
              statefulset.kubernetes.io/pod-name=runit-0
Annotations:  <none>
Status:       Running
IP:           10.244.3.4
IPs:
  IP:           10.244.3.4
Controlled By:  StatefulSet/runit
Containers:
  runit:
    Container ID:   containerd://5ed64882a4b98f5e438ee6cc7440b0fc72e704a96cff07a9ef87d3680e2ca3b6
    Image:          runit
    Image ID:       sha256:04eb6b11654009d5559e2f048aa34ec921495e75377637d48e9da73789791134
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 05 Mar 2021 17:04:46 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /prakash from k8s-prakash (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-jhcxj (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  k8s-prakash:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  k8s-prakash-runit-0
    ReadOnly:   false
  default-token-jhcxj:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-jhcxj
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  23m   default-scheduler  Successfully assigned default/runit-0 to sample-worker3
  Normal  Pulled     23m   kubelet            Container image "runit" already present on machine
  Normal  Created    23m   kubelet            Created container runit
  Normal  Started    23m   kubelet            Started container runit
```
-> Is it like claim-0 can attach to only POD-0 ???

## Updating image 

```shell
[root@localhost prakash]# kubectl get pod
NAME      READY   STATUS        RESTARTS   AGE
runit-0   1/1     Running       0          79s
runit-1   1/1     Running       0          41m
runit-2   0/1     Terminating   0          41m
[root@localhost prakash]# kubectl get pod
NAME      READY   STATUS              RESTARTS   AGE
runit-0   1/1     Running             0          82s
runit-1   1/1     Running             0          41m
runit-2   0/1     ContainerCreating   0          2s
[root@localhost prakash]# kubectl get pod
NAME      READY   STATUS        RESTARTS   AGE
runit-0   1/1     Running       0          83s
runit-1   1/1     Terminating   0          41m
runit-2   1/1     Running       0          3s
[root@localhost prakash]# kubectl get pod
NAME      READY   STATUS        RESTARTS   AGE
runit-0   1/1     Running       0          86s
runit-1   1/1     Terminating   0          41m
runit-2   1/1     Running       0          6s
[root@localhost prakash]# kubectl get pod
NAME      READY   STATUS        RESTARTS   AGE
runit-0   1/1     Running       0          92s
runit-1   0/1     Terminating   0          41m
runit-2   1/1     Running       0          12s
[root@localhost prakash]# kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
runit-0   1/1     Running   0          7m57s
runit-1   1/1     Running   0          8m57s
runit-2   1/1     Running   0          9m17s

```
## Ordinal Index
->  N replicas, each Pod in the StatefulSet will be assigned an integer ordinal  
-> from 0 up through N-1, that is unique over the Set.

## Stable N/W ID 
-> hostname of pod = {name}-{oridinal}
```shell
[root@localhost prakash]# kubectl exec -it runit-0 hostname
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
runit-0
[root@localhost prakash]# kubectl exec -it runit-1 hostname
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
runit-1
[root@localhost prakash]# kubectl exec -it runit-2 hostname
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
runit-2
```



