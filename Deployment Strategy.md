## Deployment Strategy


### Task 1: Rolling Update in Kubernetes 

```
vi dep.yaml 
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dep1
  name: dep1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dep1
  strategy: {}
  template:
    metadata:
      labels:
        app: dep1
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
```

Note the strategy.Since nothing is given the default strategy comes into place which is the rolling update.

Apply the yaml file
```
kubectl apply -f dep.yaml
```
```
kubectl get deployments.apps,pod,rs
```
Note the Statgey and events by describing the deployment
```
kubectl describe deployments.apps dep1
```
To increase/decrease the number of replicas manually, either edit the yaml file, or edit the running deployment or execute the below command
```
kubectl scale deployment dep1 --replicas 5
```
For Autoscaling
```
kubectl get hpa
```
```
kubectl autoscale deployment dep1 --min 4 --max 12 --cpu-percent 70 --name hpa-dep1
```
```
kubectl get hpa
```
To upgrade/migrate the version of image, either edit the yaml file, or edit the deployment or execute the below command
Add a watch on the pods in a new tab
```
kubectl get po -w
```
Set a new image for the deployment
```
kubectl set image deploy dep1 nginx=nginx:latest --record
```
Check how the pods are getting deleted and recreated. 

Cross check if the image has been updated by executing the below command
```
kubectl describe deployments.apps dep1
```
To check the rollout history. The below command shows history of 10 versions.
```
kubectl rollout history deploy dep1
```
To Rollback
```
kubectl rollout undo deploy dep1 --to-revision 1
```
To check the history of a particular revision
```
kubectl rollout history deploy dep1 --revision=<revision-number>
```

### Task 2: Recreate Strategy in Kubernetes 
```
vi recreate.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dep2
  name: dep2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dep2
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: dep2
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
```
```
kubectl apply -f recreate.yaml
```
Add a watch on the pods in a new tab
```
kubectl get po -w
```
Set a new image for the deployment
```
kubectl set image deploy dep2 nginx=nginx:latest --record
```
Check how the pods are getting deleted and recreated. 

Cross check if the image has been updated by executing the below command
```
kubectl describe deployments.apps dep2
```

### Task 3: Blue/Green Deployment in Kubernetes 
```
vi web-blue.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-blue
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: web-blue
    spec:
      containers:
      - image: mandarct/web-blue:v1
        name: web-blue
        ports:
        - containerPort: 80
          protocol: TCP
```
```		 
kubectl apply -f web-blue.yaml
```
```
kubectl get deploy
```
```
kubectl get pods
```

Now create NodePort service to access application
```		 
vi svc-web.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-web
spec:
  ports:
  - port: 80        # Service port(internal)
    protocol: TCP
    targetPort: 80  # Container port
    nodePort: 32123 # Range from 30000 to 32767 . This is the external port number
  selector:
    app: web-blue
  type: NodePort

```
```	 
kubectl apply -f svc-web.yaml
```
```
kubectl get svc
```
```
kubectl get po -o wide
```
Check the endpoints
```
kubectl get ep svc svc-web
```
Access you application

Create another deployment (Green)
```
vi web-green.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-green
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: web-green
    spec:
      containers:
      - image: mandarct/web-green:v1
        name: web-green
        ports:
        - containerPort: 80
          protocol: TCP
```
```
kubectl apply -f web-green.yaml
```
```
kubectl get deployment
```
```
kubectl get pods
```
Access this application using same service that we created previously, by changing the selector in the Service yaml file.

Replace Selector `web-blue` by `web-green`
```	 
kubectl apply -f svc-web.yaml
```
```
kubectl get svc
```
```
kubectl get po -o wide
```
Check the endpoints
```
kubectl get ep svc svc-web
```
Access you application on the port 32123

### Task 4: Canary Deployment in Kubernetes 

Service and deployment should have a common label.
Add `type: web-app` to yaml file of both the deployments and apply again.

Use `replace` , as apply command might throw an error.
```
kubectl replace -f web-green.yaml
```
```
kubectl replace -f web-blue.yaml
```
In the yaml file of Service change the Selector to 'type: web-app` and replace.

Check the endpoints of the service. It should show all the pods of both the deployments.
```
kubectl get ep svc svc-web
```
```
kubectl get po -o wide
```
Access the application on port 32123. Sometime it will show Blue, other time it will show green.




