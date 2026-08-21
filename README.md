# Jerney Blog — End-to-End DevSecOps Deployment on AWS EKS

An end-to-end DevSecOps project for deploying a 3-tier web application on Amazon EKS using Terraform, Docker, Kubernetes, Jenkins, and automated security scanning.

The project demonstrates Infrastructure as Code, containerization, CI/CD automation, Kubernetes orchestration, application autoscaling, rolling updates, and security checks across the development and deployment lifecycle.

---

## 🚀 Project Overview

Jerney is a 3-tier blog application consisting of:

- React frontend
- Node.js backend
- PostgreSQL database

The application is containerized using Docker and deployed on Amazon EKS using Kubernetes.

AWS infrastructure is provisioned using Terraform, while Jenkins automates the CI/CD and DevSecOps workflow.

### High-Level Architecture

```text
                         GitHub
                           │
                           ▼
                        Jenkins
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
          ▼                ▼                 ▼
       Gitleaks         Checkov           Trivy
       Secret Scan      IaC Scan       Vulnerability Scan
          │                │                 │
          └────────────────┼─────────────────┘
                           ▼
                       SonarQube
                           │
                     Quality Gate
                           │
                           ▼
                    Docker Build
                           │
                     Trivy Scan
                           │
                           ▼
                      Docker Hub
                           │
                           ▼
                     Amazon EKS
                    Auto Mode
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
        React Frontend            Node.js Backend
              │                         │
              └────────────┬────────────┘
                           ▼
                      PostgreSQL

🏗️ Infrastructure Architecture

The AWS infrastructure is provisioned using Terraform.

AWS Region
Region: ap-south-1
Location: Mumbai
Infrastructure Components
Amazon VPC
Public subnets
Private subnets
Internet Gateway
NAT Gateway
Amazon EKS
EKS Auto Mode
EKS NodePools
IAM
Security Groups
Amazon EBS integration
Application Load Balancer integration

The VPC is configured across 3 Availability Zones for improved availability.

                         AWS Mumbai
                        ap-south-1
                             │
                ┌────────────┴────────────┐
                │                         │
             AZ-1                      AZ-2
                │                         │
        ┌───────┴───────┐        ┌───────┴───────┐
        │               │        │               │
     Public          Private   Public          Private
     Subnet          Subnet    Subnet          Subnet
        │               │        │               │
        └───────────────┴────────┴───────────────┘
                             │
                         EKS Auto Mode
                             │
                       Jerney Application

A third Availability Zone is also provisioned by the Terraform configuration.

🧱 Infrastructure as Code

Terraform is used to provision and manage the AWS infrastructure.

Terraform provisions
VPC
Public and private subnets
Route tables
Internet Gateway
NAT Gateway
EKS cluster
EKS Auto Mode configuration
EKS NodePools
Required networking configuration
IAM configuration required by EKS
Terraform Structure
terraform/
├── main.tf
├── variables.tf
├── provider.tf
├── outputs.tf
└── terraform.tfvars
Terraform Workflow
Terraform Configuration
          │
          ▼
     terraform init
          │
          ▼
   terraform validate
          │
          ▼
      terraform plan
          │
          ▼
     terraform apply
          │
          ▼
 AWS Infrastructure
          │
          ▼
      Amazon EKS
☁️ Amazon EKS Auto Mode

The project uses Amazon EKS Auto Mode.

EKS manages the Kubernetes control plane, while Auto Mode manages the underlying compute required for Kubernetes workloads.

No manually created EC2 master/control-plane instance is required.

Auto Mode Configuration
EKS Cluster
    │
    ├── Control Plane
    │      └── AWS Managed
    │
    └── Auto Mode
           │
           ├── general-purpose NodePool
           └── system NodePool

This allows the project to demonstrate Kubernetes workload management without manually maintaining traditional worker-node groups.

🐳 Containerization

The frontend and backend are containerized using Docker.

Frontend
React Application
       │
       ▼
   Dockerfile
       │
       ▼
Nginx Container
       │
       ▼
Port 8080
Backend
Node.js Application
       │
       ▼
   Dockerfile
       │
       ▼
Node.js Container
       │
       ▼
Port 5000

The frontend Nginx configuration proxies API requests to the backend service.

Browser
   │
   ▼
Frontend
   │
   │ /api/*
   ▼
Backend
   │
   ▼
PostgreSQL
🐘 Database

PostgreSQL is used as the application's database.

For local development, PostgreSQL is configured through Docker Compose.

Frontend
    │
    ▼
Backend
    │
    ▼
PostgreSQL

The backend communicates with PostgreSQL using environment-based database configuration.

🧪 Local Development

Docker Compose is provided for running the application locally.

Start the application
docker compose up --build
Check running containers
docker compose ps
Stop the application
docker compose down

The local environment allows the frontend, backend, and PostgreSQL services to be tested before deploying to Kubernetes.

☸️ Kubernetes Deployment

The application is deployed to Amazon EKS using Kubernetes manifests.

k8s/
└── manifest.yaml

The Kubernetes configuration manages:

Namespace
Frontend Deployment
Backend Deployment
Services
Horizontal Pod Autoscaler
Health probes
NetworkPolicies
Persistent storage
Application configuration
📈 Kubernetes Autoscaling

Horizontal Pod Autoscaler (HPA) is configured for the application workloads.

Backend
Minimum replicas: 2
Maximum replicas: 5
Frontend
Minimum replicas: 2
Maximum replicas: 4

The HPA automatically adjusts the number of application pods based on resource utilization.

              Application Load
                     │
                     ▼
                    HPA
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      Scale Out              Scale In
      More Pods              Fewer Pods
🔄 Rolling Updates

Kubernetes rolling updates are used to deploy new application versions without stopping the entire application.

Old Version
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
          │
          ▼
     New Version
          │
   ├── New Pod
   ├── New Pod
   └── New Pod

The Jenkins pipeline verifies the rollout using:

kubectl rollout status

This ensures that the deployment successfully reaches the desired state.

❤️ Health Checks

Kubernetes health probes are used to monitor application health.

The deployment uses Kubernetes health-check mechanisms to determine whether application containers are ready to receive traffic and whether unhealthy containers need to be restarted.

🔐 DevSecOps

Security checks are integrated throughout the CI/CD pipeline.

Security Tools
Tool	Purpose
Gitleaks	Secret detection
Trivy	Filesystem and container vulnerability scanning
Checkov	Terraform/IaC security scanning
SonarQube	Code quality and security analysis
Kubeaudit	Kubernetes security auditing
🔍 Checkov — Infrastructure Security

Checkov scans the Terraform infrastructure configuration for security and compliance issues.

The configuration is stored at the repository root:

.checkov.yml

Terraform is scanned from:

terraform/

Checkov is configured with:

soft-fail: false

This ensures that a failing Checkov security scan can stop the CI/CD pipeline.

🔄 CI/CD Pipeline

Jenkins automates the complete build, security scanning, containerization, and Kubernetes deployment workflow.

Pipeline Flow
                    GitHub
                       │
                       ▼
                 Git Checkout
                       │
                       ▼
              Gitleaks Secret Scan
                       │
                       ▼
             Frontend npm Install
                       │
                       ▼
                Frontend Tests
                       │
                       ▼
              Backend npm Install
                       │
                       ▼
                 Backend Tests
                       │
                       ▼
             Trivy Filesystem Scan
                       │
                       ▼
                Checkov IaC Scan
                       │
                       ▼
              SonarQube Analysis
                       │
                       ▼
              SonarQube Quality Gate
                       │
                       ▼
             Build Frontend Image
                       │
                       ▼
             Build Backend Image
                       │
                       ▼
            Trivy Image Scan
                       │
                       ▼
             Push to Docker Hub
                       │
                       ▼
               Kubeaudit Scan
                       │
                       ▼
          Update Kubernetes Images
                       │
                       ▼
        Kubernetes Manifest Validation
                       │
                       ▼
                Deploy to EKS
                       │
                       ▼
          Backend Rollout Verification
                       │
                       ▼
         Frontend Rollout Verification
                       │
                       ▼
            Kubernetes Health Checks
                       │
                       ▼
                 Verify HPA
                       │
                       ▼
              Email Notification
🔒 Security Pipeline

Security checks are performed at multiple layers:

Source Code
    │
    └── Gitleaks
          │
          ▼
Filesystem
    │
    └── Trivy
          │
          ▼
Terraform Infrastructure
    │
    └── Checkov
          │
          ▼
Application Code
    │
    └── SonarQube
          │
          ▼
Docker Images
    │
    └── Trivy
          │
          ▼
Kubernetes Configuration
    │
    └── Kubeaudit
          │
          ▼
Deployment to EKS
📦 Docker Images

The application uses separate Docker images for the frontend and backend.

Frontend Image
sagarsmanjunath/jerney-frontend
Backend Image
sagarsmanjunath/jerney-backend

Jenkins generates a build-specific image tag using the Jenkins build number.

Example:

sagarsmanjunath/jerney-frontend:25
sagarsmanjunath/jerney-backend:25

This allows Kubernetes deployments to use versioned images instead of relying only on the latest tag.

📧 Jenkins Notifications

The Jenkins pipeline sends an email notification after the pipeline completes.

The notification includes:

Jenkins job name
Build number
Pipeline status
Git branch
Frontend Docker image
Backend Docker image
Jenkins console output link
Security scan reports

Reports generated during the pipeline include:

gitleaks-report.sarif
trivy-fs-report.txt
checkov-report.txt
trivy-frontend-image-report.txt
trivy-backend-image-report.txt
kubeaudit-report.txt
📁 Project Structure
Jerney/
│
├── backend/
│   ├── src/
│   ├── Dockerfile
│   ├── package.json
│   └── .dockerignore
│
├── frontend/
│   ├── src/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── package.json
│   └── .dockerignore
│
├── k8s/
│   └── manifest.yaml
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── provider.tf
│   ├── outputs.tf
│   └── terraform.tfvars
│
├── .checkov.yml
├── Jenkinsfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
├── .env
└── .env.example
🛠️ Technologies Used
Application
React
Node.js
PostgreSQL
Cloud
AWS
Amazon VPC
Amazon EKS
EKS Auto Mode
Amazon EBS
Application Load Balancer
IAM
Infrastructure as Code
Terraform
Containers
Docker
Docker Hub
Kubernetes
Kubernetes
Deployments
Services
Horizontal Pod Autoscaler
Health Probes
NetworkPolicies
Persistent Volumes
CI/CD
Jenkins
Git
GitHub
DevSecOps
Gitleaks
Trivy
Checkov
SonarQube
Kubeaudit
🚀 Deployment Workflow
1. Clone the repository
git clone https://github.com/sagar-smanjunath/Jerney.git
cd Jerney
git checkout devops
2. Provision AWS Infrastructure

Navigate to the Terraform directory:

cd terraform

Initialize Terraform:

terraform init

Validate the configuration:

terraform validate

Review the infrastructure plan:

terraform plan

Apply the infrastructure:

terraform apply

Confirm the deployment when prompted.

3. Configure kubectl

After the EKS cluster is created:

aws eks update-kubeconfig \
  --region ap-south-1 \
  --name jerney-eks

Verify the cluster:

kubectl get nodes

Check EKS Auto Mode NodePools:

kubectl get nodepools
4. Deploy the Application

The preferred deployment method is through Jenkins.

The Jenkins pipeline:

Checks out the devops branch
Runs application tests
Performs security scans
Builds Docker images
Pushes images to Docker Hub
Validates Kubernetes manifests
Deploys the application to EKS
Verifies rollouts
Checks application health
Verifies HPA configuration
Sends an email notification
🔎 Verify the Deployment

Check the namespace:

kubectl get namespace

Check pods:

kubectl get pods -n jerney

Check services:

kubectl get svc -n jerney

Check deployments:

kubectl get deployments -n jerney

Check HPA:

kubectl get hpa -n jerney

Check persistent volume claims:

kubectl get pvc -n jerney

Check ingress:

kubectl get ingress -n jerney

Check rollout:

kubectl rollout status deployment/jerney-backend -n jerney
kubectl rollout status deployment/jerney-frontend -n jerney
🧹 Cleanup

When the project is no longer required, destroy the AWS infrastructure created by Terraform.

From the Terraform directory:

terraform destroy

Review the resources carefully and confirm the destruction.

WARNING:
terraform destroy removes the infrastructure managed by this
Terraform configuration. Use it only when the environment is
no longer required.
🎯 Key DevOps Concepts Demonstrated

This project demonstrates practical implementation of:

Infrastructure as Code using Terraform
AWS VPC networking
Multi-AZ cloud infrastructure
Amazon EKS
EKS Auto Mode
Kubernetes container orchestration
Docker containerization
Jenkins CI/CD
Git-based workflows
Automated testing
Secret scanning
Infrastructure security scanning
Container vulnerability scanning
Static code analysis
Kubernetes security auditing
Docker image versioning
Kubernetes rolling updates
Horizontal Pod Autoscaling
Kubernetes health checks
NetworkPolicies
Persistent storage
Automated deployment verification
CI/CD email notifications
👨‍💻 Author

Sagar S M

AWS DevOps Engineer | Cloud & DevOps Enthusiast

GitHub:
https://github.com/sagar-smanjunath

LinkedIn:
https://www.linkedin.com/in/sagar-sm

