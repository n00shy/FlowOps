# 🚀 DataFlow Platform — Jenkins CI/CD with Docker Compose

A hands-on DevOps project that demonstrates a complete containerized application delivery workflow using **Jenkins, Docker Compose, Docker Hub, Gitleaks, Trivy, Prometheus, Grafana, and cAdvisor**.

The project focuses on automating the build, security scanning, image publishing, deployment, and infrastructure monitoring of a React frontend, Node.js backend, and MongoDB database.

---

## 🏗️ Architecture

```text
Developer
    │
    │ Git Push
    ▼
GitHub Repository
    │
    ▼
Jenkins CI/CD
    │
    ├── Checkout
    │
    ├── Gitleaks
    │
    ├── Docker Build
    │
    ├── Trivy Scan
    │
    ├── Docker Hub Push
    │
    ├── Docker Compose Deploy
    │
    └── Health Checks
             │
             ▼
      Docker Compose Stack
             │
     ┌───────┼────────┐
     ▼       ▼        ▼
 Frontend  Backend   MongoDB
 React     Node.js   MongoDB 7
     │       │
     └───────┘
             │
             ▼
      Monitoring Stack
             │
        ┌────┴────┐
        ▼         ▼
     cAdvisor  Prometheus
                   │
                   ▼
                Grafana
```

---

## 🎯 Project Goals

This project demonstrates practical DevOps workflows rather than application development.

Key objectives:

- Automate CI/CD with Jenkins Pipeline as Code
- Build container images using Docker Compose
- Detect leaked secrets before image delivery
- Scan container images for vulnerabilities
- Publish versioned images to Docker Hub
- Deploy the application using Docker Compose
- Verify deployment health automatically
- Monitor container resource usage with Prometheus and Grafana
- Keep application secrets outside the Git repository

---

## 🧰 Technology Stack

### Application

- React / Vite — Frontend
- Node.js / Express — Backend API
- MongoDB 7 — Database

### CI/CD

- Jenkins
- Jenkinsfile
- Docker
- Docker Compose
- GitHub
- Docker Hub

### Security

- Gitleaks — Secret detection
- Trivy — Container vulnerability scanning

### Monitoring

- cAdvisor — Container metrics
- Prometheus — Metrics collection
- Grafana — Metrics visualization

---

## 🔄 CI/CD Pipeline

The Jenkins pipeline follows this workflow:

```text
Checkout
   ↓
Gitleaks Scan
   ↓
Prepare Runtime Secrets
   ↓
Build Docker Images
   ↓
Trivy Scan
   ↓
Docker Login
   ↓
Push Images
   ↓
Deploy with Docker Compose
   ↓
Health Checks
```

### 1. Checkout

Jenkins checks out the latest source code from GitHub.

### 2. Gitleaks

The source tree is scanned for accidentally committed secrets.

If a secret is detected, the pipeline stops before image delivery.

### 3. Runtime Secrets

Application JWT secrets are supplied through **Jenkins Credentials** and written to the runtime environment during the pipeline.

Secrets are not stored in the Git repository.

Required Jenkins credentials:

```text
dataflow-jwt-secret
dataflow-jwt-refresh-secret
```

Both should be configured as **Secret Text** credentials.

### 4. Build

Docker Compose builds:

```text
n00shy/dataflow-backend
n00shy/dataflow-frontend
```

Each build is tagged with the Jenkins build number in addition to `latest`.

Example:

```text
n00shy/dataflow-backend:15
n00shy/dataflow-frontend:15
```

### 5. Trivy

The generated images are scanned for **HIGH** and **CRITICAL** vulnerabilities.

The current pipeline reports these findings without blocking the build.

### 6. Docker Hub

After the security scan, Jenkins pushes both the build-number tag and `latest` tag to Docker Hub.

### 7. Deployment

Jenkins deploys the stack using:

```bash
docker compose down
docker compose up -d
```

### 8. Health Checks

The pipeline verifies:

- Backend API: `http://localhost:5000/api/health`
- Frontend: `http://localhost:3000`

A failed health check fails the Jenkins build.

---

## 🐳 Docker Compose Stack

The Compose stack contains:

| Service | Purpose | Host Port |
|---|---|---:|
| Frontend | React application served by Nginx | 3000 |
| Backend | Node.js / Express API | 5000 |
| MongoDB | Application database | Internal 27017 |
| Prometheus | Metrics collection | 9090 |
| Grafana | Monitoring dashboards | 3001 |
| cAdvisor | Container metrics | 8085 |

All services communicate through the Docker network:

```text
dataflow-net
```

MongoDB data is persisted using:

```text
mongo_data
```

Grafana data is persisted using:

```text
grafana_data
```

---

## 📊 Monitoring Architecture

The current monitoring layer focuses on **infrastructure/container metrics**.

```text
Docker Containers
       │
       ▼
   cAdvisor
       │
       ▼
   Prometheus
       │
       ▼
    Grafana
```

This provides visibility into metrics such as:

- Container CPU usage
- Container memory usage
- Network traffic
- Container activity/status

The project currently does **not** expose custom application/business metrics from the Node.js backend.

---

## 🔐 Security Practices

The project integrates security into the CI/CD workflow:

- Gitleaks secret detection
- Trivy container vulnerability scanning
- Jenkins Credentials for runtime secrets
- Secrets excluded from Git
- Versioned Docker images using Jenkins build numbers

> Any previously committed secret should be considered compromised and rotated.

---

## 📂 Project Structure

```text
.
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── src/
│
├── monitoring/
│   └── prometheus.yml
│
├── docker-compose.yml
├── Jenkinsfile
└── README.md
```

---

## 🚀 Run Locally

Clone the repository:

```bash
git clone https://github.com/n00shy/jenkins-pipline-with-docker-compose.git
cd jenkins-pipline-with-docker-compose
```

Configure the required backend environment variables locally, then start the stack:

```bash
docker compose up -d --build
```

Check running containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

---

## 🔍 Useful Endpoints

When running locally:

```text
Frontend    → http://localhost:3000
Backend     → http://localhost:5000
Prometheus  → http://localhost:9090
Grafana     → http://localhost:3001
cAdvisor    → http://localhost:8085
```

Backend health:

```text
GET /api/health
```

---

## 🔮 Future Improvements

Potential next phases:

- Immutable deployment using commit SHA image tags
- Deploy exact Jenkins build tags instead of relying on `latest`
- Automated Grafana dashboard provisioning
- Application-level Prometheus metrics
- Kubernetes deployment
- Terraform infrastructure automation
- Cloud deployment

---

## 👨‍💻 Author

**Abdullah Ahmed**  
Junior DevOps Engineer

- GitHub: https://github.com/n00shy
- LinkedIn: https://linkedin.com/in/n00shy
- Portfolio: https://n00shy.github.io/
