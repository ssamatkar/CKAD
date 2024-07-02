## Node Labelling

### Task 1: Node labeling and constraining pods
View the default labels for all the nodes. Make a note of the NAME of one of the nodes. This node will be used to schedule a pod
```
kubectl get nodes --show-labels
```
Add a label to the node
```
kubectl label nodes <node_name> disktype=ssd
```
View the labels again to verify labeling from the previous 
```
kubectl get nodes --show-labels
``` 
Create a pod and use nodeSelector to constrain it to a particular node
```
vi nlns-pod.yaml
```
```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: nlns-nginx-pod
  labels:
    env: test
spec:
  containers:
  - name: nlns-nginx-ctr
    image: nginx
  nodeSelector:
    disktype: ssd

```
Create the pod using the yaml created in the previous step
```
kubectl apply -f nlns-pod.yaml
```
Verify that the pod is scheduled on the desired node
```
kubectl get pods -o wide
``` 

### Task 2: Cleanup
Delete the objects created in this lab
```
kubectl delete -f nlns-pod.yaml 
```
```
kubectl label nodes <node_name> disktype-   
```
`-(minus)` for removing label
 
Verify that the label is deleted.
```
kubectl get nodes --show-labels | grep SSD
```
