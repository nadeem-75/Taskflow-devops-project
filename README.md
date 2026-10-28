# TaskFlow – CI/CD Pipeline & Kubernetes Deployment

TaskFlow is a task management web application built with Node.js and Express. This project builds a complete DevOps workflow around it: a Jenkins pipeline checks code quality and security, builds a Docker image, pushes it to Docker Hub, and deploys the app. The same image can be deployed to a Kubernetes cluster (AWS EKS) with Argo CD.

---

## Application Features

- View, add, complete, and delete tasks
- REST API built with Express
- `/health` endpoint for Docker and Kubernetes health checks
- Tasks are stored in memory, so they reset when the app restarts

### API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/tasks` | Get all tasks |
| POST | `/api/tasks` | Add a new task (`{ "title": "..." }`) |
| PUT | `/api/tasks/:id` | Mark a task complete or incomplete |
| DELETE | `/api/tasks/:id` | Delete a task |
| GET | `/health` | Health check |

## Tech Stack

| Category | Tools |
|---|---|
| Application | Node.js, Express |
| Version Control | Git, GitHub |
| CI/CD | Jenkins |
| Code Quality | SonarQube |
| Security Scanning | OWASP Dependency-Check, Trivy, Docker Scout |
| Containerization | Docker, Docker Hub |
| Orchestration | Kubernetes (AWS EKS) |
| GitOps / CD | Argo CD |
| Monitoring | Prometheus, Grafana |

## CI/CD Workflow

```
GitHub  --->  Jenkins Pipeline
                 |-- SonarQube analysis + quality gate
                 |-- OWASP Dependency-Check
                 |-- Trivy file scan
                 |-- Docker build + push to Docker Hub
                 |-- Docker Scout image scan
                 |-- Deploy container
                 '-- Email notification with logs

Docker Hub image  --->  Argo CD  --->  AWS EKS  --->  Prometheus + Grafana
```

> Add your architecture diagram here: `![Architecture](images/architecture.png)`

## Jenkins Pipeline Stages

1. **Clean Workspace:** Removes files from the previous build.
2. **Git Checkout:** Pulls the `main` branch from GitHub.
3. **SonarQube Analysis:** Scans the code for bugs and code smells.
4. **Code Quality Gate:** Waits for the SonarQube quality gate result.
5. **Install NPM Dependencies:** Runs `npm install`.
6. **OWASP Dependency-Check:** Scans libraries for known vulnerabilities and publishes the report.
7. **Trivy File Scan:** Scans project files and saves the result to `trivy.txt`.
8. **Build Docker Image:** Builds the `taskflow` image.
9. **Tag and Push to Docker Hub:** Pushes `sknadeem786/taskflow:latest`.
10. **Docker Scout:** Checks the image for vulnerabilities and recommendations.
11. **Deploy to Container:** Runs the new image as a container on port 3000.
12. **Email Notification:** Sends the build status with the Trivy report attached.

## Run Locally

**With Node.js**
```bash
npm install
npm start
```

**With Docker**
```bash
docker build -t taskflow .
docker run -d -p 3000:3000 --name taskflow taskflow
```

Open `http://localhost:3000` in your browser.

## Jenkins Setup

Configure these in Jenkins before running the pipeline:

- **Tools:** `jdk17`, `node23`, `sonar-scanner`, and the OWASP `DP-Check` installation
- **Plugins:** SonarQube Scanner, NodeJS, OWASP Dependency-Check, Docker Pipeline, Email Extension
- **Credentials:** `docker` (Docker Hub) and `Sonar-token` (SonarQube)
- **Server:** `sonar-server` added under SonarQube servers
- **Installed on the Jenkins server:** Docker, Trivy, Docker Scout

## Kubernetes Deployment

The manifests are in the `k8s/` folder.

| File | Purpose |
|---|---|
| `k8s/deployment.yaml` | 2 replicas, resource limits, readiness and liveness probes on `/health` |
| `k8s/service.yaml` | LoadBalancer service on port 80, forwarding to container port 3000 |
| `argocd/application.yaml` | Argo CD application that syncs the `k8s/` folder from this repo |

### Deploy with kubectl
```bash
kubectl apply -f k8s/
kubectl get pods
kubectl get svc taskflow-service
```
Open the `EXTERNAL-IP` of the service in your browser.

### Deploy with Argo CD (GitOps)
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -f argocd/application.yaml
```
Argo CD watches the `k8s/` folder and keeps the cluster in sync with Git. When you change a manifest and push it, Argo CD updates the cluster automatically.

### Create the EKS cluster
```bash
eksctl create cluster --name taskflow-cluster --region <your-region> --nodes 2
kubectl get nodes
```

## Monitoring

Prometheus and Grafana can be installed on the cluster with Helm:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack
```

Then open Grafana and use the built-in Kubernetes dashboards to view pod and node metrics.

## Project Structure

```
.
├── public/                  # Frontend files
├── server.js                # Express server and API
├── package.json
├── Dockerfile
├── Jenkinsfile              # CI/CD pipeline
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
├── argocd/
│   └── application.yaml
├── images/                  # README screenshots
└── README.md
```

## Screenshots


### 1. TaskFlow application running
The app deployed by the pipeline and running on port 3000.

![TaskFlow App](images/01-taskflow-app.png)

### 2. Jenkins pipeline
Pipeline stages from clean workspace to deploying the container.

![Jenkins Pipeline](images/02-jenkins-pipeline.png)

### 3. SonarQube code quality report
The quality gate passed with 0 bugs, 0 vulnerabilities, and 0 code smells.

![SonarQube Report](images/03-sonarqube-report.png)

### 4. Docker Scout image scan
Docker Scout scanned the image and reported vulnerabilities in the `node:20-alpine` base image, with a recommendation to update the base image. See the improvements below.

![Docker Scout Scan](images/04-docker-scout-scan.png)

## What I Learned

- Building a full CI/CD pipeline with Jenkins
- Checking code quality and security with SonarQube, OWASP Dependency-Check, Trivy, and Docker Scout
- Containerizing a Node.js app and publishing images to Docker Hub
- Writing Kubernetes manifests with health probes and resource limits
- Using GitOps with Argo CD
- Monitoring a cluster with Prometheus and Grafana

## Future Improvements

- Update the base image from `node:20-alpine` to a newer version, as recommended by Docker Scout
- Run the container as a non-root user
- Use version tags (for example, the build number) instead of only `latest`
- Add automated tests to the pipeline
- Store tasks in a database so data is not lost on restart

## Author

**Shaik Nadeem**

- GitHub: [nadeem-75](https://github.com/nadeem-75)
- LinkedIn: [Shaik Nadeem](https://www.linkedin.com/in/shaik-nadeem-1954aa321)
- Email: nadeemshaik55036@gmail.com
