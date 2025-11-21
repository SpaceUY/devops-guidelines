---
title: Introduction
layout: default
nav_order: 1
---

# Introduction to DevOps Guidelines

Welcome to the DevOps Guidelines.  

This document defines how we deploy, manage, and standardize our infrastructure across all projects. The goal is to maintain **consistency, scalability, security, and operational clarity** in every development cycle.

These guidelines formalize our preferred approach:  

**AWS Elastic Beanstalk for backend deployments** (Nest.js, Node.js, and any compute workloads)  

+  

**AWS Amplify for frontend deployments** (React, Vite, Next.js).

This combination gives us a modern, efficient, auditable, and secure foundation that scales with project requirements.

---

# 1. Why We Standardize on Elastic Beanstalk (Backends)

AWS Elastic Beanstalk provides a fully managed layer on top of EC2 that automatically handles:

- Application deployments  

- Application versioning  

- Capacity provisioning  

- Load balancing (via ALB)  

- Auto Scaling Groups  

- Health checks  

- Rollbacks  

- Logging and monitoring integration  

It gives us the best midpoint between:

- **The simplicity of PaaS**

- **The power and flexibility of full AWS control**

For our backend services—primarily using **Nest.js**—Beanstalk allows us to deploy quickly without sacrificing the ability to inspect, modify, or tune the underlying infrastructure when needed.

## 1.1 Why This Is Our Preferred Option

**1. Full control + simple management**  

Unlike alternative PaaS platforms, Beanstalk does not hide the underlying components. We maintain full visibility over:

- EC2 instances  

- Load balancers  

- Security Groups  

- VPC  

- IAM roles  

- Auto Scaling  

- Instance profiles  

- Health checks  

- Logs in CloudWatch  

This aligns perfectly with our needs for SOC2, compliance, cost visibility, and internal auditing.

**2. Standard AWS architecture out of the box**  

Beanstalk deploys using the same primitives we already use everywhere else:

Route 53 + ACM (SSL)

↓

Application Load Balancer (HTTPS + health checks)

↓

Auto Scaling Group

↓

EC2 instances (private subnets, SG locked down)

↓

RDS (private subnet)

↓

CloudWatch logs + metrics

This means no surprises, no lock-in beyond AWS fundamentals, and predictable behavior across projects.

**3. Built-in versioning system**  

Beanstalk manages "Application Versions" stored in S3.  

This allows us to:

- Track every deployment  

- Rollback fast  

- Analyze deploy history  

- Replicate environments easily  

**4. Easy to evolve**  

If a project grows into microservices, ECS/Fargate, or EKS, the same VPC and network layout still works. Nothing is wasted.

---

# 2. Our Standard: Dockerized Backends (When Needed)

Although Beanstalk supports native Node.js deployments, we **standardize Docker for projects that require more than one backend per deployment** or when we need strict runtime reproducibility.

## 2.1 When we use Docker inside Beanstalk

- Multi-container applications  

- Multiple APIs sharing 1 deployment  

- Need for OS-level dependencies (FFmpeg, Python scripts, libs, etc.)  

- Consistency between local dev / CI / production  

- Preparing future migration to ECS/EKS  

## 2.2 How Docker works in Beanstalk

We deploy via:

- **Single-container Docker** (simple cases)

- **Multi-container (docker-compose) via ECS integrated mode** (complex cases)

Beanstalk will:

- Pull the Docker image from **Amazon ECR**

- Start the container(s)

- Expose the defined ports to the ALB

- Monitor container health

- Auto-restart or replace unhealthy containers

## 2.3 Limitations to consider

- Application Version bundles still require a small ZIP in S3 (even when using ECR)

- EC2 instance disk size must be enough to store images (default 8–20GB can be increased)

- Deployments of large images may take longer

- Logs inside containers are streamed to CloudWatch but require correct config

---

# 3. Elastic Beanstalk Environment Types (dev / staging / prod)

We support and standardize **multiple isolated environments**:

### **Dev Environment**

- EC2 t3.micro or t3.small  

- Optional ALB  

- Lower cost, fast feedback  

- Used for internal QA and developer testing  

### **Staging Environment**

- Mirrors production architecture  

- Has ALB + autoscaling  

- Used for pre-release testing, UAT, demos  

- Connects to staging databases

### **Production Environment**

- ALB with HTTPS (ACM)  

- Rolling or Blue/Green deployments  

- Auto Scaling Group with rules based on:  

  - CPU  

  - Latency  

  - NetworkIn/Out  

- RDS Multi-AZ optionally  

- Strict IAM and SG rules  

- CloudWatch alarms + SNS alerts  

Each environment maintains:

