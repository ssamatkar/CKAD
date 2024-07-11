## ConfigMap & Secrets

### Task 1: Directly inject variables - Traditional Method
```
vi envtask1.yaml
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
kubectl apply -f envtask1.yaml
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
vi envtask2.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod-task2
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
kubectl apply -f envtask2.yaml
```
```
kubectl describe pod web-pod-task2
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod-task2 -- sh
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

```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject particular variable from the ConfigMap into the Pod Yaml File
```
vi envtask3.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod-task3
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
kubectl apply -f envtask3.yaml
```
```
kubectl describe pod web-pod-task3
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod-task3 -- sh
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
kubectl create cm cm-2 --from-file=token         #--from-file=<filen-name>. This file name acts as the key
```
```
kubectl get cm
```
```
kubectl describe cm cm-2
```
Inject particular variable from the ConfigMap into the Pod Yaml File
```
vi envtask4.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod-task4
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: cm-2
```
```
kubectl apply -f envtask4.yaml
```
```
kubectl describe pod web-pod-task4
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod-task4 -- sh
```
```
echo $token
```
```
env | grep token
```

### Task 5 : Injecting ConfigMap as volume mount


Create a file
```
vi token-task5
```
```
This is CKAD Training. We are practicing Injecting ConfigMap as volume mount into POD.
```
Create a ConfigMap
```
kubectl create cm cm-3 --from-file=token-task5       
```
```
kubectl get cm
```
```
kubectl describe cm cm-3
```
Inject as volume mount
```
vi envtask5.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod-task5
spec:
  volumes:
  - name: cm-volume
    configMap:
      name: cm-3
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
kubectl apply -f envtask5.yaml
```
```
kubectl describe pod web-pod-task5
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod-task5 -- sh
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
Declerative
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
