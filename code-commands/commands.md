# Minikube & Kubectl Commands Guide

This document explains the commands used to start, inspect, and monitor the local Kubernetes cluster using **Minikube** and **kubectl**.

---

## 1. `minikube start --driver=docker`

### What it does:
Starts a local single-node Kubernetes cluster on your computer.

### Explanation:
- **`minikube start`**: Initializes the virtualized environment, downloads required images, configures Kubernetes components, and sets up your local `kubectl` configuration.
- **`--driver=docker`**: Instructs Minikube to run the entire Kubernetes cluster inside a **Docker container** rather than a heavy virtual machine (like VirtualBox or Hyper-V). This makes startup faster and uses fewer system resources.

### What happened in your terminal:
1. Minikube checked for an existing container.
2. Re-created the `minikube` Docker container with 2 CPUs and 3072MB RAM.
3. Installed Kubernetes components (containerd, CNI network, storage provisioner).
4. Configured `kubectl` to automatically connect to this cluster.

---

## 2. `minikube status`

### What it does:
Checks the current health and operational status of your local Minikube cluster.

### Explanation:
This command displays four core components:
- **`host: Running`**: The underlying Docker container hosting the Minikube node is active.
- **`kubelet: Running`**: The node agent that communicates with the API server and manages containers on the node is healthy.
- **`apiserver: Running`**: The Kubernetes API server (the control hub of Kubernetes) is accepting requests.
- **`kubeconfig: Configured`**: Your local credentials file (`~/.kube/config`) is properly pointing to this Minikube cluster.

---

## 3. `kubectl get nodes`

### What it does:
Displays all the nodes (machines or containers acting as servers) in the Kubernetes cluster.

### Explanation:
Minikube is a single-node cluster, so it lists one node:
- **`NAME` (`minikube`)**: The identifier of the node.
- **`STATUS` (`NotReady` / `Ready`)**:
  - `NotReady`: The node is still initializing networking plugins (CNI) and system pods right after startup.
  - `Ready`: The node is fully prepared to schedule and run your application pods.
- **`ROLES` (`control-plane`)**: This node manages the cluster control plane (scheduling, state management, API).
- **`AGE`**: Time elapsed since the node was created.
- **`VERSION` (`v1.37.0`)**: The Kubernetes version running on the node.

---

## 4. `kubectl get pods -A`

### What it does:
Lists all running or starting Pods across **all namespaces** in the cluster.

### Explanation:
- **`kubectl get pods`**: Retrieves pods in the default namespace.
- **`-A` (or `--all-namespaces`)**: Tells kubectl to show pods in every namespace, including system namespaces.

### System Pods Explained:
All these pods run under the **`kube-system`** namespace to keep Kubernetes functioning:

| Pod Name | Purpose in Simple Terms |
| :--- | :--- |
| **`kube-apiserver`** | The front door to Kubernetes. Receives all commands from `kubectl` and coordinates the cluster. |
| **`etcd`** | The cluster's database. Stores all cluster state, secrets, and configuration. |
| **`kube-scheduler`** | The matchmaker. Decides which node should run a newly created pod based on available resources. |
| **`kube-controller-manager`** | The supervisor. Constantly monitors the cluster to make sure the current state matches the desired state. |
| **`kube-proxy`** | The traffic cop. Directs network traffic and manages IP routing rules for services. |
| **`kindnet`** | The network cable. Container Network Interface (CNI) that assigns IP addresses and enables communication between pods. |
| **`coredns`** | Internal phonebook. Provides DNS service discovery so pods can find each other by name. |
| **`storage-provisioner`** | Hard drive manager. Automatically creates local persistent storage volumes for pods when requested. |

---

## 5. `ls -R`

### What it does:
Recursively lists all files and folders in the current directory and all subdirectories.

### Explanation:
- **`ls`**: Lists the files and folders inside the directory you are currently in.
- **`-R` (Recursive)**: Tells the command to look inside every subfolder, and inside subfolders of those subfolders, giving you the complete directory tree structure.

### Why it is useful in this project:
It allows you to quickly verify that all your project assets (such as `website/index.html`, Dockerfiles, or documentation files) are placed in their proper folders before building and deploying.

Example output:
```text
.:
code-commands  README.md  website

./code-commands:
commands.md

./website:
index.html
```

---

## 6. `minikube image build -t static-website:v1 .`

### What it does:
Builds a Docker container image directly inside Minikube's internal container storage.

### Explanation:
- **`minikube image build`**: Reads the `Dockerfile` in the specified directory and builds the image inside the Minikube environment. This avoids needing to push the image to an external registry like Docker Hub.
- **`-t static-website:v1`**: Tags the image with a name (`static-website`) and version tag (`v1`).
- **`.`**: Refers to the current directory (the build context) where the `Dockerfile` and `website/` folder are located.

### What happened in your terminal:
1. Minikube loaded the build context and `Dockerfile`.
2. Pulled the base `nginx:alpine` image.
3. Executed `COPY website/ /usr/share/nginx/html/` to copy your HTML files into Nginx.
4. Exported and named the image `docker.io/library/static-website:v1` locally inside the cluster.

---

## 7. `minikube image ls | grep static-website`

### What it does:
Checks and verifies that your built image is available in Minikube's local image registry.

### Explanation:
- **`minikube image ls`**: Lists all container images stored locally inside Minikube.
- **`|` (Pipe)**: Sends the output of `minikube image ls` directly as input to `grep`.
- **`grep static-website`**: Filters the list and prints only lines matching `static-website`.

### What happened in your terminal:
```text
docker.io/library/static-website:v1
```
This confirms that Kubernetes can find and run your website container locally without looking up Docker Hub.

---

## Quick Reference Summary

| Command | Simple Meaning |
| :--- | :--- |
| `minikube start --driver=docker` | "Start my local Kubernetes cluster inside Docker" |
| `minikube status` | "Is my cluster up and healthy?" |
| `kubectl get nodes` | "Show me the servers/nodes running in this cluster" |
| `kubectl get pods -A` | "Show me all containers/pods running everywhere in the cluster" |
| `ls -R` | "List all files and subfolders recursively (show project tree)" |
| `minikube image build -t static-website:v1 .` | "Build Docker image directly inside Minikube from current folder" |
| `minikube image ls \| grep static-website` | "Check if my website image exists in Minikube's image list" |
