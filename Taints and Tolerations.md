## Taints and Tolerations

Node affinity is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.

Tolerations are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints. Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

Deploy some pods and check on which nodes they are getting scheduled 
```
kubectl run pod-1 --image nginx
```
```
kubectl get po -o wide
```
Label the node
```
kubectl label nodes node1 disktype=ssd
```
Deploy a pod with the same Node Selector
```
vi pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    env: test
spec:
  containers:
  - name: pod1-crt
    image: nginx
  nodeSelector:
    disktype: ssd

```
Apply the yaml file
```
kubectl apply -f pod.yaml
```
Check on which Node it gets scheduled.
```
kubectl get pods -o wide
```
Taint Node1 to NoSchedule
```
kubectl taint nodes node1 key1=value1:NoSchedule
```
Replace pod1
```
kubectl replace -f pod.yaml --force
```
The Pod goes in a Pending State, since it has NodeSelector mapping it to  Node1 but the Node1 is tainted to NoSchedule

Add toleration to the same pod and replace
```
vi pod.yaml
```
```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```
```
kubectl replace -f pod.yaml --force
```
Check on which node it gets scheduled
```
kubectl get po -o wide
```
Remove the NodeSelector and replace the Pod
```
kubectl replace -f pod.yaml --force
```
Check where the Pod has scheduled now....
```
kubectl get po -o wide
```
Taint the node on which this pod gets deployed with `NoExecute`.
```
kubectl taint nodes <node-name> key1=value1:NoExecute
```
```
kubectl get po -o wide
```
The Pod gets evicted.

Remove the taint NoExecute
```
kubectl taint nodes node1 key1=value1:NoExecute-
```
Apply the taint PreferNoSchedule to both node1 and node2
```
kubectl taint nodes node1 key1=value1:PreferNoSchedule
```
```
kubectl taint nodes node2 key1=value1:PreferNoSchedule
```
Trying deploying another pod without any toleration.

Notice it gets scheduled on Node2. reason being Node1 has NoSchedule Taint still enabled.
```
kubectl run pod-test --image nignx
```

CLEANUP
Check the taints on all the ndoes and untaint the ones that you have applied.
