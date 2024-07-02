## Metrics Server

Install Metrics Server 

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Check the status of the metrics server
```
kubectl -n kube-system get po
```
Download the patch for the Mtrics Server
```
wget https://gist.githubusercontent.com/initcron/1a2bd25353e1faa22a0ad41ad1c01b62/raw/008e23f9fbf4d7e2cf79df1dd008de2f1db62a10/k8s-metrics-server.patch.yaml
```
Apply the patch
```
kubectl patch deploy metrics-server -p "$(cat k8s-metrics-server.patch.yaml)" -n kube-system
```
```
kubectl -n kube-system get po
```
To check the resource utilization by nodes
```
kubectl top node
```
To check the resource utilization by pods
```
kubectl top pod
```
Sort the output on CPU Utilization
```
kubectl top pod -A --sort-by cpu
```
Sort the output on Memory Utilization
```
kubectl top pod -A --sort-by memory
```
To check the total utilization by all pods
```
kubectl top pod --sum=true -A
```
To check all the options
```
kubectl top pod --help
```



