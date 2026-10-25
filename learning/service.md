# Kubernetes Service Explained (`service.yaml`)

This guide explains the Kubernetes [service.yaml](file:///c:/Users/sachintha/Desktop/minikube-static-website-deployment/k8s/service.yaml) file in simple, beginner-friendly terms.

---

## Why Do We Need a Service?

Pods in Kubernetes are temporary and disposable. If a Pod crashes or is updated:
1. Kubernetes terminates it and creates a new one.
2. The new Pod gets a **brand new internal IP address**.

If you tried to connect directly to a Pod's IP, your connection would break every time a Pod restarts.

A **Service** solves this problem by acting as a **stable front desk and load balancer**:
- It gives your application a single, permanent IP address and DNS name.
- It distributes incoming traffic evenly across both of your website Pods.

---

## Line-by-Line Code Breakdown

Here is the complete [service.yaml](file:///c:/Users/sachintha/Desktop/minikube-static-website-deployment/k8s/service.yaml) file with an explanation of each line:

### 1. Object Type & Version
```yaml
apiVersion: v1
kind: Service
```
- **`apiVersion: v1`**: The core Kubernetes API version that manages fundamental resources like Services, Pods, and ConfigMaps.
- **`kind: Service`**: Tells Kubernetes that this file describes a networking Service.

---

### 2. Metadata
```yaml
metadata:
  name: static-website-service
```
- **`name: static-website-service`**: The unique name for this service. Other pods inside the cluster can reach your website using this exact name as an internal domain name (e.g., `http://static-website-service`).

---

### 3. Service Type
```yaml
spec:
  type: NodePort
```
- **`type: NodePort`**: Exposes the service outside the Kubernetes cluster by opening a dedicated port on the node.
- **Why NodePort?**
  - **`ClusterIP`** (default): Only accessible from *inside* the cluster.
  - **`NodePort`**: Opens a port on the Minikube node so you can access the website from your personal computer's browser.
  - **`LoadBalancer`**: Used on cloud providers (AWS, GCP, Azure) to provision an external cloud load balancer.

---

### 4. Selector
```yaml
  selector:
    app: static-website
```
- **`selector`**: This is the glue connecting the Service to your Pods.
- Any Pod running in the cluster with the label **`app: static-website`** (defined in your `deployment.yaml`) will automatically receive traffic from this Service.

---

### 5. The Three Ports (Port Mapping)
```yaml
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```
This is often the most confusing part for beginners. Here is how traffic flows through the three ports:

```text
[Your Browser]
      │
      ▼
nodePort: 30080    (Opened on the Minikube machine / Node)
      │
      ▼
   port: 80        (The internal Service IP port)
      │
      ▼
targetPort: 80     (The Nginx container port inside the Pod)
```

1. **`nodePort: 30080`**: The external entry port on your Minikube node. You can access this from your browser at `http://<minikube-ip>:30080`. (Kubernetes reserves the port range `30000-32767` for NodePorts).
2. **`port: 80`**: The port that the Service itself listens on *inside* the cluster.
3. **`targetPort: 80`**: The port *inside the container* where Nginx is actually running and serving web traffic.
4. **`protocol: TCP`**: Standard networking protocol used for HTTP/web traffic.

---

## Real-World Analogy

| Kubernetes Term | Real-World Analogy |
| :--- | :--- |
| **Service** | The main phone reception desk of a company |
| **`selector: app: static-website`** | "Direct all calls about sales to anyone on the sales team" |
| **`nodePort: 30080`** | The public toll-free phone number outsiders call |
| **`port: 80`** | The office switchboard extension |
| **`targetPort: 80`** | The specific phone sitting on the sales agent's desk |

---

## Useful Commands for this Service

To apply this service to your cluster:
```bash
kubectl apply -f k8s/service.yaml
```

To see the service and its assigned ports:
```bash
kubectl get services
# or
kubectl get svc
```

To automatically open the website in your browser via Minikube:
```bash
minikube service static-website-service
```

To get the direct URL without opening a browser:
```bash
minikube service static-website-service --url
```
