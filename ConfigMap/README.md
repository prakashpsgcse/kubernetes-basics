# ConfigMap 
-> API Object to store non-confidential data    
-> Stores in key-value pairs   
-> it can store data as files   
-> Can be used as   
     1. Env variables   
     2. Cmd line args  
     3. files in volume   
     4. Use K8s API to read  

# Commands 
```shell
kubectl get cm 
kubectl get cm {name} -o yaml
kubectl describe cm {name}
kubectl edit cm {cm-name}
kubectl delete cm {cm-name}

kubectl apply -f {yaml file}
kubectl create configmap {cm-name} --from-file {yaml file}
----------------------------------------------
kubectl create configmap prakash-cm --from-file cm.yaml
```
## ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: { CM name}
data:
#key value as CM
#   "key":"value"
   "kafkabrokers":"localhost:9092"
   "threads":5
#file as config map
#   {filename}: |
#      key=val
   server.properties: |
     kafkabrokers=localhost:9092
     num.stream.threads=10
```
## CM as Env variables 
-> add in container section  
-> create key and get the values in CM (env:)  
-> Attach full CM as env var (envFrom)  
-> Attach file(CM) as Volume  
```yaml
    spec:
      containers:
        - name: runit
          image: runit
          imagePullPolicy: Never
          ports:
            - containerPort: 80
          env:
            -name: { env-var-name}
             valueFrom:
               configMapKeyRef:
                 name: {name of the CM }
                 key: { key name to get value in  CM}

```

##Sample CM 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: runit-env-cm
data:
  # property-like keys; each key maps to a simple value
  kafkabrokers: "localhost:9092"
  threads: "5"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: runit-env-cm1
data:
  server.properties: |
    kafkabrokers=localhost:9092
    num.stream.threads=10
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: runit-env-file
data:
  # property-like keys; each key maps to a simple value
  kafkabrokers: "localhost:9093"
  maxrecords: "5"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: runit
  labels:
    app: runit
spec:
  replicas: 3
  selector:
    matchLabels:
      app: runit
  template:
    metadata:
      labels:
        app: runit
    spec:
      containers:
        - name: runit
          image: runit
          imagePullPolicy: Never
          ports:
            - containerPort: 80
          env:
            - name: KAFKA_BROKER_URL
              valueFrom:
                configMapKeyRef:
                  name: runit-env-cm
                  key: kafkabrokers
            - name: Stream_threads
              valueFrom:
                configMapKeyRef:
                  name: runit-env-cm
                  key: threads
          envFrom:
            - configMapRef:
                name: runit-env-cm1
          volumeMounts:
            - name: server-properties-from-cm
              mountPath: /etc/config
      volumes:
        - name: server-properties-from-cm
          configMap:
            name: runit-env-file
```
```shell
[root@localhost ConfigMap]# kubectl exec -it runit-c6dc8cf65-ctlw6 env
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=runit-c6dc8cf65-ctlw6
Stream_threads=5
kafkabrokers=localhost:9093
maxrecords=5
KAFKA_BROKER_URL=localhost:9092
KUBERNETES_SERVICE_PORT=443
/etc/config # ls -l
total 0
lrwxrwxrwx    1 root     root            19 Feb 28 06:49 kafkabrokers -> ..data/kafkabrokers
lrwxrwxrwx    1 root     root            17 Feb 28 06:49 maxrecords -> ..data/maxrecords

/etc/sv/secondrunitservice # cat /etc/config/maxrecords
5 
/etc/sv/secondrunitservice # cat /etc/config/kafkabrokers 
localhost:9093 
/etc/config # ls
server.properties
/etc/config # cat server.properties 
kafkabrokers=localhost:9092
num.stream.threads=10
```

## Passing Cm as CMD args 
-> using apline as base image  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: linux
  labels:
    app: linuz
spec:
  replicas: 1
  selector:
    matchLabels:
      app: linux
  template:
    metadata:
      labels:
        app: linux
    spec:
      containers:
        - name: alpine
          image: alpine
          imagePullPolicy: Never
          ports:
            - containerPort: 81
          command: [ "/bin/echo", "$(welcome_message) $(KAFKA_BROKER_URL)" ]
          env:
           - name: welcome_message
             value: "Welcome to cm"
           - name: KAFKA_BROKER_URL
             valueFrom:
               configMapKeyRef:
                 name: runit-env-cm
                 key: kafkabrokers
```
```shell
[root@localhost ConfigMap]# kubectl logs -f linux-69dff849c8-mdlf6
Welcome to cm localhost:9092
```