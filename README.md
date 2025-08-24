# Containerized Web App (Flask + Docker + Kubernetes + Jenkins CI/CD)

This repo contains a simple Flask app, Dockerfile, Kubernetes manifests, and a Jenkins pipeline to build, push, and deploy the app to a Kubernetes cluster with autoscaling.

## Prerequisites
- Docker (local build/push)
- Docker Hub account
- Kubernetes cluster (minikube for local, or EKS/GKE/AKS)
- `kubectl` configured to talk to your cluster
- Jenkins with:
  - Docker installed on the agent
  - `kubectl` available on the agent and KUBECONFIG configured
  - Credentials (ID: `dockerhub-pass`) of type "Username with password" for Docker Hub

## Replace placeholders
In files:
- `k8s/deployment.yaml` → set `image: your-dockerhub-username/flask-app:latest`
- `Jenkinsfile` → set `REGISTRY = "your-dockerhub-username"`

## Local build & test
```bash
docker build -t your-dockerhub-username/flask-app:latest .
docker run -d -p 5000:5000 your-dockerhub-username/flask-app:latest
# visit http://localhost:5000
```

## Push to Docker Hub
```bash
docker login
docker push your-dockerhub-username/flask-app:latest
```

## Kubernetes deploy (first time)
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/hpa.yaml
kubectl get pods,svc,hpa
```

- On minikube: `minikube service flask-service`
- On cloud: use Service EXTERNAL-IP

## Jenkins CI/CD
1. Create credentials in Jenkins:
   - ID: `dockerhub-pass` (Username with password → Docker Hub)
2. Create a Pipeline job pointing to this repo.
3. (Optional first-time apply): Add a **boolean parameter** named `APPLY_MANIFESTS` and tick it for the first build to apply k8s manifests.
4. Pipeline stages:
   - Build Docker image
   - Push image to Docker Hub
   - Rolling update on Kubernetes (`kubectl set image` + `kubectl rollout status`)

## Scale & High Availability
- Deployment runs `replicas: 3`
- HPA scales from 3 to 10 based on 50% CPU
- Service type LoadBalancer for traffic distribution

## Cleanup
```bash
kubectl delete -f k8s/hpa.yaml
kubectl delete -f k8s/service.yaml
kubectl delete -f k8s/deployment.yaml
```
