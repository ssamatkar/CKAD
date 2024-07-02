## Bootstrap a Kubernetes Cluster using Kubeadm

To begin, log in to AWS Console.

### Task 1: Launching Instances on AWS

* `3` instances of `t2.medium` instance with OS version as `Ubuntu 22.04 LTS` in your preferred region.
* `Storage : 10 GB`
* Instead of opening all ports you can open these ports internally.
* `Type : Custom TCP`
* `Source type : Anywhere`
    |      Nodes	      |    Port Number	 |         Use Case                       |
    |---------------------|------------------|----------------------------------------|
    | Master, Workers	  |    `2379-2380`   |  Etcd Client API / Server API          |
    | Master              |       `6443`  	 |  Kubernetes API Server (Secure Port)   |
    | Master, Workers     |   `6782-6784`    |  Weave Net Server/Client API #CNI      |
    | Master, Workers     |   `10250-10255`	 |  Kubelet Communication                 |
    | Workers             |   `30000-32767`	 |  Reserved of NodePort Ips              |	   


* Add the below code in Advanced Details -> User data - optional.
```
#!/bin/bash
#
# Common setup for all servers (Control Plane and Nodes)

set -euxo pipefail

# Kuernetes Variable Declaration

KUBERNETES_VERSION="1.29.0-1.1"

# disable swap
sudo swapoff -a

# keeps the swaf off during reboot
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
sudo apt-get update -y


# Install CRI-O Runtime

OS="xUbuntu_22.04"

VERSION="1.28"

# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -

sudo apt-get update
sudo apt-get install cri-o cri-o-runc -y

sudo systemctl daemon-reload
sudo systemctl enable crio --now

echo "CRI runtime installed susccessfully"

# Install kubelet, kubectl and Kubeadm

sudo apt-get update -y
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-1-28-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-1-28-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes-1.28.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-1-29-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-1-29-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes-1.29.list

sudo apt-get update -y
sudo apt-get install -y kubelet="$KUBERNETES_VERSION" kubectl="$KUBERNETES_VERSION" kubeadm="$KUBERNETES_VERSION"
sudo apt-get update -y
sudo apt-mark hold kubelet kubeadm kubectl

sudo apt-get install -y jq

local_ip="$(ip --json addr show eth0 | jq -r '.[0].addr_info[] | select(.family == "inet") | .local')"
cat > /etc/default/kubelet << EOF
KUBELET_EXTRA_ARGS=--node-ip=$local_ip
EOF
```

* Launch the Instance.

### Task-2: Setting up Machines
* Task 2 can be skipped if the user data has been updated as shown in Task 1
* All steps in this task are to be performed on all the machines
* Connect all VMs with putty.
Switch to root.
```
sudo su
```

Create kubeadm.sh script file
```
vi kubeadm-setup.sh
```
```
#!/bin/bash
#
# Common setup for all servers (Control Plane and Nodes)

set -euxo pipefail

# Kuernetes Variable Declaration

KUBERNETES_VERSION="1.29.0-1.1"

# disable swap
sudo swapoff -a

# keeps the swaf off during reboot
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
sudo apt-get update -y


# Install CRI-O Runtime

OS="xUbuntu_22.04"

VERSION="1.28"

# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -

sudo apt-get update
sudo apt-get install cri-o cri-o-runc -y

sudo systemctl daemon-reload
sudo systemctl enable crio --now

echo "CRI runtime installed susccessfully"

# Install kubelet, kubectl and Kubeadm

sudo apt-get update -y
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-1-28-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-1-28-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes-1.28.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-1-29-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-1-29-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes-1.29.list

sudo apt-get update -y
sudo apt-get install -y kubelet="$KUBERNETES_VERSION" kubectl="$KUBERNETES_VERSION" kubeadm="$KUBERNETES_VERSION"
sudo apt-get update -y
sudo apt-mark hold kubelet kubeadm kubectl

sudo apt-get install -y jq

local_ip="$(ip --json addr show eth0 | jq -r '.[0].addr_info[] | select(.family == "inet") | .local')"
cat > /etc/default/kubelet << EOF
KUBELET_EXTRA_ARGS=--node-ip=$local_ip
EOF

```
```
bash kubeadm-setup.sh
```

### Task 3: Initializing the Cluster

rename the VM's as Master, Node1 and Node2 from the AWS Console.

Set the hostname to all three nodes as master, Node1, and Node2 in their respective terminals for easy understanding, by running the below command:

Connect to Master.

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Master
```
```
bash
```

Connect to Node1.

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Node1
```
```
bash
```

Connect to Node2.

Switch to root.
```
sudo su
```
```
hostnamectl set-hostname Node2
```
```
bash
```

Start kubeadm only on **master**
```
kubeadm init --ignore-preflight-errors=all
```
If the it runs successfully, it will provide a join command which can be used to join the master. Make a note of the highlighted part.
Run the following commands to configure kubectl on master.
```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
chown $(id -u):$(id -g) $HOME/.kube/config
```
 
### Task 4: Joining a Cluster


Run the kubeadm join command in **worker nodes**, that was previously noted from the master node in the previous task.
```
kubeadm join --token <your_token> --discovery-token-ca-cert- hash <your_discovery_token> 
```
**Note
If you want to list and generate tokens again to join worker nodes, then follow the below steps(optional)
```
kubeadm token list
kubeadm token create  --print-join-command
```

View node information on the **master**
```
kubectl get nodes
```

### Task 5: Deploy Container Networking Interface
Go to **master** node and 
Apply weave CNI (Container Network Interface) as shown below:
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
(reference link1:- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)

View nodes to see that they are ready.
```
kubectl get nodes
```

View all Pods including system Pods and see that dns and weave are running.
```
kubectl get pod -n kube-system
```

### Task 6: Create Pods
To check the API version of any resource
```
kubectl api-resources
```

Create a Pod called http based on a Docker image on the master.
```
kubectl run httpd --image=httpd
```

View the status of Pod and make sure it’s in the running state.
```
kubectl get pods
```

Get a shell to the container in the Pod using the Pod name from the previous step.
```
kubectl exec -it <pod_name> -- /bin/bash
``` 

Install curl in the container
```
apt update
apt install curl -y
```

Run curl on the localhost (container) to verify the http installation.
```
curl localhost 
exit
```

If you get the below `error` when running kubectl commands, execute the command given below.

`The connection to the server localhost:8080 was refused - did you specify the right host or port?`
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

#### *** DO NOT EXECUTE THE BELOW COMMAND UNTIL DOUBLY SURE ***
To remove the joins in your cluster
```
kubeadm reset
```


 

