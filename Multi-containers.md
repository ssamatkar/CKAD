## Multi-container

### Task 1: Sidecar container
```
vi sidecar.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    # Main application container

  - name: sidecar-container
    image: debian:latest
    command: ['sh', '-c', 'apt update && apt install curl -y && while true; do echo "Sidecar Running"; sleep 10; done']
    # Sidecar container
```
```	
kubectl apply -f sidecar.yaml
```
```
kubectl get pod
```
```
kubectl exec -it sidecar-pod -c sidecar-container -- sh
```
``` 
curl 'http://localhost:80/'
```


### Task 2: Init container
```
vi init.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /app
    # Main application container

  initContainers:
  - name: init-container
    image: busybox:latest
    command: ['sh', '-c', 'echo "Init Container Completed" > /work-dir/completed.txt']
    volumeMounts:
    - name: workdir
      mountPath: /work-dir
    # Init container
  volumes:
  - name: workdir
    emptyDir: {}

```
```	
kubectl apply -f init.yaml
```
```
kubectl get pod
```
```
kubectl exec -it init-pod -c main-container
```
```
cd /app && ls
```
```
cat completed.txt
```

