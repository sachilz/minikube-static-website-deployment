# Kubernetes Deployment Explained (`deployment.yaml`)

This guide explains the Kubernetes [deployment.yaml](file:///c:/Users/sachintha/Desktop/minikube-static-website-deployment/k8s/deployment.yaml) file in simple, beginner-friendly terms.

---

## What is a Kubernetes Deployment?

In Kubernetes, you usually don't run containers or pods directly by hand. Instead, you create a **Deployment**.

Think of a **Deployment** as a **manager**:
- You tell the manager: *"I want 2 copies of my website running at all times."*
- If one copy crashes or dies, the manager automatically creates a new one to replace it.
- If you want to update the website, the manager rolls out the new version smoothly without downtime.

---

## Line-by-Line Code Breakdown

Here is the complete [deployment.yaml](file:///c:/Users/sachintha/Desktop/minikube-static-website-deployment/k8s/deployment.yaml) file with simple explanations for each section:

### 1. Object Type & Version
```yaml
apiVersion: apps/v1
kind: Deployment
```
- **`apiVersion: apps/v1`**: Specifies which Kubernetes API schema version to use. The `apps/v1` API group manages controllers like Deployments and StatefulSets.
- **`kind: Deployment`**: Tells Kubernetes what kind of object you want to create (a Deployment).

---

### 2. Metadata
```yaml
metadata:
  name: static-website
  labels:
    app: static-website
```
- **`name: static-website`**: The unique name for this Deployment in your cluster.
- **`labels`**: Tags (key-value pairs) used to organize and identify this resource.

---

### 3. Deployment Specification (`spec`)
```yaml
spec:
  replicas: 2
```
- **`replicas: 2`**: Tells Kubernetes to always run **2 identical copies** (Pods) of this website. If one pod fails or node restarts, Kubernetes automatically starts a replacement pod.

---

### 4. Pod Selector
```yaml
  selector:
    matchLabels:
      app: static-website
```
- **`selector`**: How the Deployment manager knows which Pods belong to it.
- **`matchLabels: app: static-website`**: The Deployment looks for any Pod with the label `app: static-website` and takes responsibility for managing it.

---

### 5. Pod Template (`template`)
```yaml
  template:
    metadata:
      labels:
        app: static-website
```
- **`template`**: The "cookie cutter" or blueprint used to stamp out each of the 2 Pods.
- **`labels: app: static-website`**: Every Pod created from this template is stamped with this label so the `selector` above recognizes it.

---

### 6. Container Definition (`spec.template.spec.containers`)
```yaml
    spec:
      containers:
        - name: nginx
          image: static-website:v1
          imagePullPolicy: Never
```
- **`containers`**: A list of containers running inside each Pod (in our case, just one container).
- **`name: nginx`**: The name given to this container inside the Pod.
- **`image: static-website:v1`**: The container image we built earlier containing Nginx and our HTML website files.
- **`imagePullPolicy: Never`** *(Very Important!)*:
  - By default, Kubernetes tries to pull images from Docker Hub or the internet.
  - Setting this to `Never` forces Kubernetes to **only use the image built locally in Minikube**. It prevents image pull errors (`ErrImagePull`).

---

### 7. Container Ports
```yaml
          ports:
            - containerPort: 80
```
- **`containerPort: 80`**: Documents that Nginx inside the container is listening for incoming web traffic on HTTP port `80`.

---

### 8. Resource Requests & Limits
```yaml
          resources:
            requests:
              cpu: "50m"
              memory: "32Mi"
            limits:
              cpu: "250m"
              memory: "128Mi"
```
Resource management keeps your cluster stable so one buggy container cannot hog all your computer's RAM or CPU.

- **`requests` (Guaranteed Minimum)**:
  - The minimum resources Kubernetes guarantees to reserve for the container before scheduling it onto a node.
  - `cpu: "50m"`: 50 millicores (0.05 of a CPU core).
  - `memory: "32Mi"`: 32 Megabytes of RAM.

- **`limits` (Hard Maximum)**:
  - The ceiling that the container cannot exceed.
  - `cpu: "250m"`: 250 millicores (0.25 of a CPU core). If exceeded, CPU is throttled.
  - `memory: "128Mi"`: 128 Megabytes of RAM. If exceeded, the container is restarted to protect the node.

---

## Real-World Analogy

| Concept | Analogy |
| :--- | :--- |
| **Deployment** | Restaurant manager |
| **`replicas: 2`** | "We must always have 2 chefs on duty" |
| **`selector`** | "Any worker wearing a blue apron (`app: static-website`) is one of our chefs" |
| **`template`** | The hiring & uniform requirements for each chef |
| **`image: static-website:v1`** | The recipe book the chef follows |
| **`imagePullPolicy: Never`** | "Use our kitchen's internal pantry; don't order from external suppliers" |
| **`resources`** | Minimum and maximum kitchen workspace allocated to each chef |

---

## Useful Commands for this Deployment

To apply this deployment to your Minikube cluster:
```bash
kubectl apply -f k8s/deployment.yaml
```

To see the status of the deployment:
```bash
kubectl get deployments
```

To see the 2 pods created by this deployment:
```bash
kubectl get pods -l app=static-website
```

To view logs from the pods:
```bash
kubectl logs -l app=static-website
```
