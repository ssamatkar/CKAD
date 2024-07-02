## Pod Affinity

Create a yaml file with the content given below. Note that the label used is app: nginx
```
vi aff-pa-pod1.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pa-nginx-pod1
  labels:
    app: pa-nginx-app
spec:
  containers:
  - name: pa-nginx-ctr1
    image: nginx
```
Save the file using "ESCAPE + :wq!"

Create a pod using the yaml created in the previous step and View the node on which it was scheduled
```
kubectl apply -f aff-pa-pod1.yaml
```
```
kubectl get pods -o wide
```
Create a new pod yaml using the content given below
```
vi aff-pa-pod2.yaml
```
```yaml	
apiVersion: v1
kind: Pod
metadata:
  name: pa-nginx-pod2
spec:
  containers: 
  - name: pa-nginx-ctr2
    image: nginx
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - pa-nginx-app
        topologyKey: kubernetes.io/hostname
``` 
Save the file using "ESCAPE + :wq!" and Apply the yaml created in the previous step
```
kubectl apply -f aff-pa-pod2.yaml
```
View on which node the pod is scheduled. Notice that the pod is scheduled on the same node as the pod created previously
```
kubectl get pods -o wide
```

### Task 3: Clean-up
Delete the pods created in the previous task.
```
kubectl delete -f aff-pa-pod1.yaml
```
```
kubectl delete -f aff-pa-pod2.yaml
```
