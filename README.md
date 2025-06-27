# 🪄 Goblet of Fire Application 🏆

## ✨ The Story: Why the Goblet of Fire? 🧙‍♂️

In the magical world of Harry Potter, the Goblet of Fire is a legendary artifact used to select champions for the Triwizard Tournament. It is a symbol of fate, courage, and the spirit of competition. This application brings that magic to life in the digital realm—allowing users to submit their names to the Goblet, experience the thrill of selection, and see their fate unfold. Whether you're a wizard, witch, or Muggle, step up and let the Goblet decide your destiny! ⚡️🪄

A Harry Potter-inspired full-stack application where users can submit their names to the Goblet of Fire. Built with Next.js, Node.js, .NET, Redis, and MongoDB.

---

## 🗂️ Table of Contents
- [🏰 Project Structure](#project-structure)
- [🐳 Running Locally with Docker Desktop](#running-locally-with-docker-desktop)
- [☁️ Deploying to Azure Kubernetes Service (AKS)](#deploying-to-azure-kubernetes-service-aks)
- [🌐 Service URLs](#service-urls)
- [🧹 Troubleshooting](#troubleshooting)
- [Docker Build and Push Instructions for All Services](#docker-build-and-push-instructions-for-all-services)

---

## 🏰 Project Structure

- `frontend/` - Next.js frontend (Harry Potter themed) ✨
- `api/` - Node.js/Express API (connects to MongoDB & Redis) 🦉
- `admin-app/` - Admin dashboard (Node.js/Express) 🧑‍💼
- `worker/` - .NET Worker service ⚙️
- `k8s-specification/` - Kubernetes YAML manifests 📦
- `docker-compose.yml` - Docker Compose file for local development 🐳

---

## 🐳 Running Locally with Docker Desktop

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running 🐳
- [Git](https://git-scm.com/) installed 🧙‍♂️

### 1. Clone the Repository
```sh
git clone <your-repo-url>
cd goblet-of-fire
```

### 2. Environment Variables
Ensure `.env` files exist in `api/` and `admin-app/` with correct MongoDB and Redis connection strings. Example:

**api/.env**
```
MONGODB_URI=mongodb://mongo:27017/goblet
REDIS_URL=redis://redis:6379
```

**admin-app/.env**
```
MONGODB_URI=mongodb://mongo:27017/goblet
```

### 3. Build and Start All Services
```sh
docker-compose up --build
```
This will build and start:
- MongoDB (port 27017) 🍃
- Redis (port 6379) 🟥
- API (port 4000) 🦉
- Admin App (port 5000) 🧑‍💼
- Frontend (port 3000) ✨
- Worker (port 8080) ⚙️

### 4. Access the Applications
- **Frontend:** http://localhost:3000 ✨
- **API:** http://localhost:4000 🦉
- **Admin App:** http://localhost:5000 🧑‍💼
- **Worker:** http://localhost:8080 ⚙️
- **MongoDB:** localhost:27017 🍃
- **Redis:** localhost:6379 🟥

---

## ☁️ Deploying to Azure Kubernetes Service (AKS)

### Prerequisites
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) installed ☁️
- [kubectl](https://kubernetes.io/docs/tasks/tools/) installed 🐙
- [Docker](https://www.docker.com/products/docker-desktop/) installed 🐳
- Azure subscription 🪄

### 1. Login to Azure
```sh
az login
```

### 2. Create Resource Group & AKS Cluster
```sh
az group create --name goblet-rg --location <azure-region>
az aks create --resource-group goblet-rg --name goblet-aks --node-count 2 --generate-ssh-keys
```

### 3. Connect kubectl to AKS
```sh
az aks get-credentials --resource-group goblet-rg --name goblet-aks
```

### 4. Create Azure Container Registry (ACR)
```sh
az acr create --resource-group goblet-rg --name <youracr> --sku Basic
az acr login --name <youracr>
```

### 5. Build & Push Docker Images to ACR
Replace `<youracr>` with your ACR name.
```sh
docker build -t <youracr>.azurecr.io/goblet-api:latest ./api
# Repeat for admin-app, frontend, worker
docker build -t <youracr>.azurecr.io/goblet-admin:latest ./admin-app
docker build -t <youracr>.azurecr.io/goblet-frontend:latest ./frontend
docker build -t <youracr>.azurecr.io/goblet-worker:latest ./worker
az acr login --name <youracr>
docker push <youracr>.azurecr.io/goblet-api:latest
docker push <youracr>.azurecr.io/goblet-admin:latest
docker push <youracr>.azurecr.io/goblet-frontend:latest
docker push <youracr>.azurecr.io/goblet-worker:latest
```

### 6. Update Image Names in Kubernetes YAMLs
In `k8s-specification/*-deployment.yml`, set the `image:` fields to your ACR images.

### 7. Deploy MongoDB and Redis
```sh
kubectl apply -f k8s-specification/mongo-deployment.yml
kubectl apply -f k8s-specification/redis-deployment.yml
kubectl apply -f k8s-specification/mongo-service.yml
kubectl apply -f k8s-specification/redis-service.yml
```

### 8. Deploy Application Services
```sh
kubectl apply -f k8s-specification/api-deployment.yml
kubectl apply -f k8s-specification/admin-app-deployment.yml
kubectl apply -f k8s-specification/frontend-deployment.yml
kubectl apply -f k8s-specification/worker-deployment.yml
kubectl apply -f k8s-specification/api-service.yml
kubectl apply -f k8s-specification/admin-app-service.yml
kubectl apply -f k8s-specification/frontend-service.yml
kubectl apply -f k8s-specification/worker-service.yml
```

### 9. Expose Frontend (LoadBalancer)
The frontend service is of type `LoadBalancer`. Get the external IP:
```sh
kubectl get svc goblet-frontend
```
Access the app at the EXTERNAL-IP shown.

### 10. (Optional) Ingress Setup
For custom domains or HTTPS, set up an Ingress controller (e.g., NGINX) and DNS records. 🧙‍♂️

---

## 🌐 Service URLs
- **Frontend:**
  - Docker: http://localhost:3000 ✨
  - AKS: http://<EXTERNAL-IP-from-kubectl>
- **API:**
  - Docker: http://localhost:4000 🦉
  - AKS: http://<cluster-ip>:4000 (internal)
- **Admin App:**
  - Docker: http://localhost:5000 🧑‍💼
  - AKS: http://<cluster-ip>:5000 (internal)
- **Worker:**
  - Docker: http://localhost:8080 ⚙️
  - AKS: http://<cluster-ip>:8080 (internal)

---

## 🧹 Troubleshooting
- Use `docker-compose logs` or `kubectl logs <pod>` for debugging. 🔍
- Ensure all environment variables are set correctly. 📝
- For AKS, make sure images are pushed to ACR and accessible. ☁️
- Check firewall/networking rules if services are not reachable. 🛡️

---

## Docker Build and Push Instructions for All Services

This section provides step-by-step instructions to build and push Docker images for each service to Docker Hub. Replace `iamcknitin` with your Docker Hub username if different.

### Prerequisites
- Docker installed and running
- Docker Hub account (https://hub.docker.com/)
- Logged in to Docker Hub (`docker login`)

---

### 1. Build and Push `goblet-frontend`

```powershell
cd frontend
# Build the Docker image
docker build -t iamcknitin/goblet-frontend:latest .
# Push the image to Docker Hub
docker push iamcknitin/goblet-frontend:latest
cd ..
```

---

### 2. Build and Push `goblet-api`

```powershell
cd api
# Build the Docker image
docker build -t iamcknitin/goblet-api:latest .
# Push the image to Docker Hub
docker push iamcknitin/goblet-api:latest
cd ..
```

---

### 3. Build and Push `goblet-admin`

```powershell
cd admin-app
# Build the Docker image
docker build -t iamcknitin/goblet-admin:latest .
# Push the image to Docker Hub
docker push iamcknitin/goblet-admin:latest
cd ..
```

---

### 4. Build and Push `goblet-worker`

```powershell
cd worker
# Build the Docker image
docker build -t iamcknitin/goblet-worker:latest .
# Push the image to Docker Hub
docker push iamcknitin/goblet-worker:latest
cd ..
```

---

**Note:**
- Ensure you are in the project root directory before running these commands.
- The `latest` tag is used for simplicity; you can use versioned tags as needed.
- Update your Kubernetes deployment files to use these Docker Hub image URLs.

---

## 📝 Notes
- Update all `<youracr>` and `<your-repo-url>` placeholders with your actual values. ✏️
- For production, consider using Azure Key Vault for secrets and configure persistent storage for MongoDB/Redis. 🗝️

---

Enjoy your magical Goblet of Fire app! 🪄
