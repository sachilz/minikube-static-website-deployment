# Minikube Static Website Deployment

## What is Minikube?

**Minikube** is a tool that runs a small, local Kubernetes cluster on your
computer. It is useful for learning Kubernetes and testing applications
without needing a cloud server.

In simple terms:

- **Kubernetes** manages and runs containers.
- **Minikube** runs Kubernetes locally.
- **Docker** packages an application into a container.
- **Nginx** serves the static website.

## What this project does

This project demonstrates how to deploy a simple static website to a local
Kubernetes cluster using Minikube. The website is packaged in a Docker
container and served by Nginx.

The deployment flow is:

1. Create a Docker image containing the website.
2. Start Minikube.
3. Deploy the image to Kubernetes.
4. Expose the website through a Kubernetes Service.
5. Open the website in a browser.

## Basic Minikube commands

Start Minikube:

```bash
minikube start
```

Check the cluster status:

```bash
minikube status
```

Open the deployed service:

```bash
minikube service <service-name>
```

Stop Minikube when finished:

```bash
minikube stop
```

## Requirements

- Docker
- Minikube
- kubectl

Minikube is a convenient way to practice Kubernetes locally before deploying
an application to a larger cluster or a cloud provider.
