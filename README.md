# Kubernetes Kubectl Autocomplete (Bash)

## Enabling Autocomplete for `kubectl` in Bash

To enable command-line autocomplete for `kubectl` in Bash, follow these steps:

### **1. Enable Autocomplete for the Current Session**
Run the following command in your terminal:
```sh
source <(kubectl completion bash)
```
This will enable autocomplete for the current session only.

### **2. Make It Permanent**
To enable autocomplete permanently, add the following line to your `~/.bashrc` or `~/.bash_profile`:
```sh
echo 'source <(kubectl completion bash)' >> ~/.bashrc
```
Then, reload your shell configuration:
```sh
source ~/.bashrc
```

### **3. Install Bash Completion (If Needed)**
If you don't have `bash-completion` installed, install it first:
- **Ubuntu/Debian**: `sudo apt install bash-completion`
- **Mac (Homebrew)**: `brew install bash-completion`
- **CentOS/RHEL**: `sudo yum install bash-completion`

### **4. Verify the Autocomplete**
Test by typing:
```sh
kubectl get [TAB]
```
You should see a list of available resources (e.g., pods, deployments, services, etc.).

Now, `kubectl` command autocomplete is enabled in your Bash shell! 🚀


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



![image](https://github.com/user-attachments/assets/053ea2fd-0b44-4aac-93a6-464a318b904c)

# 🚀 K3s Cluster Setup & Kubernetes Configurations

## 📌 Overview
This guide walks through setting up a **K3s cluster** with 1 control plane (server) and 1 worker node (agent). It also covers configuring `kubectl`, creating a custom plugin, and deploying an application using YAML.

---

## 🔧 1. Setup K3s Cluster (1 Control Plane + 1 Worker Node)
### 🖥️ Install K3s on the Control Plane (Server)
```sh
curl -sfL https://get.k3s.io | sh -
```
This installs and starts the K3s control plane automatically.

### 🔑 Retrieve the Node Token
```sh
sudo cat /var/lib/rancher/k3s/server/node-token
```
Copy the token for later use.

### ⚙️ Install K3s on the Worker Node (Agent)
Replace `<server-ip>` with the actual IP of your control plane node:
```sh
curl -sfL https://get.k3s.io | K3S_URL="https://<server-ip>:6443" K3S_TOKEN="<copied-token>" sh -
```

### ✅ Verify Nodes are Ready
```sh
kubectl get nodes
```

---

## 📂 2. Create Namespace `iti-45`
```sh
kubectl create namespace iti-45
```

---

## 🎯 3. Configure `kubectl` Context

### 🌍 Add a New Context (`iti-context`)
```sh
kubectl config set-context iti-context --cluster=default --user=default --namespace=iti-45
```

### 🔄 Switch to the New Context
```sh
kubectl config use-context iti-context
```

### 🔍 Verify the Current Context
```sh
kubectl config current-context
```

---

## 🛠️ 4. Create a Custom `kubectl` Plugin (`kubectl hostnames`)
### 📄 Create the Plugin Script
```sh
mkdir -p ~/.kube/plugins
nano ~/.kube/plugins/kubectl-hostnames
```

### ✍️ Add the Following Content
```sh
#!/bin/bash
kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
```

### 🔓 Make the Script Executable
```sh
chmod +x ~/.kube/plugins/kubectl-hostnames
```

### 🚀 Move it to a Global Path
```sh
sudo mv ~/.kube/plugins/kubectl-hostnames /usr/local/bin/kubectl-hostnames
```

### 🔄 Test the Plugin
```sh
kubectl hostnames
```

---

## 📜 5. Create Deployment (`deployment.yaml`)
Create a file named `deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: iti-45
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        env:
        - name: FOO
          value: "ITI"
```

### 🚀 Deploy the Application
```sh
kubectl apply -f deployment.yaml
```

### 🔎 Verify Deployment
```sh
kubectl get deployments -n iti-45
kubectl get pods -n iti-45
```

---

## 🎯 Conclusion
Congratulations! 🎉 You have successfully set up a **K3s cluster**, configured `kubectl`, created a custom plugin, and deployed an application. Now, you can scale and manage your cluster with ease. 🚀🔥

For any issues, check the logs:
```sh
kubectl logs <pod-name> -n iti-45
```

Happy Kubernetes-ing! 🐳💙


![Screenshot 2025-03-15 123620](https://github.com/user-attachments/assets/a80598aa-4e45-417a-a693-b71ef8523a62)

![Screenshot 2025-03-15 123709](https://github.com/user-attachments/assets/342a48a1-8a4f-4330-8604-7e85d044684e)

# Kubernetes Lab 4

## Task 1: Persistent Volumes

### a. Create a PersistentVolume named `cool-volume`
Save the following YAML as `pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cool-volume
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  hostPath:
    path: "/tmp/my-cool-vol"
```

Apply it:
```sh
kubectl apply -f pv.yaml
```

---

### b. Create a PersistentVolumeClaim named `my-claim`
Save the following YAML as `pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: ""
```

Apply it:
```sh
kubectl apply -f pvc.yaml
```

---

### c. Create a Pod named `pvc-user`
Save the following YAML as `pvc-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-user
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: "/mnt/share/my-pvc"
          name: my-volume
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: my-claim
```

Apply it:
```sh
kubectl apply -f pvc-pod.yaml
```

---

## Task 2: ConfigMaps

### a. Create a file `/opt/cm.yaml`
Save the following YAML as `/opt/cm.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: birke
data:
  tree: birke
  level: "3"
  department: park
```

Apply it:
```sh
kubectl apply -f /opt/cm.yaml
```

---

### b. Create a ConfigMap named `trauerweide`
Save the following YAML as `trauerweide-cm.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: trauerweide
data:
  tree: trauerweide
```

Apply it:
```sh
kubectl apply -f trauerweide-cm.yaml
```

---

### c. Create a Pod `pod1` with environment variable and volume mount
Save the following YAML as `pod1.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
    - name: nginx
      image: nginx:alpine
      env:
        - name: TREE1
          valueFrom:
            configMapKeyRef:
              name: trauerweide
              key: tree
      volumeMounts:
        - name: config-volume
          mountPath: "/etc/birke"
  volumes:
    - name: config-volume
      configMap:
        name: birke
```

Apply it:
```sh
kubectl apply -f pod1.yaml
```

---

## Summary
This README provides YAML configurations and instructions to:
1. Set up a PersistentVolume, PersistentVolumeClaim, and a Pod that mounts the volume.
2. Create ConfigMaps and a Pod that uses them as an environment variable and volume.

Run the given `kubectl apply -f` commands to apply the configurations.

Happy Coding! 🚀






![Screenshot 2025-03-15 123533](https://github.com/user-attachments/assets/7b32dd9f-0c65-4ab5-ba33-69a861bd73e9)

# 🚀 Kubernetes Lab 3 - Ingress and Services

## 🎯 **Objective**
This lab focuses on deploying applications in a Kubernetes cluster, exposing them as services, and configuring an Ingress resource to route traffic.

---

## 📌 **Prerequisites**
- ✅ A running Kubernetes cluster
- ✅ `kubectl` installed and configured
- ✅ Ingress controller (e.g., NGINX) installed in the cluster

---

## 🏗 **Step 1: Create the `world` Namespace**
```sh
kubectl create namespace world
```

---

## 📦 **Step 2: Create Deployments**
Create a file `deployments.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: africa
  namespace: world
spec:
  replicas: 2
  selector:
    matchLabels:
      app: africa
  template:
    metadata:
      labels:
        app: africa
    spec:
      containers:
      - name: africa
        image: husseingalal/africa:latest
        ports:
        - containerPort: 80

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: europe
  namespace: world
spec:
  replicas: 2
  selector:
    matchLabels:
      app: europe
  template:
    metadata:
      labels:
        app: europe
    spec:
      containers:
      - name: europe
        image: husseingalal/europe:latest
        ports:
        - containerPort: 80
```

Apply the deployments:
```sh
kubectl apply -f deployments.yaml
```

---

## 🌐 **Step 3: Expose Deployments as Services**
Run the following commands to expose the deployments as ClusterIP services:

```sh
kubectl expose deployment africa --port=8888 --target-port=80 --type=ClusterIP --namespace=world
kubectl expose deployment europe --port=8888 --target-port=80 --type=ClusterIP --namespace=world
```

---

## 🔀 **Step 4: Create the Ingress Resource**
Create a file `ingress.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: world
  namespace: world
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: world.universe.mine
    http:
      paths:
      - path: /africa
        pathType: Prefix
        backend:
          service:
            name: africa
            port:
              number: 8888
      - path: /europe
        pathType: Prefix
        backend:
          service:
            name: europe
            port:
              number: 8888
```

Apply the ingress configuration:
```sh
kubectl apply -f ingress.yaml
```

---

## 🖥 **Step 5: Update `/etc/hosts`**
On your local machine, add the following line to `/etc/hosts` (Linux/macOS) or `C:\Windows\System32\drivers\etc\hosts` (Windows):

```
<your-k8s-node-ip> world.universe.mine
```

Replace `<your-k8s-node-ip>` with the actual IP of your Kubernetes node.

---

## ✅ **Step 6: Verify Setup**
Check if the resources are created successfully:
```sh
kubectl get deployments -n world
kubectl get services -n world
kubectl get ingress -n world
```

Test access using:
```sh
curl http://world.universe.mine/africa
curl http://world.universe.mine/europe
```

---

### 🎉 **End of Message (EOMGY)** 🚀







![Screenshot 2025-03-15 123736](https://github.com/user-attachments/assets/976dd8c1-0f70-469f-ab4b-becb93cf60ff)

# LAB - 5 🖥️🚀

## Overview
This lab focuses on **Role-Based Access Control (RBAC)** and **HELM** in Kubernetes. The tasks involve setting up user permissions and deploying applications using Helm.

---

## 1️⃣ RBAC (10 points) 🔐
### Objective:
Configure **RBAC policies** for the user `smoke` with specific permissions.

### Tasks:
- ✅ Grant `smoke` the ability to **create and delete** Pods, Deployments, and StatefulSets in the **applications** namespace.
- ✅ Provide `smoke` with **view permissions** (similar to the default `view` ClusterRole) in **all namespaces except `kube-system`**.
- ✅ Verify permissions using the following command:
  ```sh
  kubectl auth can-i <action> --as=smoke
  ```

---

## 2️⃣ HELM (10 points) 🎩⚓
### Objective:
Deploy an **Apache web server** using **Helm**.

### Task:
- ✅ Install the Apache Helm chart using:
  ```sh
  helm install my-apache bitnami/apache
  ```

---

## ✅ Completion Checklist
- [ ] Configured RBAC policies correctly 🛠️
- [ ] Verified permissions using `kubectl auth can-i` ✅
- [ ] Installed Apache using Helm 📦
- [ ] Confirmed deployment is running 🖥️

### 💡 Need Help?
For any issues, refer to Kubernetes [RBAC documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) or Helm [official documentation](https://helm.sh/docs/).

🚀 Happy Learning! 🎯

