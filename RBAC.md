## RBAC

RBAC stands for Role-Based Access Control. 

It is a method of regulating access to resources based on the roles of individual users within an organization. With RBAC, you can specify what actions (such as create, read, update, delete) a user or group of users can perform on specific Kubernetes resources.

### Task 1: Role and Role Binding

Roles: A role is a set of permissions that define what actions can be performed on Kubernetes resources within a namespace. For example, you can define a role that allows read-only access to pods or a role that allows full control over deployments.

RoleBindings: A role binding binds a role to a user, group, or service account within a namespace. It specifies who has access to the resources defined by the role. For example, you can create a role binding that associates a specific role with a particular user, granting them the permissions defined by that role.

#### Create a new ServiceAccount
```
kubectl get ns
```
```
kubectl create ns ns1

```
```
kubectl create sa -n ns1 sa1 
```
```
kubectl get ns
```

#### Create a new Role and RoleBinding 
```
kubectl create role role1 --verb=list --resource=pods -n ns1 --dry-run=client -o yaml > role.yaml
```
```
kubectl apply -f role.yaml
```
```
kubectl create rolebinding role-binding1 --role=role1 --serviceaccount=ns1:sa1 -n ns1
```
Describe the Rold and role binding
```
kubectl -n ns1 describe role,rolebindings.rbac.authorization.k8s.io 
```

#### Test whether you are able to do a GET request to Kubernetes API 
```
kubectl run pod1 --image=nginx -n ns1
```
```
kubectl exec -it -n ns1 pod1 -- /bin/bash 
```
Check the details of the Service Account assosciated with the pod
```
cd /var/run/secrets/kubernetes.io/serviceaccount/ && ls
```
```
ls
```
```
cat ca.crt
```
```
cat token
```
```
cat namespace
```

Make an API call to list all the pods in the Namespace ns1
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/ns1/pods | grep '"name": "pod'
```
The API call is forbidden
```
exit
```
```
vi pod-sa.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: ns1
spec:
  serviceAccountName: sa1
  containers:
  - name: my-container
    image: nginx
```
```
kubectl apply -f pod-sa.yaml
```
```
kubectl exec -it -n ns1 pod2 -- /bin/bash 
```
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/ns1/pods | grep '"name": "pod'
```
You should be able to see the details of all the pods

Access to Pods in deafult namespace is forbidden
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /ar/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/default/pods
```

#### Test whether you are able to do a POST request to Kubernetes API (create a new pod)

For creating new Pod we can use `POST`

```
curl -v \
  --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "my-pod",
      "namespace": "ns1"
    },
    "spec": {
      "containers": [
        {
          "name": "my-container",
          "image": "nginx"
        }
      ]
    }
  }' \
  https://kubernetes.default/api/v1/namespaces/ns1/pods

```
You would get Forbidden Error.
```
exit
```

Now edit the role to update the permissions.
```
vi role.yaml 
```
Add the below permissions
```
 - create
 - update
```
Replace the role
```
kubectl replace -f role.yaml --force
```
```
kubectl exec -it -n ns1 pod2 -- /bin/bash
```
Create a pod
```
curl -v \
  --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "my-pod",
      "namespace": "ns1"
    },
    "spec": {
      "containers": [
        {
          "name": "my-container",
          "image": "nginx"
        }
      ]
    }
  }' \
  https://kubernetes.default/api/v1/namespaces/ns1/pods
```
Retrieve the Pod details
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/ns1/pods | grep '"name":'
```
```
exit
```

### Task 2: Cluster Role and Cluster Role Binding

Cluster Role : A ClusterRole is a set of permissions that can be applied across the entire Kubernetes cluster.ClusterRoles are not namespace-scoped; they apply cluster-wide. This means that permissions granted by a ClusterRole affect resources in all namespaces.

Cluster Role Binding: A ClusterRoleBinding binds a ClusterRole to a set of subjects (users, groups, or service accounts) across the entire cluster. It effectively grants the permissions defined in the ClusterRole to the subjects specified in the binding. Similar to ClusterRoles, ClusterRoleBindings are also cluster-wide and apply across all namespaces.

#### Create a new ServiceAccount
```
kubectl create ns ns1
```
```
kubectl create sa -n ns1 sa2
```
```
kubectl get ns
```

#### Create a new Cluster Role and Cluster Role Binding 

 ```
vi cluster-role.yaml
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```
```
kubectl apply -f cluster-role.yaml
```
```
vi cluster-role-binding.yaml
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-user-binding
subjects:
- kind: ServiceAccount
  name: sa2
  namespace: ns1
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

```
```
kubectl apply -f cluster-role-binding.yaml
```
Describe the Cluster Role and Cluster role binding
```
kubectl describe clusterrole pod-reader 
```
```
kubectl describe clusterrolebindings.rbac.authorization.k8s.io my-user-binding
```

#### Test whether you are able to do a GET request to Kubernetes API 
```
kubectl run pod3 --image=nginx -n ns1
```
```
kubectl exec -it -n ns1 pod3 -- /bin/bash 
```
Make an API call to list all the pods in the Namespace ns1
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/ns1/pods | grep '"name": "pod'
```
The API call is forbidden

Make an API call to list all the pods in the Namespace default
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/default/pods | grep '"name": "pod'
```
```
exit
```
```
vi pod-sa2.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod4
  namespace: ns1
spec:
  serviceAccountName: sa2
  containers:
  - name: my-container
    image: nginx
```
```
kubectl apply -f pod-sa2.yaml
```
```
kubectl exec -it -n ns1 pod4 -- /bin/bash 
```
Make an API call to list all the pods in the Namespace ns1
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/ns1/pods | grep '"name": "pod'
```
The API call is successful

Make an API call to list all the pods in the Namespace default
```
curl -v --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default/api/v1/namespaces/default/pods | grep '"name": "pod'
```
```
exit
```
