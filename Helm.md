## Helm

### Task 1:  Commonly used Helm Commands

To check the version of Helm installed
```
helm version
```

To add a Helm chart repository to your local environment. Below we are adding two Helm Chart Repositories.
```
helm repo add bitnami https://charts.bitnami.com/bitnami 
```
```
helm repo add stable-charts https://charts.helm.sh/stable
```
To list the Helm chart repositories configured in your local environment.
```
helm repo list
```
To search for Helm charts related to WordPress in the configured Helm repositories
```
helm search repo wordpress
```
To install the WordPress application using the Bitnami Helm chart with the release name my-wordpress
```
helm install my-wordpress bitnami/wordpress
```
To list all the releases currently installed on your Kubernetes cluster
```
helm list
```
```
helm fetch bitnami/wordpress
```
```
ls
``` 
Extract the compressed file using the below command.
```
tar -xvzf wordpress-19.2.6.tgz
```
Now, deploy the WordPress using helm install command.  This will set up the pods and services for WordPress.
```
helm install wordpress wordpress
``` 
#### Verify that WordPress has been set up 
Verify that the pods are running.
```
kubectl get pods
```
Now Notice that MariaDB (database) pods are part of statefulset.
```
kubectl get sts
```
```
kubectl describe sts
```
Also Notice that the front end of wordpress are part of deployment
```
kubectl get deploy
```
View the services using the below commands. Make note of the EXTERNAL IP of the LoadBalancer service.
```
kubectl get svc
```
```
kubectl get svc --namespace default -w wordpress-1637763307
``` 
Open the browser and paste the service endpoint noted on the previous step. Observe that the WordPress site is up and running.

 
 
