## ConfigMap & Secrets

### Task 1: Directly inject variables - Traditional Method
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: ws
  name: env-pod
spec:
  containers:
  - image: nginx
    name: ng-ctr
    ports:
    - containerPort: 80
    env:
    - name: db_user   #key
      value: admin
    - name: db_pwd
      value: "1234"
```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod env-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it env-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
env | grep db_
```

### Task 2: Inject `ALL` variables from ConfigMaps(FromLiteral) into POD.
Create a ConfigMap
```
kubectl create cm cm-1 --from-literal=db_user=admin --from-literal=db_pwd=1234
```
```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject the ConfigMap into the Pod Yaml File
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: cm-1
```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod web-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
env | grep db_
```

### Task 3: Inject `PARTICULAR` variables from ConfigMaps(FromLiteral) into POD.
Create a ConfigMap
```
kubectl create cm cm-1 --from-literal=db_user=admin --from-literal=db_pwd=1234
```
```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject particular variable from the ConfigMap into the Pod Yaml File
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    env:
    - name: db_password
      valueFrom:
        configMapKeyRef:
          name: cm-1
          key: db_pwd
```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod web-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
echo $db_password
```
```
env | grep db_
```
### Task 4: Inject variables from ConfigMaps(FromFile) into POD.
Create a file
```
vi token
```
```
This is CKAD Training. We are practicing Injecting variables from ConfigMaps(FromFile) into POD.
```
Create a ConfigMap
```
kubectl create cm cm-1 --from-file=token         #--from-file=<filen-name>. This file name acts as the key
```
```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject particular variable from the ConfigMap into the Pod Yaml File
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: cm-1
```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod web-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
echo $token
```
```
env | grep token
```

### Task 5 : Injecting ConfigMap as volume mount

ConfigMaps consumed as environment variables are not updated automatically and require a pod restart. When a ConfigMap currently consumed in a volume is updated, projected keys are eventually updated as well. The kubelet checks whether the mounted ConfigMap is fresh on every periodic sync. However, the kubelet uses its local cache for getting the current value of the ConfigMap. The type of the cache is configurable using the configMapAndSecretChangeDetectionStrategy field in the KubeletConfiguration struct. A ConfigMap can be either propagated by watch (default), ttl-based, or by redirecting all requests directly to the API server. As a result, the total delay from the moment when the ConfigMap is updated to the moment when new keys are projected to the Pod can be as long as the kubelet sync period + cache propagation delay, where the cache propagation delay depends on the chosen cache type (it equals to watch propagation delay, ttl of cache, or zero correspondingly).

Create a file
```
vi token
```
```
This is CKAD Training. We are practicing Injecting variables from ConfigMaps(FromFile) into POD.
```
Create a ConfigMap
```
kubectl create cm cm-1 --from-file=token        
```
```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject as volume mount
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  volumes:
  - name: cm-volume
    configMap:
      name: cm-1
  containers:
  - image: httpd
    name: ctr-1
    volumeMounts:
    - name: cm-volume
      mountPath: /app
    ports:
    - containerPort: 80

```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod web-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
cd /app
```
```
cat token
```

### Task 6 : Secret
Imperative
```
kubectl create secret generic secret-1 --from-literal=db_user=admin --from-literal=db_pwd=123
```
```
kubectl get secret
```
```
kubectl describe secret secret-1
```
Declrative
```
vi secret.yaml
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
data:
  ##all below values are base64 encoded

  ## To encode and print
  ## echo -n '<value-to-be-encoded>' | base64

  ## To decode and print
  ## echo '<value-to-be-decoded>' | base64 -d

  ##rootpw is root
  rootpw: cm9vdAo=

  ##user is user
  user: dXNlcgo=

  ##password is mypwd
  password: bXlwd2QK
```
```
kubectl apply -f secret.yaml
```
```
kubectl get secrets
```
```
kubectl describe secrets mysql-credentials
```
You can inject the secret in all the three ways as above.

Injecting all values
```
vi sc-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  containers:
  - image: nginx
    name: sc-ctr
    envFrom:
    - secretRef:
        name: secret-1
    - secretRef:
        name: mysql-credentials
```
```
kubectl apply -f sc-pod.yaml
```
```
kubectl get po
```
```
kubectl exec -it sc-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
echo $rootpw
```
```
echo $user
```
```
echo $password
```
```
env 
```
