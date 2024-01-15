# A Step-by-Step Guide to Setting Up a Kubernetes Cluster
![enter image description here](https://k21academy.com/wp-content/uploads/2020/10/hhhhjhkjsfsd.jpeg)

## Instructions
Kubernetes plays a pivotal role in modern application deployment, providing efficient container orchestration. This guide will walk you through the process of setting up a Kubernetes cluster, ensuring a robust foundation for your applications.


## Section 1: Preparing the Environment

#### Disable Swap:
To optimize system resources, disable swap:
```sh
sudo apt-get update
sudo swapoff -a
sudo vim /etc/fstab
sudo init 6
```
#### Configure Modules:
Ensure necessary modules are loaded for containerd:
```sh
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe br_netfilter
sudo modprobe overlay
```
#### Networking Parameters:

Configure system parameters related to networking:
```sh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```
## Section 2: Installing and Configuring Containerd

#### Install Containerd:

Update and install containerd:

```sh
sudo apt-get update
sudo apt-get install -y containerd 
```
#### Modify Containerd Configuration:

Enable systemd cgroup integration:
```sh
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd.service
sudo systemctl status containerd
```
## Section 3: Installing Kubernetes Management Tools

#### Install Kubectl, Kubeadm, and Kubelet:
```sh
# Follow official instructions: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management
```
Ensure that the system is up-to-date and has necessary dependencies:
```sh
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo apt-get install -y apt-transport-https ca-certificates curl
```
Add the Kubernetes repository GPG key:
```sh
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
```
Add the Kubernetes repository:
```sh
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Update the package list and install Kubernetes tools:
```sh
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```
Mark the installed packages to prevent unintended upgrades:
```sh
sudo apt-mark hold kubelet kubeadm kubectl
```
## Section 4: Initializing the Master Server (Only Master server)

#### Kubernetes Cluster Init:
Initialize the master server, creating the kubeconfig file:
```sh
sudo kubeadm init --apiserver-advertise-address=172.17.17.200 --pod-network-cidr=192.168.0.0/16 --cri-socket /run/containerd/containerd.sock --ignore-preflight-errors Swap

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## Section 5: Setting Up Networking with Calico (Only Master Server)
#### Install Calico Networking:

Calico is a popular choice for networking and network policy in Kubernetes clusters. Follow these steps to install Calico:
```sh
# Create Calico custom resources using the provided YAML files
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/tigera-operator.yaml

# Download Calico custom resources YAML
curl https://raw.githubusercontent.com/projectcalico/calico/v3.24.1/manifests/custom-resources.yaml -O

# Apply Calico custom resources
kubectl create -f custom-resources.yaml
```
These commands will deploy the necessary Calico components to your Kubernetes cluster. Ensure that the Calico Operator and custom resources are successfully created. You can verify the status using:
```sh
kubectl get tigerastatus 
```
This will display the status of the Calico components.
## Section 6: Joining Nodes to the Cluster (Only Worker Server)

#### Join Nodes:

Generate and execute the join command on worker nodes:
```sh
# Use the provided 'kubeadm token create --print-join-command' output
kubeadm join 192.168.0.123:6443 --token <token> --discovery-token-ca-cert-hash <hash>
```
If you need to retrieve this command later, you can use the following command to print it:
```sh
kubeadm token create --print-join-command
```
## Conclusion:

Setting up a Kubernetes cluster involves several crucial steps, from environment preparation to networking configuration. This guide provides a foundation for deploying applications seamlessly in a Kubernetes environment. Explore additional configurations and advanced topics to further enhance your cluster's capabilities.

