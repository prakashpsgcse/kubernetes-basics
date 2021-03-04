# Service
-> we cannot use pod ip to access bcoz of dynamic internal Ip
-> With a Service, you get a stable IP address that lasts for the life of the Service  
-> provides load balancing. Clients call a single, stable IP address, and their requests are balanced across the Pods
-> abstract way to expose set of Pods as a network service  
-> pod will get dynamic IP (POD will get deleteted and recreated at any time in Nodes)  
-> abstraction which defines a logical set of Pods and a policy by which to access them  
-> pod are grouped by Selectors.Pod to be a member of the Service, the Pod must have all of the labels specified in the selector  
-> Service object must be a valid DNS label name  
-> Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service  
-> Services allow your applications to receive traffic
->  service can map an incoming port to any targetPort  
-> Services can be defined with or without a selector
-> Service also provides LoadBalancing (RR ? )
-> multiport is possible (ports: must have name with mapping)
-> acts as loadbalance (random:sessionAffinity)
# Serivce file 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {service-name}
spec:
  selector:
#    key:value <- how service will find set of (same) pods/matchlabel
    app: {app-name}
  type: ClusterIP/NodePort//
  ports:
    - protocol: TCP
      port: {service port .80}
      targetPort: {pod/container port }
```
```shell
[root@localhost Service]# kubectl get service
NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes              ClusterIP   10.96.0.1     <none>        443/TCP   21m
prakash-runit-service   ClusterIP   10.96.216.8   <none>        80/TCP    6s
[root@localhost Service]# kubectl describe service prakash-runit-service
Name:              prakash-runit-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=prakash-runit
Type:              ClusterIP
IP Families:       <none>
IP:                10.96.216.8
IPs:               10.96.216.8
Port:              <unset>  80/TCP
TargetPort:        9092/TCP
Endpoints:         10.244.1.2:9092,10.244.2.2:9092,10.244.3.2:9092
Session Affinity:  None
Events:            <none>
[root@localhost Service]# kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
runit-856847875c-mgllz   1/1     Running   0          42s   10.244.2.2   sample-worker3   <none>           <none>
runit-856847875c-t625c   1/1     Running   0          42s   10.244.3.2   sample-worker    <none>           <none>
runit-856847875c-zrfpk   1/1     Running   0          42s   10.244.1.2   sample-worker2   <none>           <none>
```
-> When pod get deleted it will br removed from service endpoint 
-> When new Pod come with new IP it will be added in service 

# Ways of Exposing Service
-> ClusterIP (default)
-> NodePort
-> LoadBalancer
-> ExternalName
-> Headless Service

## ClusterIP (Within cluster access)
-> default type
-> Internal clients send requests to a stable internal IP address  
-> Stable IP address that is accessible from nodes in the cluster
-> Exposes a service which is only accessible from within the cluster
## NodePort (Outside cluster access)
-> Exposes the Service on the same port of each selected Node in the cluster using NAT
-> accessible from outside the cluster _<NodeIP>:<NodePort>_
-> each node’s IP at a static port  
-> A ClusterIP service is created automatically, and the NodePort service will route to it  
-> NodePorts are open ports on every cluster node
-> External clients call the Service by using the external IP address of a node along with the TCP port specified by nodePort
-> The request is forwarded to one of the member Pods on the TCP port specified by the targetPort
-> Internal clients can use clusterIP and port or internal ip & node port
-> type field to NodePort, the Kubernetes control plane allocates a port from a range specified by --service-node-port-range flag (default: 30000-32767).  
-> 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-np-service
spec:
  selector:
    app: products
    department: sales
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```
-> By default targetPort will be port (if uou mention only port then targetport
   also 80)
-> k8s will assign random port to nodePor  
-> If you want a specific port number, you can specify a value in the nodePort field. 
The control plane will either allocate you that port 
or report that the API transaction failed.  
```shell
kubectl get  service prakash-external-runit-service -o yaml
--------------------------------------------------------
spec:
  clusterIP: 10.96.195.226
  clusterIPs:
  - 10.96.195.226
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 31517
    port: 80
    protocol: TCP
    targetPort: 80

```
 -> lets say i have 3 nodes and node port so i can access service using all 3 nodes ?
-> all 3 nodes ip: node_port ???
-> then traffic will be routed to port in cluster ip and target port in POD/container
```shell
external IP address of one of the cluster nodes is 203.0.113.2.
 Then for the preceding example, the external client calls the Service at 203.0.113.2 on TCP port 32675.
  The request is forwarded to one of the member Pods on TCP port 8080.
 The member Pod must have a container listening on TCP port 8080.
```

**Cluster Ip service is automatically created and when you call NODE_IP:NODE_PORT then traffic is directly send to node and port the to cluster ip service
 in port then it will send it POD using targetport
 
 NODE PORT WILL HAVE CLUSTER IP** 

So port will be OPENED in ALL WORKED NODE and ANY node can receive traffic and forward it to service and service can send it to any pod 

-> NOT SECURE 

-> NodePort gives you the freedom to set up your own load balancing solution
## LoadBalancer
-> exposes the service externally using the load balancer of your cloud provider
-> create GCP/AWS LB 
-> Automated / manual ?
-> Ex in GCP (GKE) Google Cloud controller creates LB in Project
-> load balancer has a stable IP address that is accessible from outside
-> The external load balancer routes to your NodePort and ClusterIP services, which are created automatically  
-> 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: prakash-lb-service
spec:
  selector:
    app: runit
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```
```shell
spec:
  clusterIP: 10.11.242.115
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32676
    port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: runit
  type: LoadBalancer
  status:
    loadBalancer:
      ingress:
      - ip: 100.10.01.233   <- LB 
```
clients call 100.10.01.233:80 and forwarded to pod running in 8080

Internal LB vs External LB ?
## ExternalName
-> ExternalName map a Service to a DNS name
-> provides an internal alias for an external DNS name
-> Internal clients make requests using the internal DNS name, and the requests are redirected to the external name
```yaml
apiVersion: v1
kind: Service
metadata:
  name: prakash-kafka
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```
# Headless Service
-> how to connect to pod directly ?  
-> Pod to Pod direct communication without service  
-> Required for StateFull apps   
-> Pod are not Identical (Redis,Zk ,Mongo like only matser can write)  
-> How client to know POds Ip address ?
-> DNS lookup (cluster ip) : client perform dns lookup k8s returns ip of cluster ip 
-> Set Spec.clusterIP=None then DNS lookup will return pod ID 
-> To create Headless service set Spec.clusterIP=None and service will not have cluster ip 
-> we can have both cluster ip and headless service for same pod 
-> whoever want to load balance they can call clusterip ()kafka initial connection
-> whovever wnt to talk directly they can (client producer to leader directly)

```shell
[root@localhost Service]# kubectl get service
NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
prakash-headless-service         ClusterIP   None            <none>        80/TCP         6s
prakash-runit-service            ClusterIP   10.96.216.8     <none>        80/TCP         18h
[root@localhost Service]# kubectl describe service prakash-headless-service
Name:              prakash-headless-service
Selector:          app=prakash-runit
Type:              ClusterIP
IP Families:       <none>
IP:                None
IPs:               None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.2:80,10.244.3.2:80,10.244.3.3:80
```