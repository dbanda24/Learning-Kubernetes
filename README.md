# Learning-Kubernetes
Setup Kubernetes Cluster on an Amazon Ubuntu EC2 Instance

## Step 1: Update Your System
Update the package list and upgrade installed packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

## Step 2: Install Dependencies
Install required tools and packages:
```bash
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```

### Step 3: Disable Swap
Kubernetes requires swap to be disabled:
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### Step 4: Install Docker or containerd
Kubernetes needs a container runtime. We'll use containerd (recommended).

#### 4.1: Install containerd
```bash
sudo apt install -y containerd
```
#### 4.2: Configure containerd
Generate the default configuration file:
```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
Restart containerd:
```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```
### Step 5: Install Kubernetes Components
Install kubeadm, kubelet, and kubectl.

⚠️ Note
The old repository (https://apt.kubernetes.io kubernetes-xenial) is deprecated. Use the new pkgs.k8s.io repository instead.

#### 5.1: Add the Kubernetes APT Repository (New Method)
##### Create a keyring directory
```bash
sudo mkdir -p /etc/apt/keyrings
```
##### Download and add the Kubernetes repository GPG key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

##### Add the Kubernetes repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list

##### Update package list
```bash
sudo apt update
```
💡 If a newer version (like v1.31) is available, replace v1.30 in the URL above.

#### 5.2: Install kubeadm, kubelet, and kubectl
```bash
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

If you see errors about Snap versions, remove them first:
```bash
sudo snap remove kubelet kubectl kubeadm
```

#### Step 6: Initialize the Kubernetes Cluster
Run the following command on the control-plane (master) node:
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

#### 6.1: Configure kubectl for the Current User
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 6.2: Save the Join Command
The output of kubeadm init will include a kubeadm join command for worker nodes. Copy and save this for later.

### Step 7: Install a Pod Network Add-On
Choose a CNI (Container Network Interface). We'll use Calico.

#### 7.1: Apply the Calico Manifest
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Step 8: Add Worker Nodes
Run the kubeadm join command from Step 6.2 on each worker node:
```bash
sudo kubeadm join <master-ip>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```
### Step 9: Verify the Cluster
#### 9.1: Check Nodes
On the master node, check the status of nodes:
```bash
kubectl get nodes
```
#### 9.2: Check Pods
Verify that the cluster is working by checking pods in the kube-system namespace:
```bash
kubectl get pods -n kube-system
```

Optional: Enable Autocompletion for kubectl
```bash
sudo apt install -y bash-completion
echo 'source <(kubectl completion bash)' >>~/.bashrc
source ~/.bashrc
```

Troubleshooting Tips
1. Reset the Cluster
If the installation fails, reset kubeadm and try again:
```bash
sudo kubeadm reset -f
sudo rm -rf $HOME/.kube
```
2. Check Logs
Use the following commands to troubleshoot issues:
```
journalctl -xeu kubelet
kubectl describe pod <pod-name> -n kube-system
```
✅ This guide is now compatible with modern Ubuntu (20.

## **Issue**

When Running the following command on the control-plane (master) node:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

i get error:

```bash
I1107 19:50:20.055497   20408 version.go:256] remote version is much newer: v1.34.1; falling back to: stable-1.30
[init] Using Kubernetes version: v1.30.14
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

This error means IP forwarding is disabled on your host system, which is required for applications like Kubernetes to route traffic between different network interfaces.

## **How to Fix this**

To fix it, we must enable IP forwarding by changing the /proc/sys/net/ipv4/ip_forward file to 1. This can be done temporarily with `sudo sysctl -w net.ipv4.ip_forward=1` or permanently by adding `net.ipv4.ip_forward=1` to a file in `/etc/sysctl.d/` and then running `sudo sysctl --system`. 



