## Probes
A probe is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the kubelet either executes code within the container, or makes a network request


### Task 1: Liveness probes

Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations.
```	  
vi liveness.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: lns
  name: liveness-pod
spec:
  containers:
  - name: liveness
    image: busybox
    args:
    - /bin/sh
    - -c
    - touch /liveness; sleep 6000
    livenessProbe:
      exec:
        command:
        - cat
        - /liveness
      initialDelaySeconds: 5
      periodSeconds: 5
```
Add a live watch on the pods in another tab
```
kubectl get pod -w
```
Toggle to the first tab and execute the below commands
```
kubectl apply -f liveness.yaml
```
```	  
kubectl get pods	  
```
```
kubectl describe pod liveness-pod
``` 
Now login to the container and delete the file
```
kubectl exec -it liveness-pod -- sh 
```
```
rm -f liveness
```
```
exit
```
```
kubectl get pods
```
```
kubectl describe pod liveness-pod
```
You can see the pod restarted

 
### Task 2: Readiness probe

Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup, or depend on external services after startup. In such cases, you don't want to kill the application, but you don't want to send it requests either. Kubernetes provides readiness probes to detect and mitigate these situations

```
vi readiness.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: rns
  name: readiness-pod
spec:
  containers:
  - name: readiness
    image: nginx
    args:
    - /bin/bash
    - -c
    - service nginx start; touch /readiness; sleep 6000
    readinessProbe:
      exec:
        command:
        - cat
        - /readiness
      initialDelaySeconds: 5
      periodSeconds: 5
```
```	  
kubectl apply -f readiness.yaml
```
```
kubectl get pods
```
Expose the pod
```
kubectl expose pod readiness-pod --name readiness-svc --port 80 
```
Describe the svc to check the endpoints or execute the below command.
```
kubectl get ep readiness-svc
```
Login inside pod and delete the file 
```
kubectl exec -it readiness-pod bash 
```
```
rm -f /readiness
```
```
exit
```

View the Pod events and see that the readiness probe has failed
```
kubectl describe pods readiness-pod
```
Now describe service
```
kubectl describe svc readiness-svc
```
End point is gone

Replace the pod
```
kubectl replace -f readiness.yaml --force
```
Check for endpoints
```
kubectl describe svc readiness-svc
```
Endpoints are back

