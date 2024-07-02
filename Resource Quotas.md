## Resource Quotas

### Task 1: Creating a Namespace

In Kubernetes, a ResourceQuota is a way to limit the amount of resources (CPU, memory, and persistent storage) that a namespace can consume. It helps in preventing resource exhaustion and ensures that one namespace doesn't negatively impact the performance or stability of others.

Keep in mind that ResourceQuotas are a way to define constraints, and they do not actively enforce them on existing resources. They are checked when new resources are created or when existing resources are updated. If a namespace exceeds its quota, the creation or update of resources may be rejected.

The below command can identify all namespaced objects
```
kubectl api-resources
```
```
kubectl create namespace ns1
```
```
kubectl get ns
```
```
kubectl describe ns ns1
```


### Task 2: Creating Resource Quota and Constraining Object Creation

Create a pod and expose it, before applying the resource quota to check if the resource quota applies to existing objects or not.
```
kubectl -n ns1 run pod1 --image nginx --port 80
```
```
kubectl -n ns1 expose pod pod1 --name pod1-svc --port 80 --type NodePort
```
```
kubectl -n ns1 run pod2 --image nginx --port 80
```
```
kubectl -n ns1 expose pod pod2 --name pod2-svc --port 80 --type NodePort
```
Imperative 
```
kubectl -n ns1 create quota rs-quota1 --hard=pods=3,services=1
```
```
kubectl describe ns ns1
```
```
kubectl get quota -n ns1
```
Declarative
```
vi rq2.yaml
```
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota2
  namespace: ns1
spec:
  hard:
    pods: "2"
    services: "2"

```
```
kubectl apply -f rq2.yaml
```
```
kubectl describe ns ns1
```
```
kubectl get quota -n ns1
```
Try deploying a new pod in namespace ns1
```
kubectl -n ns1 run pod3 --image nginx --port 80
```

### Task 3: Creating Resource Quota and Constraining Hardware Resources

```
vi rq3.yaml
```
```yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: rs-quota3
  namespace: ns1
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```
```
kubectl apply -f rq3.yaml
```
```
kubectl describe ns ns1
```

### Task 4: Verify Resource Quota Functionality
```
kubectl -n ns1 run pod4 --image nginx --port 80
```
```
vi rq4.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod5
  namespace: ns1
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "1000m"
      requests:
        memory: "600Mi"
        cpu: "350m"
    ports:
      - containerPort: 80
```
```	  
kubectl apply -f rq4.yaml
```
```
kubectl describe ns ns1
```
```
kubectl get resourcequota -n ns1 -o yaml
```
Deploy another Pod, such that the total resource exceeds the resoucre quota limit
```
vi rq5.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod6
  namespace: ns1
spec:
  containers:
  - name: quota-ctr
    image: nginx
    resources:
      limits:
        memory: "1600Mi"
        cpu: "2000m"
      requests:
        memory: "1600Mi"
        cpu: "800m"
    ports:
      - containerPort: 80
```
```
kubectl apply -f rq5.yaml
```

