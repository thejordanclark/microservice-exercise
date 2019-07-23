# Setting up Multi-node Kubernetes on virtual machine

## * Setting up the machine

#### SSH to the Ubuntu machine (Mac or Linux)
```bash
ssh username@FQDN
```

#### SSH to the Ubuntu machine (Windows)
Utilize Putty to connect to the host

#### Install docker
Docker is pre-installed on the AWS instance.

[Docker Install Instructions](https://docs.docker.com/install/)

#### Install docker-compose
Docker-compose is pre-installed on the AWS instance.  It is needed for the initial lab but not for kubernetes

[Docker Compose Install Instructions](https://docs.docker.com/compose/install/)

#### Check docker and docker-compose and image pull
```bash
docker version
docker-compose version
systemctl status docker
docker pull alpine
```

## * Install kubeadm (This step is for all nodes in the cluster)
[Kubeadm Official Installation](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```
<br>


```bash
sudo apt install -y kubeadm
```

<br>

## Disable Swap

Swap should be disabled on Kubernetes Nodes

```bash
sudo swapoff -a
```
<br>

## * Initialize master (This step is only for the master node in the cluster)

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
#### Note a similar message message.

> Your Kubernetes control-plane has initialized successfully!
>
> To start using your cluster, you need to run the following as a regular user:
>
>   mkdir -p $HOME/.kube
>   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
>   sudo chown $(id -u):$(id -g) $HOME/.kube/config
>
> You should now deploy a pod network to the cluster.
> Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
>   https://kubernetes.io/docs/concepts/cluster-administration/addons/
>
> Then you can join any number of worker nodes by running the following on each as root:
>
> kubeadm join 10.0.1.200:6443 --token suzcgn.pobqnfn1k4bc27yh \
>     --discovery-token-ca-cert-hash sha256:a7119208ef33b7a7682c7b91391a0ba32e276de37d8da41c64c18db3c5babcc5
>

### Check the status of kubelet
```bash
systemctl status kubelet
```

### Copy the connection info to the local user to allow management of kubernetes.
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### ask kubernetes to print the nodes
```bash
kubectl get node -o wide
```

### Kubernetes needs a networking overlay to interconnect pods.  Let's install flannel
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```


### ask kubernetes to print the nodes
```bash
kubectl get node -o wide
```

<br>

## Add additional Node to cluster

#### SSH to the Ubuntu machine (Mac or Linux)
```bash
ssh username@FQDN
```

#### SSH to the Ubuntu machine (Windows)
Utilize Putty to connect to the host


#### Check docker and image pull
```bash
docker version
systemctl status docker
docker pull alpine
```

## * Install kubeadm (This step is for all nodes in the cluster)
[Kubeadm Official Installation](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```
<br>


```bash
sudo apt install -y kubeadm
```

<br>

## Disable Swap

Swap should be disabled on Kubernetes Nodes

```bash
sudo swapoff -a
```
<br>

## Join the node to the cluster

First we need the join command from the master node.  On the master node run this command to show the join command.
```bash
kubeadm token create --print-join-command
```

Then back on the new node run the outputted command prefixed with sudo. For example:
```bash
sudo kubeadm join 10.0.1.100:6443 --token dys5qt.9jibqpi4giypbz5c     --discovery-token-ca-cert-hash sha256:a7119208ef33b7a7682c7b91391a0ba32e276de37d8da41c64c18db3c5babcc5
```

## Verify the new nodes

remember all kubectl commands is our example should be run from the master node.

### On the master node ask kubernetes to print the nodes
```bash
kubectl get node -o wide
```

<br>

# Notes

#### If running as root Set KUBECONFIG environment variable
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```
<br>

#### for non-root user
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
<br>

#### tokens are used to join the kube cluster, it can be retrieved using "kubeadm token" commands
```bash
kubeadm token list
```
<br>

#### A join command can be printed with

This command is needed to join additional nodes to the cluster
```bash
kubeadm token create --print-join-command
```

## * Check all pods
```bash
kubectl get pods -o wide --all-namespaces
```

#### If you have a single node cluster, Use Master node for deploying containers - Usually done for 1 node cluster
```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```
<br>


#### Join Node to master (change the token created by your Kubernetes master)
```bash
kubeadm join --token 511446.6121e5a77c23e9eb 162.44.170.117:6443 --discovery-token-ca-cert-hash sha256:586789540ff1fa8dbb8cf8c9762424b96604e5eaf40cb203b79e5c587b23630b
```
<br>

#### Check in master node
```bash
kubectl get nodes -o wide
kubectl get pods --all-namespaces -o wide
kubectl get services --all-namespaces -o wide
kubectl get secrets --all-namespaces -o wide
kubectl get serviceaccounts --all-namespaces -o wide
```

<br>


## * Setup docker private registry

#### Run the registry container
Note:- IP of machine running registry = ip1
It is always preferable to use a shared directory for storing data
```bash
docker run -d -p 5000:5000 --restart always --name registry registry:2
```
<br>

The IP of the host can be found by running on the host you started the registry on:
```bash
ip addr show dev ens5
```

The same IP should be visible when viewing the kubernetes nodes with the wide option as one of the nodes:
```bash
kubectl get node -o wide
```

#### To all machines in the cluster, append '--insecure-registry <IP>:5000' to the docker service file.
To the file '/lib/systemd/system/docker.service'

the line that starts with `ExecStart=` should look like with the adjusted IP address:
<br>
`ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry 10.0.1.100:5000`

You can use vim or nano to edit that file.

<br>

#### Reload
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
<br>
