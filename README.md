# Minikube Static Website Deployment

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.37.0-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Minikube](https://img.shields.io/badge/Minikube-v1.39.0-2E7D32?logo=minikube&logoColor=white)](https://minikube.sigs.k8s.io/)
[![Docker](https://img.shields.io/badge/Docker-Containerd-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Nginx](https://img.shields.io/badge/Nginx-Alpine-009639?logo=nginx&logoColor=white)](https://nginx.org/)
[![HTML5](https://img.shields.io/badge/HTML5-CSS3-E34F26?logo=html5&logoColor=white)](https://developer.mozilla.org/)

An end-to-end DevOps project demonstrating how to containerize a modern static web application using **Docker** and **Nginx**, build it directly inside a local **Minikube** cluster, and deploy it with high availability using **Kubernetes Deployments** and **NodePort Services**.

---

## Architecture Diagram

![Minikube Static Website Deployment Architecture](./Minikube%20Static%20Website%20Deployment%20Architecture.png)

---

## Architecture Overview

The system architecture is structured into two main phases: **Build & Deploy Pipeline** and **Cluster Runtime Architecture**.

### 1. Build and Deploy Pipeline
1. **Source Code (`website/`)**: A responsive, dark-mode static website structured in clean HTML5 & CSS3.
2. **Dockerfile**: A lightweight, multi-layer image based on `nginx:alpine` that copies the static assets to `/usr/share/nginx/html/` and exposes HTTP port `80`.
3. **In-Cluster Build (`static-website:v1`)**: Built directly into Minikube’s local container runtime cache using `minikube image build`, eliminating the need to push images to an external registry like Docker Hub.
4. **Kubernetes Manifests (`k8s/`)**: Declarative configuration files applied to the cluster using `kubectl apply`.

### 2. Cluster Runtime Architecture
- **Control Plane & Worker Node**: Single-node Minikube cluster powered by the Docker driver.
- **Deployment (`static-website`)**: Manages **2 replica Pods** ensuring high availability, automatic self-healing, and zero-downtime rolling updates.
- **NodePort Service (`static-website-service`)**: Exposes port `80` from the pods through node port `30080`, providing a stable internal IP, cluster DNS, and automatic load balancing across both pods.
- **Traffic Routing**: Traffic from the client browser travels through the Minikube tunnel (`http://127.0.0.1:<allocated-port>`) into the NodePort service and is balanced between the running Nginx container pods.

---

## Technologies Used

| Technology | Role & Purpose |
| :--- | :--- |
| **[Docker](https://www.docker.com/)** | Packages the website and Nginx web server into an immutable container image. |
| **[Kubernetes](https://kubernetes.io/)** | Container orchestrator automating deployment, scaling, health-checking, and networking. |
| **[Minikube](https://minikube.sigs.k8s.io/)** | Local single-node Kubernetes cluster running inside Docker for fast development and testing. |
| **[Nginx (Alpine)](https://nginx.org/)** | Ultra-lightweight, high-performance web server serving static files on port 80. |
| **HTML5 & CSS3** | Clean, dark-mode frontend presentation layer. |

---

## Project Structure

```text
minikube-static-website-deployment/
├── Minikube Static Website Deployment Architecture.png  # High-level architecture diagram
├── Dockerfile                                          # Container blueprint using nginx:alpine
├── README.md                                           # Project documentation
│
├── website/                                            # Application source code
│   └── index.html                                      # DevOps landing page
│
├── k8s/                                                # Kubernetes declarative manifests
│   ├── deployment.yaml                                 # Deployment config (2 replicas, limits)
│   └── service.yaml                                    # NodePort Service config (port 30080)
│
├── learning/                                           # Educational line-by-line guides
│   ├── deployment.md                                   # Comprehensive Deployment explanation
│   └── service.md                                      # Comprehensive Service explanation
│
└── code-commands/                                      # Operations & CLI handbook
    └── commands.md                                     # Step-by-step CLI execution log
```

---

## Step-by-Step Deployment Guide

### Prerequisites
Make sure the following tools are installed on your workstation (Windows / WSL2 / Linux / macOS):
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) or Docker Engine
- [Minikube](https://minikube.sigs.k8s.io/docs/start/) (`>= v1.30`)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

---

### Step 1: Start the Local Minikube Cluster
Spin up a local Kubernetes cluster with Docker as the virtualization driver:

```bash
minikube start --driver=docker
```

Verify that the cluster control plane is healthy:

```bash
minikube status
kubectl get nodes
```

---

### Step 2: (Optional) Test Locally with Docker
Before deploying to Kubernetes, verify the Docker packaging locally:

```bash
# Build the test image
docker build -t static-website:test .

# Run a test container on host port 8081
docker run --rm -d --name static-website-test -p 8081:80 static-website:test

# Verify Nginx serves the HTML
curl http://localhost:8081

# Stop the test container
docker stop static-website-test
```

---

### Step 3: Build Image Directly Inside Minikube
Build the container image directly inside Minikube's internal Docker runtime so Kubernetes can access it without pulling from Docker Hub:

```bash
minikube image build -t static-website:v1 .
```

Verify the image is in Minikube's local cache:

```bash
minikube image ls | grep static-website
```

---

### Step 4: Deploy to Kubernetes
Apply the declarative manifests to create the Deployment and NodePort Service:

```bash
# 1. Create the Deployment (2 replica pods)
kubectl apply -f k8s/deployment.yaml

# 2. Create the NodePort Service
kubectl apply -f k8s/service.yaml
```

---

### Step 5: Verify the Deployment & Networking

Check that the deployment created 2 healthy pods:
```bash
kubectl get deployments
```
*Expected Output:* `READY 2/2`

Inspect the pods and their assigned cluster IP addresses:
```bash
kubectl get pods -o wide
```

Check the active service and port bindings:
```bash
kubectl get services
```
*Expected Output:* `static-website-service   NodePort   ...   80:30080/TCP`

---

### Step 6: Access the Website in Your Browser

Because Minikube is running with the Docker driver inside an isolated network container, create a secure tunnel to expose the service to your host browser:

```bash
minikube service static-website-service
```

Or to retrieve the direct access URL:

```bash
minikube service static-website-service --url
```
*Example Output:* `http://127.0.0.1:44901`

Open the generated URL in any web browser to view your live containerized website!

> **Note:** Keep the terminal running the `minikube service` tunnel open while accessing the site.

---

## Detailed Learning Guides

For in-depth explanations designed for learning and interviews, explore the companion documentation:

- 📘 **[Kubernetes Deployment Deep Dive](learning/deployment.md)**: Line-by-line explanation of `apps/v1`, replicas, pod templates, selectors, and CPU/memory resource limits.
- 📙 **[Kubernetes Service Deep Dive](learning/service.md)**: Thorough explanation of `NodePort`, cluster DNS, and the distinction between `port`, `targetPort`, and `nodePort`.
- 📗 **[Commands Reference Handbook](code-commands/commands.md)**: Complete record of every CLI command executed with terminal outputs and flags explained.

---

## Teardown and Cleanup

When you are finished testing, clean up the cluster resources:

```bash
# Delete the Kubernetes service and deployment
kubectl delete -f k8s/service.yaml
kubectl delete -f k8s/deployment.yaml

# Stop the Minikube virtual cluster
minikube stop

# (Optional) Delete the Minikube cluster completely to free disk space
minikube delete
```

---

## Key Learnings & Takeaways

1. **Local Kubernetes Workflow**: How to use Minikube with `--driver=docker` to create a lightweight, fully functional Kubernetes cluster without cloud costs.
2. **Registry-Free Builds**: Using `minikube image build` combined with `imagePullPolicy: Never` allows immediate testing of custom images without publishing to public or private registries.
3. **High Availability & Self-Healing**: Setting `replicas: 2` ensures that if any pod terminates unexpectedly, the Deployment controller automatically provisions an identical replacement within seconds.
4. **Traffic Distribution**: The NodePort Service transparently load balances incoming client traffic across all healthy pod replicas matching the `app: static-website` selector.
