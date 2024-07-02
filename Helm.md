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
To uninstall the WordPress application deployed using Helm with the release name my-wordpress
```
helm uninstall my-wordpress
```
To download the Bitnami WordPress Helm chart to your local machine.
```
helm fetch bitnami/wordpress
```
To remove a Helm repository from your local Helm client configuration
```
helm repo remove stable-charts
```
```
helm repo remove bitnami
```
To remove Helm package manager
```
sudo apt-get remove helm
```

### Task 2:  Installing WordPress with Helm

#### Helm setup

Download the shell script for the installation of the Helm package manager and run it.
```
wget  https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/helm.sh
```
```
bash helm.sh
```
```
helm version
```

#### Setup WordPress
#Add bitnami repository to helm which contains WordPress chart.
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
```
helm repo update
```
```
helm repo list
```

Search for WordPress package from the repository and install the WordPress chart with release name wordpress-chart. You can name it as you want.
```
helm search repo Wordpress
```
Helm Charts can be directly installed from the repo, or the yaml file downloaded, extracted and then chart installed
```
helm install wordpress-chart bitnami/wordpress
```
OR Perform helm fetch to get the WordPress compressed file to the working directory. Perform ls and verify that a compressed WordPress file is present in PWD.
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
helm install wordpress --generate-name   ## OR helm install wordpress wordpress
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

#### Cleanup
List the current helm release and delete it
```
helm ls
```
```
helm delete <helm_Release_Name >
```
```
helm ls
```
 
 
