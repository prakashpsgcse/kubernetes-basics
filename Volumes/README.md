# Volumes 
-> volume is a directory which is accessible to all of the containers in a Pod
-> Volume is Associated to pod but MOUNTED to containers in pod (same of diff path)  
-> On-disk files in a container are ephemeral (lost when container dies)  
-> Sharing files between containers running together in a Pod 
-> files within a container are inaccessible to other containers running in the same Pod
-> K8s supports Ephemeral volume types (lifetime of a pod) and persistent volumes (exists after pod life)  
-> volume is just a directory

-> Specify volume info in _.spec.volumes_ and mount it in _.spec.containers[*].volumeMounts_  

# Types of Volumes
-> k8s supports most of the cloud providers block storage 
-> first create storage and mount it 
-> supports AWS/Azure/OpenStack
## awsElasticBlockStore (AWS k8s deployments)
-> First Create EBS volume in AWS (aws ec2 create-volume)
-> Instance must be on same region as EBS & single instance mounting 
```shell
volumes:
  - name: test-volume
    # This AWS EBS volume must already exist.
    awsElasticBlockStore:
      volumeID: "<volume id>"
      fsType: ext4
```
## configMap 
-> Mount/Inject CM as volume 
-> name of the ConfigMap in the volume
-> 
```yaml
volumeMounts:
        - name: apps-props
          mountPath: /etc/config
  volumes:
    - name: apps-props
      configMap:
        name: application-props
          items:
          - key: log_level
            path: log_level
```
## emptyDir
-> Initially Empty   
-> Created when pod is assigned to node   
-> Exists as long as that Pod is running on that node  
-> Can be shared b/w pods in same node 
-> All pods can READ/WRITE same file
-> deleted when pod is removed from node (all of them ?)
->  volumes are stored on whatever medium that backs the node such as disk or SSD, or network storage
-> emptyDir.medium field to "Memory", Kubernetes mounts a tmpfs (RAM-backed filesystem) 
-> tmpfs is very fast and  cleared on node reboot
```yaml
volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

## gcePersistentDisk (GCP disk)

## hostPath 
-> mounts a file or directory from the host node's filesystem  
-> Might need root permission for container to write 
```yaml
  volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```

## NFS
-> Mounts NFS shares   
-> Contents of an nfs volume are preserved and the volume is merely unmounted  
-> NFS volume can be pre-populated with data, and that data can be shared between pods  

# PersistentVolumeClaim
-> This also volume 
-> used to mount a PersistentVolume into a Pod  
-> helps to claim durable storage without knowing the details of the particular cloud environment  
# projected
-> mapping of existing source as volumes  
-> mapping CM , secret , service account token to pod 
```yaml
volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
      - secret:
          name: mysecret2
          items:
            - key: password
              path: my-group/my-password
              mode: 511
```
# secret
-> used to pass sensitive information, such as passwords, to Pods  
-> create a secret and mount it using projected volume

# Persistent Volume (PV)
-> Storage provisioned by admin  
-> backed by a persistent disk (EBS,GCE,NFS)
-> PersistentVolume lifecycle is managed by Kubernetes ????  
-> can be dynamically provisioned  (by PersistentVolumeClaims)
-> exist independently of Pods (Disk & data Exists after pod deleted)
-> provisioned dynamically through PersistentVolumeClaims, or  created by a cluster admin
