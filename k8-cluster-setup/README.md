# A Step-by-Step Guide to Setting Up a Kubernetes Cluster
![enter image description here](https://k21academy.com/wp-content/uploads/2020/10/hhhhjhkjsfsd.jpeg)

# Kubernetes Cluster Setup Guide (Kubeadm, Containerd, Flannel)

This guide walks you through setting up a Kubernetes cluster (v1.32) using `kubeadm`, `containerd`, and the **Flannel** CNI plugin.

---

## Common Steps (Run on Both Master & Worker Nodes)

### 1. Disable Swap

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
### 2. Load Required Kernel Modules
```bash
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
### 3. Set Required sysctl Params
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```
### 4. Install and Configure Containerd
```bash
sudo apt update
sudo apt install -y containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```
### 5. Install Kubernetes Tools (kubelet, kubeadm, kubectl)
```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg

sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg

echo "deb [signed-by=/etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
## Master Node Only
### 6. Initialize the Cluster
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
>Copy and save the `kubeadm join` command printed at the end!

### 7. Set up `kubeconfig` for kubectl
```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 8. Install Flannel CNI
```bash
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```
## Worker Node Only
### After completing steps 1–5, join the cluster using the join command you copied earlier:
```bash
sudo kubeadm join 192.168.0.10:6443 --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
## Verify the Cluster (From Master Node)
```bash
kubectl get nodes
```
### Expected Output:
```bash
NAME     STATUS   ROLES           AGE     VERSION
master   Ready    control-plane   XXm     v1.32.x
worker   Ready    <none>          XXm     v1.32.x
```
>✨ You now have a working Kubernetes cluster with containerd and Flannel!
## Conclusion
By following this step-by-step guide, you've successfully set up a production-ready Kubernetes cluster using `kubeadm`, `containerd`, and Flannel as the CNI. You’ve ensured system compatibility by disabling swap, configuring kernel modules and sysctl settings, and enabling systemd cgroup support for containerd. With your master and worker nodes connected and networking properly configured, your cluster is now ready to run workloads, deploy applications, and scale as needed. 

> Keep exploring Kubernetes features like Deployments, Services, ConfigMaps, and StatefulSets to harness the full power of container orchestration!