- Independent variables  

- Independent RDS  

- Independent logs  

- Independent deploy history  

---

# 4. How Beanstalk Manages Deployments and Versions

Beanstalk stores "application versions" as ZIP files in S3.  

Each deploy creates a new version containing either:

- Your source code (Node.js platform)  

- A `Dockerrun.aws.json` file pointing to ECR (Docker platform)  

A version includes:

- Metadata  

- Timestamp  

- Deployment configuration  

- Build artifacts  

- Environment variables (bound per environment)  

You can:

- Roll back instantly  

- Promote a version to another environment  

- Compare versions  

- Tag versions for documentation  

This allows us to maintain a clean and auditable release history.

---

# 5. CI/CD Workflow for Backends (Nest.js)

Below is our standardized development workflow.

### **Step-by-step flow**

1. **Local development**  

   - Developer codes in Nest.js  

   - Runs the project locally (Docker or Node)

2. **Feature branch**  

   - Developer pushes changes to a branch:  

     `feat/<feature-name>`

3. **Pull Request to develop**  

   - Code reviewed  

   - Unit tests, linting, validations

4. **Merge to `develop`**  

   - GitHub Actions pipeline triggers:

     - Build Docker image  

     - Push to Amazon ECR  

     - Generate ZIP with `Dockerrun.aws.json`  

     - Upload ZIP to S3  

     - Create new Beanstalk Application Version  

     - Deploy that version to **Beanstalk Dev Environment**

5. **Testing in dev environment**

   - QA and dev team validate  

   - BE available at:  

     `https://api-dev.<domain>.com`

6. **Merge to `main`**  

   - Same pipeline runs but deploys to:

     - Staging environment (optional)

     - Production environment (manual approval required)

7. **Production deployment**

   - New version promoted  

   - ALB handles rolling update  

   - Zero downtime expected  

---

# 6. Amplify Workflow for Frontends (React)

AWS Amplify is our preferred platform for frontend hosting because it offers:

- Full CI/CD from GitHub  

- CDN via CloudFront  

- Automatic HTTPS (ACM)  

- Branch-based environments  

- Very fast deploy times  

- Built-in redirects and SPA support

### **Amplify Deployment Flow**

1. Developer pushes to `feat/<feature>`  

2. PR to `develop`  

3. Merge triggers Amplify build  

4. Amplify deploys frontend to environment **dev**  

   - `https://app-dev.<domain>.com`

5. Merge to `main` deploys automatically to **production**  

   - `https://app.<domain>.com`

Amplify handles:

- Build (npm install, npm build)

- Cache invalidation

- Uploading static files to S3

- CDN distribution

- HTTPS certificates

- Environment variables per branch

This keeps frontend deployments extremely simple and safe.

---

# 7. Our Standard AWS Architecture (Backend + Frontend)

Across almost all projects we use the following architecture:

Route 53 (DNS)

↓

ACM (SSL Certificates)

↓

Application Load Balancer (HTTPS + health checks)

↓

Auto Scaling Group

↓

EC2 instances (private subnets, SG locked down)

↓

RDS (Postgres/MySQL, private subnet)

↓

CloudWatch (logs + metrics + alarms)

Frontend:

GitHub → Amplify → S3 hosting → CloudFront CDN → HTTPS (ACM)

Backend:

GitHub → Docker build → ECR → S3 (app version) → Beanstalk → ALB/ASG/EC2

This stack:

- Scales automatically  

- Keeps traffic encrypted end-to-end  

- Is fully compatible with SOC2  

- Provides deep-level monitoring  

- Allows us to debug and tune performance easily  

- Keeps infra cohesive and consistent across all teams  

---

# 8. Limitations of Elastic Beanstalk to Be Aware Of

We use Beanstalk consciously, knowing its boundaries:

- It always requires a small ZIP in S3 for each application version  

- Deploying very large Docker images increases deploy time  

- Environment configuration is powerful but sometimes verbose  

- Blue/Green deployments require manual orchestration unless automated  

- Does not replace ECS for large-scale microservices  

- Each environment is isolated; there is no native cross-env promotion (we automate it)

However, none of these limitations outweigh the benefits for our current scale and workflows.

---

# 9. Conclusion

Elastic Beanstalk + Docker (when necessary) + Amplify forms a strong, flexible, auditable, and scalable foundation for our projects.  

It enables clean DevOps workflows across environments, supports compliance needs, and provides predictable, maintainable infrastructure without unnecessary complexity.

These guidelines ensure that every project follows the same high standard of deployment, security, monitoring, and operational consistency.

Let's continue building reliable, scalable, and well-managed infrastructure together.
