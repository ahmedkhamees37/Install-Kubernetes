# 🚀 Kubernetes Installation Using Kubeadm and Flannel with Worker Node Integration

Welcome to this step-by-step guide on setting up a **Kubernetes** cluster! 🎉 We will walk through the process of installing Kubernetes on two Ubuntu virtual machines (VMs), using **Kubeadm** for the setup and **Flannel** as the networking solution. Additionally, you'll deploy a simple **Nginx** application to verify the installation. As a bonus, we will also set up a **K3s** lightweight Kubernetes cluster! 💡

---

## 📚 Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Create Two Ubuntu Virtual Machines](#step-1-create-two-ubuntu-virtual-machines)
- [Step 2: Install Container Runtime](#step-2-install-container-runtime)
- [Step 3: Install Kubernetes Components](#step-3-install-kubernetes-components)
- [Step 4: Initialize Kubernetes Cluster on Control Plane](#step-4-initialize-kubernetes-cluster-on-control-plane)
- [Step 5: Install Flannel CNI](#step-5-install-flannel-cni)
- [Step 6: Join Worker Node to Cluster](#step-6-join-worker-node-to-cluster)
- [Step 7: Deploy Sample Application](#step-7-deploy-sample-application)
- [Bonus: K3s Installation](#bonus-k3s-installation)
- [How to Upload Your README.md to GitHub](#how-to-upload-your-readmemd-to-github)

---

## 🛠️ Prerequisites

- Two **Ubuntu 20.04** virtual machines (VMs) or servers.
- Root or **sudo** privileges on both VMs.
- Internet access to download necessary packages.
- Both servers should be on the same network for communication.

---

## 🔧 Step 1: Create Two Ubuntu Virtual Machines

Ensure you have two **Ubuntu 20.04** VMs with fresh installations. Once the system is ready, run the following commands to update your system:

```bash
sudo apt update && sudo apt upgrade -y

 
## 🚫 Disable Swap
Kubernetes requires swap to be disabled. You can do this with:

```bash
sudo swapoff -a
