## Node Affinity and AntiAffinity

### Task 1: Node Anti Affinity
Create a nginx yaml of preferredDuringScheduling nodeAffinity using content given below
```
vi aff-na-pod1.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: na-nginx-pod1
spec:
  containers:
  - name: na-nginx-ctr1
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disktype
            operator: NotIn  #In-NodeAffinity  #NotIn-NodeAntiAffinity
            values:
            - ssd
```
Save the file using "ESCAPE + :wq!"

Create a pod using the yaml created in the previous step
```
kubectl apply -f aff-na-pod1.yaml
```
View pods and notice that it has been created despite none of the nodes having the specified label
```
kubectl get pods -o wide
``` 
Delete the pod na-nginx-pod1
```
kubectl delete -f aff-na-pod1.yaml
```

### Task 2: Node Affinity 
Create another pod with nodeAffinity type of requiredDuringScheduling using content given below
```
vi aff-na-pod2.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: na-nginx-pod2
spec:
  containers:
  - name: na-nginx-ctr2
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - SSD
```
Save the file using "ESCAPE + :wq!"

Create a pod using the yaml created in the previous step
```
kubectl create -f aff-na-pod2.yaml
```
View pods and see that it is not scheduled as any of the nodes have the specified label
```
kubectl get pods -o wide
```
View all the nodes in the cluster and make a note of the name of one of the nodes
```
kubectl get nodes
```
Using the node name from the previous step, add a label to one of the nodes
```
kubectl label nodes <node_name> disktype=ssd
```
Apply the yaml again as it is stuck in a pending state
```
kubectl apply -f aff-na-pod2.yaml
```
View the pods again and see that it is now scheduled
```
kubectl get pods -o wide
```
Delete the pod
```
kubectl delete -f aff-na-pod2.yaml
```
