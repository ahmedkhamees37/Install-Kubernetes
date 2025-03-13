![image](https://github.com/user-attachments/assets/b6955384-8845-4d93-a854-e8af0fc3d223)

# Kubernetes Installation Guide 🚀

This guide provides step-by-step instructions to install Kubernetes (v1.31.7) using `kubeadm` and `k3s` as a bonus. The setup includes:
- One **Control Plane** node
- One **Worker** node
- **Flannel** as the networking CNI

## Prerequisites 🛠️
Ensure you have the following before proceeding:
- Two Virtual Machines with **Ubuntu 22.04**
- **Minimum Resources**: 2 vCPUs, 4GB RAM each
- **Static IP Addresses**:
  - `192.168.1.100` (Control Plane)
  - `192.168.1.101` (Worker)
- Internet connectivity

---

## Step 1: Install Kubernetes Dependencies 📦
Run the following on **both** Control Plane and Worker nodes:

```bash
sudo apt update -y
sudo apt install -y apt-transport-https ca-certificates curl
```

Add the Kubernetes repository:

```bash
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg \
  https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update -y
```

Install Kubernetes components:

```bash
sudo apt install -y kubelet=1.31.7-1.1 kubeadm=1.31.7-1.1 kubectl=1.31.7-1.1
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## Step 2: Initialize the Control Plane 🎛️
Run **only on the Control Plane node (192.168.1.100):**

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Set up `kubectl` access:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## Step 3: Install Flannel CNI 🌐

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

---

## Step 4: Join Worker Node 🏗️
Retrieve the `join` command from the output of `kubeadm init`, or run on the **Control Plane**:

```bash
kubeadm token create --print-join-command
```

On the **Worker Node (192.168.1.101)**, execute the command:

```bash
sudo kubeadm join 192.168.1.100:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Verify the node status from the Control Plane:

```bash
kubectl get nodes
```

---

## Step 5: Deploy a Sample Application 🚢
Deploy an **Nginx** web server:

```bash
kubectl create deployment nginx --image=nginx --replicas=3
kubectl expose deployment nginx --port=80 --type=NodePort
```

Verify deployment:

```bash
kubectl get pods
kubectl get svc
```

---

## Bonus: Install Kubernetes with K3s 🎯
### Step 1: Install K3s Server
On the **Control Plane Node**:

```bash
curl -sfL https://get.k3s.io | sh -
```

Retrieve the K3s token:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

### Step 2: Install K3s Agent
On the **Worker Node**, replace `<token>` with the actual token:

```bash
curl -sfL https://get.k3s.io | K3S_URL="https://192.168.1.100:6443" K3S_TOKEN="<token>" sh -
```

### Step 3: Verify K3s Cluster

```bash
kubectl get nodes
```

---

## 🎉 Congratulations!
You have successfully installed Kubernetes using `kubeadm` and `k3s`. 🚀 Happy Learning!

