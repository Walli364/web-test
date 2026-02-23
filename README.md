# E-Commerce Web Application

## Overview

This project serves as a practical exercise to test your DevOps skills. It involves working with a sample e-commerce web application that allows users to browse and purchase kids' beds. Your task is to apply DevOps practices to containerize the application, set up CI/CD pipelines, and ensure proper validation and testing.

## Project Tasks

1. **Fork the Repository**
   - Begin by forking this repository to your own GitHub account.
     
2. **Ansible Automation (Same Local Machine)**
   - Use Ansible to automate app deployment and updates:
     - Create ansible/ folder with `host` file:
       `[local]`
       `127.0.0.1 ansible_connection=local`
     - install and verify basics tools for this repo e.g. `git` and `docker`

3. **Clone and create a Separate Branch**
   - Clone and create a new branch named `devops-branch` in your forked repository. This branch will be used for all your DevOps-related tasks.

4. **Containerize the Application**
   - Write a `Dockerfile` to containerize the e-commerce web application. Ensure that the Dockerfile is properly configured to build an image for the application. Apply best practices in Dockerfile creation, such as:
     - Minimizing the number of layers
     - Using a small base image
     - Caching dependencies effectively
     - Cleaning up unnecessary files to reduce the image size 

5. **Set Up Docker Compose**
   - Create a `docker-compose.yml` file to deploy the application using Docker Compose. Although PostgreSQL is not used in this application, ensure that the Docker Compose setup is properly configured for the application container.

6. **Kubernetes (Local with Minikube or Kind)**
   - Deploy your app in a local Kubernetes cluster:
   - Create:
     - k8s/deployment.yaml
     - k8s/service.yaml
   - Start Minikube or Kind and apply:
     - kubectl apply -f k8s/
     - kubectl port-forward svc/ecommerce-service 8080:80

7. **Deploy Using CI/CD Pipeline**
   - Use the `Dockerfile` to deploy the application by setting up a CI/CD pipeline. This pipeline should be configured to run only for the `devops-branch`, not for the master branch.
   - Optionally
     - Create new branches named `compose-branch` and `k8s-branch` in your forked repository. These branch will be used for all your docker compose and Kubernetes related tasks.
     - Use the `docker-compose.yml` to deploy the application by setting up a CI/CD pipeline. This pipeline should be configured to run only for the `compose-branch`, not for the master branch.
     - Use the `deployment.yaml` and `service.yaml` to deploy the application by setting up a CI/CD pipeline. This pipeline should be configured to run only for the `k8s-branch`, not for the master branch.

8. **Validation and Testing**
   - Add necessary validation and testing in your CI/CD to confirm that the application runs correctly on a specific port. Ensure that all functionalities are working as expected and the deployment is stable.

9. **Prometheus & Grafana Monitoring**
   - Monitor Docker containers and host system locally:
   - Create monitoring/docker-compose.yml with:
     - prometheus
     - node_exporter
     - grafana

---

## What I Did

### 1. Ansible - Installing Git and Docker

Created an Ansible setup to automate the installation of Git and Docker on the local machine.

**Files Created:**
- `ansible/hosts` - Inventory file for local connection
- `ansible/setup.yml` - Playbook to install and verify tools

**Hosts file content:**
```ini
[local]
127.0.0.1 ansible_connection=local
```

**Commands to run:**
```bash
# Install Ansible (if not installed)
sudo apt install ansible

# Run the playbook
ansible-playbook -i ansible/hosts ansible/setup.yml

# Or run with verbose output
ansible-playbook -i ansible/hosts ansible/setup.yml -v
```

The playbook performs:
- Updates apt package cache
- Installs Git and verifies installation
- Installs Docker and verifies installation

---

### 2. Dockerfile

Created a Dockerfile to containerize the e-commerce web application using Nginx Alpine (minimal image).

**Dockerfile content:**
```dockerfile
FROM nginx:alpine

COPY . /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Commands:**
```bash
# Build the Docker image
docker build -t my-ecommerce-app .

# Run the container
docker run -d -p 8080:80 my-ecommerce-app

# Access the application at http://localhost:8080
```

---

### 3. Docker Compose

Created a docker-compose.yml file for easy deployment of the application.

**docker-compose.yml content:**
```yaml
version: '3.8'

services:
  app:
    container_name: loving_wing
    build: .
    ports:
      - "8080:80"
    restart: always
```

**Commands:**
```bash
# Start the application
docker compose up -d --build

# View running containers
docker compose ps

# Stop the application
docker compose down

# View logs
docker compose logs -f
```

---

### 4. Kubernetes

Created Kubernetes manifests for deploying the application in a local cluster (Minikube or Kind).

**Files Created:**
- `kubernetes/deployment.yml` - Deployment with 2 replicas
- `kubernetes/service.yml` - Service to expose the application

**Commands:**
```bash
# Start Minikube (if using Minikube)
minikube start

# Build and load image (for Kind)
docker build -t my-ecommerce-app:latest .
kind load docker-image my-ecommerce-app:latest

# Apply Kubernetes manifests
kubectl apply -f kubernetes/

# Check pod status
kubectl get pods

# Check service status
kubectl get svc

# Port forward to access the application
kubectl port-forward svc/ecommerce-service 8080:80

# Access the application at http://localhost:8080
```

---

### 5. CI/CD Workflows (GitHub Actions)

Created three separate CI/CD pipelines for different deployment methods.

**Files Created:**
- `.github/workflows/docker-pipeline.yml` - Runs on `devops` branch
- `.github/workflows/compose-pipeline.yml` - Runs on `compose-branch`
- `.github/workflows/k8s-pipeline.yml` - Runs on `kubernetes-branch`

**Docker Pipeline (`devops` branch):**
- Builds Docker image
- Runs container on port 8080
- Validates application is responding

**Docker Compose Pipeline (`compose-branch`):**
- Runs docker compose up
- Verifies containers are running
- Validates application on port 8080

**Kubernetes Pipeline (`kubernetes-branch`):**
- Creates Kind cluster
- Builds and loads image into cluster
- Deploys to Kubernetes
- Port forwards and validates on port 8081

---

### 6. Monitoring (Prometheus & Grafana)

Created monitoring setup using Prometheus for metrics collection and Grafana for visualization.

**Files Created:**
- `monitoring/monitoring-compose.yml` - Docker Compose for monitoring stack
- `monitoring/prometheus.yml` - Prometheus configuration

**Services:**
- **Prometheus** - Metrics collection (Port 9090)
- **Node Exporter** - Host metrics (Port 9100)
- **Grafana** - Visualization dashboard (Port 3001)

**Commands:**
```bash
# Navigate to monitoring directory
cd monitoring

# Start monitoring stack
docker compose -f monitoring-compose.yml up -d

# Access services:
# Prometheus: http://localhost:9090
# Grafana: http://localhost:3001 (admin/admin)

# Stop monitoring stack
docker compose -f monitoring-compose.yml down
```

**Grafana Setup:**
1. Login with admin/admin
2. Add Prometheus data source: http://prometheus:9090
3. Import Node Exporter dashboard (ID: 1860)
