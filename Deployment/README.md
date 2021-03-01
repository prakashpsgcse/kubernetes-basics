# Deployment
-> Deployments are managed by the Kubernetes Deployment controller  
-> Provides declarative(application life cycle state - images , num of replicas etc ) updates for Pods and ReplicaSets
-> Using Deployment we can tell k8s how to create/modify app
-> Ensures desired num of replicas (Deployment Controller)
-> Kubernetes deployment controller is always monitoring the health of pods and nodes, it can replace a failed pod or bypass down nodes
-> Represent a set of multiple, identical Pods with no unique identities
-> Deploy replica set or pod & update them 
-> Scale up/down pods and rollback previous version 
-> Replaces any instances that fail or become unresponsive
-> works well for Stateless app with ReadOnlyMany or ReadWriteMany volumes
-> not well-suited for workloads that use ReadWriteOnce volumes
-> Uses Pod Template & Spec inside to define pod 
-> Runs on default node pool(group of node without label) (create node lables like streaming / api and use nodeSelector)
-> each pod can run one or more containers 
###Deployment file
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {name of the deployment }
  labels:
#    key: value
    app: api

spec:
  replicas: {num replicas}
  selector:
    matchLabels: #used when mapping pods with deployment/service etc
      app: nginx
  template:
    metadata:
      labels:
#        key: val
        app: nginx
    spec:
      containers:
      - name: {name of the container }
        image: { image with version or with full URL }
        ports:
        - containerPort: {port -P in docker run }

