#Local cluster setup with kind
-> To create single node cluster 
```shell
kind create cluster --name {name}
--------------------------
kind create cluster --name sample
[root@localhost k8s]# kubectl get nodes
NAME                   STATUS   ROLES                  AGE     VERSION
sample-control-plane   Ready    control-plane,master   2m13s   v1.20.2
```
-> To delete cluster 
```shell
kind delete cluster {name}
-----------------------------------
kind create cluster --name sample
```

-> to check available docker images in kind cluster 
```shell
docker exec -it {master-docker-id} crictl images
```
-> To load local docker image into kind cluster 
```shell
kind load docker-image {image-name} --name {kind-cluster-name}
--------------------------------------------------------------
kind load docker-image runit --name sample
```

-> To create 3 worker cluster 
```shell
kind create cluster --config {config}.yaml --name {kind-cluster-name}
---------------------------------------------------------------------
kind create cluster --config kind-cluster-config.yaml --name sample
------------------------------------------------------------------
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```
-> Set imagePullPolicy: Never 

-> To get cluster info 
```shell
kubectl cluster-info  
-------------------------------------------------------
Kubernetes control plane is running at https://127.0.0.1:37617
KubeDNS is running at https://127.0.0.1:37617/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```