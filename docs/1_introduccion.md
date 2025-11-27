---
title: Introduccion Beanstalk & Amplify Standard.
layout: default
nav_order: 1
---

# Beanstalk & Amplify Deployment Standard

Welcome to the DevOps Guidelines.  

This document defines how we deploy, manage, and standardize our infrastructure across all projects. The goal is to maintain **consistency, scalability, security, and operational clarity** in every development cycle.

These guidelines formalize our preferred approach:  

**AWS Elastic Beanstalk for backend deployments** (Nest.js, Node.js, and any compute workloads)  

+  

**AWS Amplify for frontend deployments** (React, Vite, Next.js).

This combination gives us a modern, efficient, auditable, and secure foundation that scales with project requirements.

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

# 2. Elastic Beanstalk Deployment Options: EC2 Native vs Docker

We support two deployment approaches in Elastic Beanstalk, depending on the project requirements:

## 2.1 EC2 Native Deployment (Standard - Single Nest.js Application)

This is our **default and preferred approach** for most projects that deploy a single Nest.js application.

### When to use EC2 Native

- Single Nest.js backend application
- Standard Node.js runtime requirements
- Simpler deployment process
- Faster deployment times
- Lower resource overhead

### How EC2 Native works in Beanstalk

Beanstalk deploys your application directly on EC2 instances using the Node.js platform:

- Your source code is packaged as a ZIP file
- Beanstalk installs Node.js and npm on the EC2 instance
- Runs `npm install` to install dependencies
- Starts your application using the configured start command (e.g., `npm start`)
- Monitors application health via health check endpoints
- Auto-restarts or replaces unhealthy instances

### Deployment process

1. Package your Nest.js application source code
2. Upload ZIP to S3 as an Application Version
3. Beanstalk extracts and installs dependencies
4. Application runs natively on EC2

This approach is simpler, faster, and requires less infrastructure overhead.

## 2.2 Docker Deployment (When Multiple Nest.js Applications Are Needed)

We use Docker when you need to deploy **more than one Nest.js application in the same Beanstalk environment**, or when you need strict runtime reproducibility and OS-level dependencies.

### When to use Docker

- **Multiple Nest.js applications** sharing the same Beanstalk deployment
- Multiple APIs that need to run together in one environment
- Need for OS-level dependencies (FFmpeg, Python scripts, system libraries, etc.)
- Strict consistency between local dev / CI / production environments
- Preparing future migration to ECS/EKS

### How Docker works in Beanstalk

We deploy via:

- **Single-container Docker** (one Nest.js app per container)
- **Multi-container (docker-compose) via ECS integrated mode** (multiple Nest.js apps or services)

Beanstalk will:

- Pull the Docker image(s) from **Amazon ECR**
- Start the container(s) on EC2 instances
- Expose the defined ports to the ALB
- Monitor container health
- Auto-restart or replace unhealthy containers

### Deployment process

1. Build Docker image(s) containing your Nest.js application(s)
2. Push image(s) to Amazon ECR
3. Create `Dockerrun.aws.json` file pointing to ECR image(s)
4. Package `Dockerrun.aws.json` in a ZIP and upload to S3
5. Beanstalk pulls image(s) from ECR and starts container(s)

### Limitations to consider

- Application Version bundles still require a small ZIP in S3 (even when using ECR)
- EC2 instance disk size must be enough to store images (default 8–20GB can be increased)
- Deployments of large images may take longer
- Logs inside containers are streamed to CloudWatch but require correct config
- More complex setup and maintenance compared to native deployments

### Decision Matrix

| Scenario | Recommended Approach |
|----------|---------------------|
| Single Nest.js application | **EC2 Native** |
| Multiple Nest.js applications in one environment | **Docker (Multi-container)** |
| Need OS-level dependencies | **Docker** |
| Strict dev/prod consistency | **Docker** |
| Fastest deployment times | **EC2 Native** |
| Simpler setup | **EC2 Native** |

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

# 4. How Beanstalk Manages Deployments and Versions

Beanstalk stores "application versions" as ZIP files in S3.  

Each deploy creates a new version containing either:

- Your source code (EC2 Native / Node.js platform)  

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

# 5. CI/CD Workflow for Backends (Nest.js)

Below is our standardized development workflow for both deployment approaches.

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

     **For EC2 Native deployments:**
     - Package source code into ZIP
     - Upload ZIP to S3
     - Create new Beanstalk Application Version
     - Deploy that version to **Beanstalk Dev Environment**

     **For Docker deployments:**
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

Backend (EC2 Native):

GitHub → Source code ZIP → S3 (app version) → Beanstalk → ALB/ASG/EC2 (Node.js runtime)

Backend (Docker):

