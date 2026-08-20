# Jerney - End-to-End DevSecOps 3-Tier Blog Application

An end-to-end DevSecOps implementation of a 3-tier blog application using React, Node.js, PostgreSQL, Docker, Jenkins, SonarQube, Trivy, Gitleaks, Kubeaudit, Docker Hub, and Amazon EKS.

The project demonstrates how a containerized application can be securely built, scanned, continuously integrated, and deployed to Kubernetes using a Jenkins-based CI/CD pipeline.

---

## Application Architecture

The application consists of three tiers:

- **Frontend:** React + Nginx
- **Backend:** Node.js
- **Database:** PostgreSQL

### Application Flow

```text
                    Internet
                       |
                       v
                  AWS ALB
                       |
                       v
              Frontend Service
                       |
                       v
               React + Nginx
                  Port 8080
                       |
                 /api/ requests
                       |
                       v
              Backend Service
                       |
                       v
                Node.js API
                  Port 5000
                       |
                       v
                 PostgreSQL
                  Port 5432
                       |
                       v
                 Kubernetes PVC
                       |
                       v
                     EBS
DevSecOps Architecture
                         GitHub
                           |
                       devops branch
                           |
                           v
                        Jenkins
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
       Gitleaks          Trivy          SonarQube
     Secret Scan      Security Scan    Code Analysis
          |                |                |
          +----------------+----------------+
                           |
                           v
                    npm Install & Test
                           |
                           v
                      Docker Build
                     /            \
                    /              \
                   v                v
              Frontend           Backend
                Image              Image
                   \                /
                    \              /
                     v            v
                    Trivy Image Scan
                           |
                           v
                       Docker Hub
                           |
                           v
                       Kubeaudit
                           |
                           v
                 Kubernetes Validation
                           |
                           v
                    Amazon EKS
                           |
            +--------------+--------------+
            |                             |
            v                             v
       Frontend Pods                Backend Pods
       React + Nginx                  Node.js
            |                             |
            +--------------+--------------+
                           |
                           v
                      PostgreSQL
                           |
                           v
                         PVC
                           |
                           v
                         EBS
Technologies Used
Application
React
Node.js
PostgreSQL
Nginx
Containerization
Docker
Docker Compose
CI/CD
Jenkins
GitHub
DevSecOps / Security
Gitleaks
Trivy
SonarQube
Kubeaudit
Container Registry
Docker Hub
Cloud & Kubernetes
AWS
Amazon EKS
Kubernetes
AWS Application Load Balancer
EBS
Kubernetes PersistentVolumeClaim
Monitoring
Prometheus
Node Exporter
Blackbox Exporter
Grafana
Project Structure
Jerney/
│
├── backend/
│   ├── src/
│   ├── Dockerfile
│   └── .dockerignore
│
├── frontend/
│   ├── src/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── .dockerignore
│
├── k8s/
│   └── manifest.yaml
│
├── docker-compose.yml
├── Jenkinsfile
├── .gitignore
├── .env
├── .env.example
└── README.md
Local Development

Docker Compose is used to run the complete application locally.

The Compose setup contains:

PostgreSQL
     |
     v
Backend
     |
     v
Frontend + Nginx
Start the Application

From the project root:

docker compose up --build

The frontend is available at:

http://localhost

The backend runs internally on:

5000

PostgreSQL runs on:

5432
Stop the Application
docker compose down

To remove the PostgreSQL volume as well:

docker compose down -v
Docker Configuration

The application uses multi-stage Docker builds.

Backend

The backend Docker image:

Uses Node.js Alpine
Uses a multi-stage build
Installs production dependencies
Runs as a non-root user
Uses dumb-init
Exposes port 5000
Node.js
   |
   v
Multi-stage Docker Build
   |
   v
Production Image
   |
   v
Node.js :5000
Frontend

The frontend Docker image:

Builds the React application
Uses Nginx Alpine for production
Removes the default Nginx configuration
Uses a custom Nginx configuration
Runs Nginx as a non-root user
Exposes port 8080
React Source
     |
     v
npm Build
     |
     v
React dist/
     |
     v
Nginx
     |
     v
Port 8080
Nginx Configuration

The frontend container uses:

frontend/nginx.conf

Nginx serves the React frontend and proxies API requests to the backend.

/api/*
    |
    v
jerney-backend:5000


/*
    |
    v
React Static Files

The backend service is accessed internally through:

jerney-backend:5000
Kubernetes Deployment

The Kubernetes resources are maintained in:

k8s/manifest.yaml

The manifest contains:

Namespace
Kubernetes Secret
StorageClass
PersistentVolumeClaim
PostgreSQL Deployment
PostgreSQL Service
Backend Deployment
Backend Service
Frontend Deployment
Frontend Service
EKS Auto Mode Ingress
Horizontal Pod Autoscaler
PodDisruptionBudget
NetworkPolicies
Kubernetes Architecture
                    AWS ALB
                       |
                       v
              Frontend Service
                    :80
                       |
                       v
                Frontend Pods
                 Nginx :8080
                       |
                       |
              /api/ requests
                       |
                       v
              Backend Service
                    :5000
                       |
                       v
                Backend Pods
                 Node.js
                       |
                       v
                PostgreSQL
                    :5432
                       |
                       v
                     PVC
                       |
                       v
                    EBS
Kubernetes Networking

The frontend is exposed through an AWS Application Load Balancer.

The backend is not exposed directly to the internet.

The PostgreSQL database is also not exposed externally.

Internet
   |
   v
ALB
   |
   v
Frontend
   |
   v
Backend
   |
   v
PostgreSQL
Kubernetes Autoscaling

Horizontal Pod Autoscaling is configured for the frontend and backend.

Backend
Minimum replicas: 2
Maximum replicas: 5

The backend HPA uses CPU and memory utilization.

Frontend
Minimum replicas: 2
Maximum replicas: 4

The frontend HPA also uses CPU and memory utilization.

Rolling Updates

Frontend and backend deployments use Kubernetes RollingUpdate strategy.

The deployment configuration uses:

maxUnavailable: 0
maxSurge: 1

This allows new pods to be started before old pods are removed and helps maintain application availability during deployments.

Persistent Storage

PostgreSQL uses a Kubernetes PersistentVolumeClaim.

PostgreSQL
     |
     v
PVC
     |
     v
EBS gp3

The EBS storage configuration uses:

gp3
Encryption
ReadWriteOnce
Dynamic provisioning
Kubernetes Security

The Kubernetes deployment includes several security controls.

Container Security
Non-root containers
allowPrivilegeEscalation: false
Linux capabilities dropped
Resource requests and limits
Read-only filesystem where applicable
Service account token disabled where unnecessary
Network Security

NetworkPolicies restrict communication between application tiers.

Frontend
   |
   v
Backend
   |
   v
PostgreSQL

PostgreSQL only accepts traffic from backend pods.

Backend traffic is restricted to frontend pods.

CI/CD Pipeline

Jenkins is used as the CI/CD engine.

The pipeline is configured to use the:

devops

branch.

The Jenkins pipeline performs the following stages:

Git Checkout
      |
      v
Gitleaks
      |
      v
Frontend npm Install
      |
      v
Frontend Tests
      |
      v
Backend npm Install
      |
      v
Backend Tests
      |
      v
Trivy Filesystem Scan
      |
      v
SonarQube Analysis
      |
      v
SonarQube Quality Gate
      |
      v
Docker Build
      |
      +----------------+
      |                |
      v                v
   Frontend         Backend
      |                |
      +--------+-------+
               |
               v
        Trivy Image Scan
               |
               v
          Docker Hub
               |
               v
           Kubeaudit
               |
               v
     Kubernetes Validation
               |
               v
             EKS
               |
               v
       Rolling Deployment
               |
               v
       Rollout Verification
               |
               v
       HPA / Pod / Service
           Verification
               |
               v
        Email Notification
Security Scanning
Gitleaks

Gitleaks is used to detect accidentally committed secrets such as:

Passwords
API keys
Tokens
Credentials

Pipeline stage:

Gitleaks Secret Scan
Trivy Filesystem Scan

Trivy scans the source repository for:

Vulnerabilities
Secrets
Misconfigurations

Pipeline stage:

Trivy Filesystem Scan
SonarQube

SonarQube performs static code analysis.

It helps identify:

Bugs
Code smells
Security issues
Maintainability problems

The pipeline waits for the SonarQube Quality Gate before continuing.

SonarQube Analysis
        |
        v
   Quality Gate
        |
   +----+----+
   |         |
 PASS       FAIL
   |         |
   v         v
Continue    Stop
Trivy Image Scanning

After Docker images are built, Trivy scans the images for high and critical vulnerabilities.

Images scanned:

sagarsmanjunath/jerney-frontend
sagarsmanjunath/jerney-backend
Kubeaudit

Kubeaudit scans the Kubernetes manifest for Kubernetes security issues before deployment.

The Kubernetes manifest being scanned is:

k8s/manifest.yaml
Docker Hub

Docker images are pushed to Docker Hub.

Frontend
sagarsmanjunath/jerney-frontend
Backend
sagarsmanjunath/jerney-backend

Jenkins creates both a build-specific tag and the latest tag.

Example:

sagarsmanjunath/jerney-frontend:25
sagarsmanjunath/jerney-backend:25

Build-specific tags make it possible to identify which Jenkins build produced a deployed image.

Deployment to EKS

After the required security checks pass, Jenkins deploys the application to Amazon EKS.

The deployment process is:

Docker Hub
     |
     v
EKS
     |
     v
kubectl apply
     |
     v
Kubernetes Deployments
     |
     v
Rolling Update
     |
     v
Rollout Verification

Jenkins verifies:

Backend rollout
Frontend rollout
Pods
Services
Deployments
HPA
PVC
Ingress
Monitoring

The project uses two levels of monitoring.

System-Level Monitoring

Node Exporter is used to collect system-level metrics such as:

CPU usage
Memory usage
Disk usage
System statistics

Node Exporter exposes metrics that can be collected by Prometheus.

System
  |
  v
Node Exporter
  |
  v
Prometheus
  |
  v
Grafana
Website-Level Monitoring

Blackbox Exporter is used to monitor application availability from the outside.

It can monitor:

HTTP availability
Endpoint response
Application reachability
Website
   |
   v
Blackbox Exporter
   |
   v
Prometheus
   |
   v
Grafana
Monitoring Architecture
                    Prometheus
                    /        \
                   /          \
                  v            v
          Node Exporter    Blackbox Exporter
               |                 |
               v                 v
        System Metrics      Website Metrics
               \                 /
                \               /
                 v             v
                    Grafana
Environment Variables

Environment-specific configuration is managed using environment variables.

The actual .env file contains local configuration and secrets and should not be shared publicly.

An .env.example file can be used to document the required variable names without exposing real credentials.

Example:

POSTGRES_USER=your-user
POSTGRES_PASSWORD=your-password
POSTGRES_DB=your-database

Actual credentials should be provided through a secure mechanism.

Important Security Practices

This project follows several DevSecOps practices:

Secrets scanning with Gitleaks
Filesystem scanning with Trivy
Docker image vulnerability scanning with Trivy
Static code analysis using SonarQube
Kubernetes security auditing with Kubeaudit
Non-root containers
Resource limits
Kubernetes NetworkPolicies
Encrypted EBS storage
Kubernetes health probes
Rolling updates
Horizontal Pod Autoscaling
Internal-only backend and database services
Running the Application Locally

Clone the repository:

git clone -b devops https://github.com/sagar-smanjunath/Jerney.git

Move into the project:

cd Jerney

Start the application:

docker compose up --build

Open:

http://localhost

Stop the application:

docker compose down
Deploying to Kubernetes Manually

Make sure your kubectl context points to the EKS cluster.

Apply the Kubernetes manifest:

kubectl apply -f k8s/manifest.yaml

Check pods:

kubectl get pods -n jerney

Check services:

kubectl get svc -n jerney

Check deployments:

kubectl get deployments -n jerney

Check HPA:

kubectl get hpa -n jerney

Check PVC:

kubectl get pvc -n jerney

Check ingress:

kubectl get ingress -n jerney

Check rollout:

kubectl rollout status deployment/jerney-frontend -n jerney
kubectl rollout status deployment/jerney-backend -n jerney
Project Highlights

This project demonstrates practical implementation of:

3-tier application architecture
Docker containerization
Multi-stage Docker builds
Docker Compose
Jenkins CI/CD
DevSecOps security scanning
GitHub integration
Docker Hub image management
Kubernetes deployments
Amazon EKS
EKS Auto Mode
AWS Application Load Balancer
Kubernetes Services
PersistentVolumeClaims
EBS storage
Rolling updates
Horizontal Pod Autoscaling
Kubernetes health probes
Kubernetes NetworkPolicies
System monitoring
Website monitoring
CI/CD Tools Summary
Tool	Purpose
GitHub	Source code management
Jenkins	CI/CD automation
Gitleaks	Secret detection
npm	Dependency management and testing
SonarQube	Code quality and security analysis
Trivy	Filesystem and container image scanning
Docker	Containerization
Docker Hub	Container image registry
Kubeaudit	Kubernetes security auditing
Kubernetes	Container orchestration
Amazon EKS	Managed Kubernetes
Prometheus	Metrics collection
Node Exporter	System metrics
Blackbox Exporter	Website availability monitoring
Grafana	Metrics visualization
Project Status

The project is implemented as a Jenkins-based DevSecOps pipeline.

Current deployment architecture:

GitHub
  |
  v
Jenkins
  |
  v
Security Scanning
  |
  v
Docker Build
  |
  v
Docker Hub
  |
  v
Kubeaudit
  |
  v
Amazon EKS
  |
  v
Rolling Updates + HPA
  |
  v
Monitoring
Author

Sagar S M

Cloud & DevOps Enthusiast

GitHub:
https://github.com/sagar-smanjunath