```
-> .spec.selector is a required field that specifies a label selector for the Pods targeted by this Deployment  
-> .spec.selector must match .spec.template.metadata.labels, or it will be rejected by the API  
-> .spec.strategy.type can be "Recreate" or "RollingUpdate"(default)
## K8s commands
```shell
kubectl get deployments
kubectl rollout status deployment/nginx-deployment
kubectl get rs
kubectl get pods --show-labels
```
## Updating a Deployment
-> When .spec.template element is changed
-> can be done in 
```shell
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1 --record
kubectl edit deployment
```
-> when updating k8s will keep 75%(available) of replicas and update 25% (max unavailable)  
-> lets say we have app v1 with 4 replicas 
-> when we update k8s willnot kill all 4 at same time and get v2 app
-> to maintain availability (75% and 25%) it kills one pod and gets new pod with v2
-> First create new pod with v2 if its healthy then delete one in old (terminationGracePeriod=30 sec def)
-> container willbe in terminating state but it will terminate after 30 sec 
-> LB will not send trafic to terminating gracefull is to finish existing req
-> max available pods (125%) due to create and delete


```shell
[root@localhost prakash]# kubectl describe deployment runit
Name:                   runit
Namespace:              default
CreationTimestamp:      Mon, 01 Mar 2021 20:10:22 +0530
Labels:                 app=runit
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=runit
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=runit
  Containers:
   runit:
    Image:      cust
    Port:       80/TCP
    Host Port:  0/TCP
    Environment Variables from:
      runit-env-cm1  ConfigMap  Optional: false
    Environment:
      KAFKA_BROKER_URL:  <set to the key 'kafkabrokers' of config map 'runit-env-cm'>  Optional: false
      Stream_threads:    <set to the key 'threads' of config map 'runit-env-cm'>       Optional: false
    Mounts:
      /etc/config from server-properties-from-cm (rw)
  Volumes:
   server-properties-from-cm:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      runit-env-file
    Optional:  false
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   runit-7cf8dcdbd5 (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m13s  deployment-controller  Scaled up replica set runit-7d674dfb7d to 3
  Normal  ScalingReplicaSet  52s    deployment-controller  Scaled up replica set runit-7cf8dcdbd5 to 1
  Normal  ScalingReplicaSet  51s    deployment-controller  Scaled down replica set runit-7d674dfb7d to 2
  Normal  ScalingReplicaSet  51s    deployment-controller  Scaled up replica set runit-7cf8dcdbd5 to 2
  Normal  ScalingReplicaSet  50s    deployment-controller  Scaled down replica set runit-7d674dfb7d to 1
  Normal  ScalingReplicaSet  50s    deployment-controller  Scaled up replica set runit-7cf8dcdbd5 to 3
  Normal  ScalingReplicaSet  48s    deployment-controller  Scaled down replica set runit-7d674dfb7d to 0
```

-> To check rollout status 
```shell
[root@localhost prakash]# kubectl rollout status deployment/runit
Waiting for deployment "runit" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "runit" rollout to finish: 1 old replicas are pending termination...
deployment "runit" successfully rolled out
```

->if firstpod fails then it will not start next (read readiness/liveness) ???

-> to rollback update previous images and trigger or use UNDO
```shell
kubectl rollout undo {name}
kubectl rollout undo {name} --to-revision={revision number}
------------------------------------
kubectl rollout undo deployment/runit
kubectl rollout undo deployment/runit --to-revision=2
```
#### Checking history 
```shell
kubectl rollout history {name}
kubectl rollout history {name} --revision=2 -> to check what is changed
---------------------------------------
[root@localhost prakash]# kubectl rollout history deployment/runit
deployment.apps/runit 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

[root@localhost prakash]# kubectl rollout history deployment/runit --revision=2
deployment.apps/runit with revision #2
Pod Template:
  Labels:	app=runit
	pod-template-hash=7cf8dcdbd5
  Containers:
   runit:
    Image:	cust
    Port:	80/TCP
    Host Port:	0/TCP
    Environment Variables from:
      runit-env-cm1	ConfigMap	Optional: false
    Environment:
      KAFKA_BROKER_URL:	<set to the key 'kafkabrokers' of config map 'runit-env-cm'>	Optional: false
      Stream_threads:	<set to the key 'threads' of config map 'runit-env-cm'>	Optional: false
    Mounts:
      /etc/config from server-properties-from-cm (rw)
  Volumes:
   server-properties-from-cm:
    Type:	ConfigMap (a volume populated by a ConfigMap)
    Name:	runit-env-file
    Optional:	false

```
# Scaling
-> scale up and down  
 ```shell
kubectl scale deployment/{name} --replicas={count}
---------------------------------------------------
kubectl scale deployment/runit --replicas=4
```
#Autoscaling with CPU
```shell
kubectl autoscale deployment/{name} --min=10 --max=15 --cpu-percent=80
```
# Pausing and Resuming
```shell
kubectl rollout pause deployment/{name}
kubectl rollout resume deployment/{name}
```
## Failed Deployment
Insufficient quota
Readiness probe failures
Image pull errors
Insufficient permissions
Limit ranges
Application runtime misconfigurati

# Watch
-> watch any k8s object 
```shell
kubectl get pod --watch
kubectl get rs --watch
```
```shell
[root@localhost Deployment]# kubectl get pod --watch
NAME                     READY   STATUS    RESTARTS   AGE
runit-7d674dfb7d-f7mvm   1/1     Running   0          7m2s
runit-7d674dfb7d-sr7j6   1/1     Running   0          7m2s
runit-7d674dfb7d-wp2v9   1/1     Running   0          7m2s
runit-7d674dfb7d-f7mvm   1/1     Terminating   0          7m17s
runit-7d674dfb7d-q2brp   0/1     Pending       0          0s
runit-7d674dfb7d-q2brp   0/1     Pending       0          0s
runit-7d674dfb7d-q2brp   0/1     ContainerCreating   0          0s
runit-7d674dfb7d-q2brp   1/1     Running             0          1s
runit-7d674dfb7d-f7mvm   0/1     Terminating         0          7m49s
runit-7d674dfb7d-f7mvm   0/1     Terminating         0          8m
runit-7d674dfb7d-f7mvm   0/1     Terminating         0          8m

```