GitHub → Docker build → ECR → S3 (app version with Dockerrun.aws.json) → Beanstalk → ALB/ASG/EC2 (Docker containers)

This stack:

- Scales automatically  

- Keeps traffic encrypted end-to-end  

- Is fully compatible with SOC2  

- Provides deep-level monitoring  

- Allows us to debug and tune performance easily  

- Keeps infra cohesive and consistent across all teams  

# 8. Limitations of Elastic Beanstalk to Be Aware Of

We use Beanstalk consciously, knowing its boundaries:

- It always requires a small ZIP in S3 for each application version  

- Deploying very large Docker images increases deploy time  

- Environment configuration is powerful but sometimes verbose  

- Blue/Green deployments require manual orchestration unless automated  

- Does not replace ECS for large-scale microservices  

- Each environment is isolated; there is no native cross-env promotion (we automate it)

However, none of these limitations outweigh the benefits for our current scale and workflows.

# 9. When NOT to Use Elastic Beanstalk

While Beanstalk is our preferred solution for most backend deployments, there are specific scenarios where alternative approaches are more appropriate:

## 9.1 High-Concurrency with Serverless (Lambda)

When a project requires **extremely high concurrency** with unpredictable traffic patterns, AWS Lambda with API Gateway is a better fit.

**Use Lambda instead when:**
- Traffic spikes are sudden and unpredictable
- Request patterns are event-driven or sporadic
- You need to scale to zero during idle periods
- Cost optimization for low-to-medium traffic is critical
- Request processing is stateless and short-lived

**Example scenarios:**
- Webhook handlers
- API endpoints with burst traffic
- Event processing pipelines
- Real-time data transformation

## 9.2 Complex Multi-EC2 Infrastructure or Multi-Tenant Deployments (SST)

When a project requires **interconnected infrastructure across multiple EC2 instances** or needs **rapid deployments to multiple clients** with the same infrastructure stack, **SST (Serverless Stack)** provides better infrastructure-as-code capabilities.

**Use SST instead when:**
- You need to deploy the same infrastructure to multiple tenants/clients quickly
- Infrastructure involves multiple interconnected EC2 instances, services, and resources
- You require more granular control over infrastructure components
- You need to manage complex multi-service architectures
- Infrastructure needs to be versioned and replicated across environments easily

**Example scenarios:**
- Multi-tenant SaaS platforms
- Complex microservices architectures with multiple services
- Infrastructure that needs frequent replication
- Projects requiring advanced infrastructure orchestration

## 9.3 Pure Microservices Architecture

When a project is designed as **pure microservices** with independent scaling, deployment, and lifecycle management, container orchestration platforms are more suitable.

**Use ECS/EKS instead when:**
- Services need completely independent scaling policies
- Services have different resource requirements (CPU, memory)
- Services require different deployment schedules
- You need service mesh capabilities (Istio, App Mesh)
- Services communicate via service discovery and internal APIs
- You require advanced container orchestration features

**Example scenarios:**
- Large-scale distributed systems
- Services with vastly different performance characteristics
- Systems requiring service-to-service communication patterns
- Projects with dedicated DevOps teams managing orchestration

## 9.4 Batch/Queue-Heavy Workloads (SQS + Lambda)

When a project is primarily **queue-driven** with batch processing requirements, a combination of SQS and Lambda (or Step Functions) is more efficient than maintaining always-on EC2 instances.

**Use SQS + Lambda instead when:**
- Workload is primarily asynchronous and queue-based
- Processing can be done in small, discrete chunks
- Jobs can be processed independently
- You want to pay only for processing time (not idle time)
- Processing patterns are event-driven or scheduled

**Example scenarios:**
- Image/video processing pipelines
- Data ETL jobs
- Email sending queues
- Report generation systems
- Scheduled batch operations

## Decision Summary

| Scenario | Recommended Solution |
|----------|---------------------|
| Standard Nest.js API with predictable traffic | **Elastic Beanstalk** |
| High concurrency, unpredictable spikes | **Lambda + API Gateway** |
| Multi-tenant, complex infrastructure | **SST** |
| Pure microservices architecture | **ECS/EKS** |
| Queue-heavy, batch processing | **SQS + Lambda** |
| Multiple Nest.js apps in one deployment | **Beanstalk with Docker** |

# 10. Conclusion

Elastic Beanstalk (EC2 Native for single apps, Docker for multiple apps) + Amplify forms a strong, flexible, auditable, and scalable foundation for our projects.  

It enables clean DevOps workflows across environments, supports compliance needs, and provides predictable, maintainable infrastructure without unnecessary complexity.

These guidelines ensure that every project follows the same high standard of deployment, security, monitoring, and operational consistency.

Let's continue building reliable, scalable, and well-managed infrastructure together.
