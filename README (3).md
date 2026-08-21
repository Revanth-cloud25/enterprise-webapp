# Enterprise Multi-Tier Web Application on AWS

> Production-style, highly available, secure, scalable and automated web application infrastructure provisioned with Terraform and deployed through a CI/CD pipeline.

## Project Overview

This project demonstrates the design and deployment of an enterprise-style multi-tier web application on AWS using Infrastructure as Code (IaC), automated deployment, centralized monitoring, secure networking, high availability, auto scaling, and remote Terraform state management with locking.

The infrastructure is designed across multiple Availability Zones and separates public-facing components from private application workloads.

> **Status:** Core infrastructure, CI/CD, and monitoring are fully implemented and deployed. A database tier (RDS) and DNS/HTTPS layer (Route 53 + ACM) are in active development — see [Planned Enhancements](#planned-enhancements) below.

![Architecture Diagram](enterprise-multi-tier-aws-architecture.png)

## Architecture

### High-Level Flow

```text
Users
  |
  v
Application Load Balancer (Public Subnets)
  |
  v
Auto Scaling Group
  |
  +-----------------------------+
  |                             |
  v                             v
EC2 App Server AZ-1         EC2 App Server AZ-2
(Private Subnet)            (Private Subnet)

Private EC2 workloads
        |
        v
   NAT Gateway
        |
        v
 Internet Gateway
        |
      Internet

CI/CD:
GitHub -> Jenkins -> Deployment -> EC2/ASG

Management:
AWS Systems Manager (SSM) -> EC2

Monitoring:
CloudWatch + CloudWatch Agent -> Metrics/Logs/Alarms

Terraform State:
Terraform -> S3 (Versioning)
             |
             +-> DynamoDB (State Locking)
```

See the accompanying `enterprise-multi-tier-aws-architecture.png` for the visual architecture diagram.

## Architecture Components

### 1. Networking

* **Amazon VPC** with an isolated CIDR range.
* **Two public subnets** distributed across multiple Availability Zones.
* **Private application subnets** across multiple Availability Zones.
* **Internet Gateway** for public subnet internet connectivity.
* **NAT Gateway** for controlled outbound internet access from private workloads.
* Separate **public and private route tables**.
* Route table associations managed through Terraform.
* Security Groups enforce tier-to-tier communication.

### 2. Application Tier

* **Application Load Balancer (ALB)** distributes incoming traffic across application instances.
* **Launch Template** defines the EC2 configuration.
* **Auto Scaling Group (ASG)** maintains the desired application capacity.
* Application servers run inside **private subnets** and are not directly exposed to the internet.
* ALB health checks remove unhealthy instances from service automatically.
* Instances can scale horizontally across Availability Zones.

### 3. CI/CD

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   +--> Build / Test
   |
   +--> Deployment
   |
   v
Application Environment
```

Jenkins automates the application deployment workflow using source code hosted in GitHub.

### 4. Infrastructure as Code

Terraform is used to provision and manage AWS infrastructure.

The Terraform configuration is modularized so networking, compute, security, load balancing, auto scaling and supporting resources can be maintained independently.

Terraform operations used throughout the project include:

```bash
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy
terraform state list
terraform state pull
terraform import
```

### 5. Terraform Remote State

Terraform state is stored remotely in **Amazon S3** rather than relying on a local state file.

The S3 backend provides:

* Centralized state storage
* State versioning
* Recovery from accidental state changes
* Persistent state outside the EC2 workstation

**DynamoDB** is used for state locking to prevent concurrent Terraform operations from corrupting the state file.

Example backend design:

```hcl
terraform {
  backend "s3" {
    bucket         = "enterprise-webapp-terraform-state"
    key            = "enterprise-webapp/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

> Never commit `terraform.tfstate`, credentials, secrets or generated provider binaries to Git.

### 6. IAM and Security

IAM is used to provide controlled AWS access to EC2 and automation components.

Implemented security controls include:

* IAM roles and instance profiles
* Least-privilege policies
* Security Groups
* Private application subnets
* Restricted administrative access
* SSM-based instance management
* No direct public exposure of application instances
* Secrets kept outside source control

### 7. Systems Manager

**AWS Systems Manager (SSM)** provides secure management access to EC2 instances.

This reduces the dependency on direct SSH access and allows operational tasks to be performed through Session Manager.

### 8. Monitoring and Logging

**Amazon CloudWatch** is used for infrastructure observability.

The CloudWatch Agent collects application/server logs and publishes them to CloudWatch Logs.

Monitoring includes:

* EC2 CPU utilization
* Application health
* ALB health checks
* ALB request metrics
* Application logs
* System logs
* Auto Scaling activity
* CloudWatch alarms

Example application log observed during deployment:

```text
"GET / HTTP/1.1" 200
"ELB-HealthChecker/2.0"
```

This confirms that the ALB health checker successfully reached the application endpoint.

### 9. Cost Optimization

Cost-control practices include:

* Auto Scaling based on workload requirements
* Right-sized EC2 instances
* Controlled NAT Gateway usage
* Private/public subnet separation
* S3 lifecycle management where appropriate
* CloudWatch monitoring for unused resources
* AWS Budgets and cost alerts
* Removing temporary development resources after testing

## Planned Enhancements

The following components are designed and in active development, targeted for completion shortly:

* **Amazon RDS** — private database subnets, Multi-AZ deployment, automated backups, and point-in-time recovery, with database access restricted to the application security group only.
* **Amazon Route 53** — DNS resolution for the application domain.
* **AWS Certificate Manager (ACM) + HTTPS** — SSL/TLS termination at the ALB, with automatic HTTP → HTTPS redirection.

## Repository Structure

```text
enterprise-webapp/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
├── versions.tf
├── backend.tf
├── terraform.tfvars
├── README.md
│
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── iam.tf
│   │
│   ├── security/
│   ├── alb/
│   ├── asg/
│   └── monitoring/
│
├── userdata/
│   └── app-init.sh
│
└── scripts/
    ├── deploy.sh
    └── monitoring.sh
```

## Example Application Bootstrap

Application instances install Apache and publish a simple application page during initialization.

```bash
#!/bin/bash

dnf update -y
dnf install -y httpd

systemctl enable httpd
systemctl start httpd

TOKEN=$(curl -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

INSTANCE_ID=$(curl \
  -H "X-aws-ec2-metadata-token:$TOKEN" \
  "http://169.254.169.254/latest/meta-data/instance-id")

echo "Enterprise Web Application Instance: $INSTANCE_ID" \
  > /var/www/html/index.html
```

The instance ID is intentionally displayed to make load balancing and Auto Scaling behavior easy to demonstrate.

## Deployment Process

### Prerequisites

* AWS account
* AWS CLI
* Terraform
* Git
* Jenkins
* Appropriate IAM permissions
* GitHub repository
* SSH key pair only where required for bootstrap/recovery

### Initialize Terraform

```bash
terraform init
```

### Validate Configuration

```bash
terraform validate
```

### Review Infrastructure Changes

```bash
terraform plan
```

### Deploy

```bash
terraform apply
```

### Verify State

```bash
terraform state list
terraform state pull
```

### Verify Application

1. Access the application through the ALB DNS name.
2. Confirm the ALB target group reports healthy instances.
3. Verify application responses from multiple instances.
4. Check CloudWatch logs and metrics.
5. Verify Auto Scaling behavior.
6. Verify Jenkins deployment status.

## Validation Checklist

* [x] VPC created successfully
* [x] Public/private subnet separation
* [x] Multiple Availability Zones
* [x] Internet Gateway configured
* [x] NAT Gateway configured
* [x] Route tables and associations configured
* [x] Security Groups configured
* [x] Application Load Balancer deployed
* [x] Launch Template configured
* [x] Auto Scaling Group deployed
* [x] Application health checks passing
* [x] EC2 application instances deployed
* [x] Jenkins CI/CD pipeline configured
* [x] GitHub source integration configured
* [x] IAM roles and instance profiles configured
* [x] AWS Systems Manager configured
* [x] CloudWatch Agent installed and running
* [x] Application logs available in CloudWatch
* [x] Terraform remote state migrated to S3
* [x] S3 versioning enabled
* [x] DynamoDB state locking configured
* [x] Cost monitoring configured
* [ ] RDS database deployed in private subnets *(in progress)*
* [ ] Route 53 DNS configured *(in progress)*
* [ ] ACM certificate + HTTPS enabled *(in progress)*

## Key Engineering Outcomes

### High Availability

Application instances run across multiple Availability Zones behind an ALB, reducing the impact of an individual instance or Availability Zone failure.

### Scalability

The Auto Scaling Group automatically adjusts application capacity according to workload requirements.

### Security

Application servers remain in private subnets while the ALB provides the controlled public entry point.

### Observability

CloudWatch metrics, alarms and centralized logs provide visibility into infrastructure and application health.

### Automation

Terraform automates infrastructure provisioning while Jenkins automates application deployment.

### State Management

Remote Terraform state in S3 with versioning, combined with DynamoDB state locking, provides reliable and conflict-free state management for collaborative infrastructure operations.
