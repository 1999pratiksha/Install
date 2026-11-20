Installation kubernetes on master and worker node
First we are installing Kubeadm to run self managed cluster

Refer install repo on github kubernetes. Every time check the version and run 
Create 1 ec2 instance for Master and ssh it  >select t3 small 2cpu and 2gb ram required
In github repo Go to install>kubernetes>raw >copy the URL
In master node cmd > curl -s <github url in > | bash -s master" >copy the join command
In ec2 master instance allow security group as all traffic for learning purpose
Create 2 ec2 instance for worker nodes >ssh it and run github url repo curl -s <github url in > | bash -s worker
In worker node copy the join commands from master 
if we forgot not have join command then run On MASTER run:  sudo kubeadm token create --print-join-command
Once all nodes joined check it from master node kubectl get nodes

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


# Installation of Kubernetes on Master and Worker Nodes Using Kubeadm

## Overview

We are setting up a self-managed Kubernetes cluster using **kubeadm** on AWS EC2 instances.

We will create:
- 1 Master Node (Control Plane)
- 2 Worker Nodes

We will use a GitHub installation script to install the required Kubernetes components on each EC2 instance.

## 1. Create EC2 Instances

Create three Ubuntu EC2 instances.

| Node | Instance Type | vCPU | RAM |
|---|---|---:|---:|
| Master | t3.small | 2 | 2 GB |
| Worker 1 | t3.small | 2 | 2 GB |
| Worker 2 | t3.small | 2 | 2 GB |

**Note:** These are small lab instances. For production or larger workloads, use more CPU and memory.

## 2. Configure Security Group

For learning purposes, From master node security group allow all traffic

Configure the following inbound rules:

| Type | Port | Source | Purpose |
|---|---:|---|---|
| SSH | 22 | My IP | SSH access |
| Custom TCP | 6443 | Cluster security group | Kubernetes API server |
| Custom TCP | 10250 | Cluster security group | Kubelet communication |
| All traffic | All | Same security group | Lab cluster communication |

**Important:** Allowing all traffic from the same cluster security group is convenient for a learning environment. Do not allow all traffic from `0.0.0.0/0`. For production, configure only the required ports and protocols.

## 3. Install Kubernetes on Master Node

### Step 1: Connect to the Master EC2 instance

SSH into the master instance:

```bash
ssh -i key.pem ubuntu@<MASTER-PUBLIC-IP>
```

### Step 2: Copy the installation script URL

1. Open the GitHub installation repository.
2. Navigate to `Install → Kubernetes_install`.
3. Open the Raw file.
4. Copy the Raw GitHub URL.
5. Check the Kubernetes version and installation script before executing it.

### Step 3: Execute the installation script

Run the following command on the Master node:

```bash
curl -fsSL <GITHUB-RAW-URL> | bash -s master
```

The `master` argument tells the script to configure the instance as the Kubernetes control-plane node.

The script installs the required components and initializes the Kubernetes master node using kubeadm.

### Step 4: Generate the Worker Join Command

After the master installation is completed, run:

```bash
sudo kubeadm token create --print-join-command
```

This generates a command similar to:

```bash
sudo kubeadm join <MASTER-PRIVATE-IP>:6443 \
--token <TOKEN> \
--discovery-token-ca-cert-hash sha256:<HASH>
```

**Copy and save the complete join command.** We will use it to connect the worker nodes to the master.

## 4. Install Kubernetes on Worker Nodes

### Step 1: Connect to Worker 1

SSH into the first worker EC2 instance:

```bash
ssh -i key.pem ubuntu@<WORKER-1-PUBLIC-IP>
```

### Step 2: Execute the installation script

Run:

```bash
curl -fsSL <GITHUB-RAW-URL> | bash -s worker
```

The `worker` argument tells the script to configure the instance as a Kubernetes worker node.

### Step 3: Join Worker 1 to the Cluster

Paste and execute the join command generated from the Master node:

```bash
sudo kubeadm join <MASTER-PRIVATE-IP>:6443 \
--token <TOKEN> \
--discovery-token-ca-cert-hash sha256:<HASH>
```

Repeat the same steps on Worker 2.

**Note:** Generate a fresh join command on the master if the previous token has expired.

## 5. If You Forgot the Join Command

If you forgot to save the join command, connect to the Master node and execute:

```bash
sudo kubeadm token create --print-join-command
```

Copy the newly generated command and execute it on the worker nodes.

## 6. Verify the Kubernetes Cluster

After joining both worker nodes, connect to the Master node.

Run:

```bash
kubectl get nodes
```

Expected output:

```text
NAME                 STATUS   ROLES           VERSION
ip-172-31-11-123     Ready    control-plane   v1.xx.x
ip-172-31-xx-xxx     Ready    <none>          v1.xx.x
ip-172-31-xx-xxx     Ready    <none>          v1.xx.x
```

Check all Kubernetes pods:

```bash
kubectl get pods -A
```

Check the cluster information:

```bash
kubectl cluster-info
```

## 7. Final Result

Our self-managed Kubernetes cluster is successfully configured with:

- 1 Master Node (Control Plane)
- 2 Worker Nodes
- kubeadm for cluster initialization
- kubelet for node management
- kubectl for cluster administration
- containerd as the container runtime
- Calico for pod networking (if configured by the installation script)

We can now deploy applications and manage containers using Kubernetes.

---

**Important:** Always check the Kubernetes version and the GitHub installation script before running it. Make sure the master and worker nodes use compatible Kubernetes versions.
