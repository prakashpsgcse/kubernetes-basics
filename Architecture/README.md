#K8S Architecture
-> Mater node and slave node 
-> 4 services on master node 
#Master (Control Plane)
-> manages cluster state/nodes/objects 
-> major components: kube-apiserver, kube-controller-manager and kube-scheduler
-> all those components can run in single mastr node 
-> It can be replicated using multiple master node 
#### API server 
   -> interface for intracting with UI 
   -> authendicatio & authorization 
   -> kubectl / rest api will talk to this server
   -> 
#### Schedular 
-> API server will send all req after validation to scheduler 
-> schedule pod in worker node 
-> check resource in worker nodes 
-> find least busy / more resource available schedule pod here 
-> this will ask kublet in worker to to start pod 
-> checks conditions and fins a right pod (pod affinity/node affinity etc)
-> Node meets Pod req are called feasible nodes
-> Schedular fins this and notifies API server is called binding
#### Controler manager
-> monitor workload(pod/service) etc 
-> detect state changes like crashed/deleted pod 
-> try to recover by asking scheduler to redeploy pod 
-> multiple controller for seperation of duties 
-> **Node Controller:** - Noticing and responding when nodes go down
-> **Replication controler:** - Maintaining the correct number of pods
-> **Endpoints controller:** Populates the Endpoints(Service) object (maps Services & Pods)
-> **Service Account & Token controllers:** Create default accounts and API access tokens
#### etcd 
-> distributed kry/value store 
-> all info about pods ,services ,etc is stored in KV 
-> K8S DB 
-> all k8s configs / cluster date 
-> k8s made up with multiple etcd (distributed)
####Kubelet
-> "node agent" that runs on each node
-> it will register node in API Server
-> get PodSpec fromAPI Server and runs container in it 
-> 
-> loading container 
-> sending report of node / pod 
-> AGENt on each node listen to schedular 
-> managers container in node 
##NODE
-> physical/ virtual machine
-> runs containers 
-> contains kubelet , kube-proxy and container runtime 
##kube-proxy 
-> Kubernetes network proxy on each node 
->  implementing part of the Kubernetes Service concept.
-> forwards traffic 

###Container runtime 
-> responsible for running containers
-> multiple runtimes supported 
    1. Containerd
    2. Docker 
    3. CRI-O

