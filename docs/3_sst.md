---
title: SST
layout: default
nav_order: 3
---

# SST (Serverless Stack)

> **Important Note:** As mentioned in the introduction, for projects that don't have the complexity to justify SST, you should use **Elastic Beanstalk with Amplify** as the default stack. SST is reserved for multi-tenant projects, complex infrastructure with multiple interconnected services, or when you need to quickly replicate the same infrastructure for multiple clients.

## Official Template

We have created a reference template with all the necessary infrastructure for complex projects:

🔗 **[SpaceUY/sst-template](https://github.com/SpaceUY/sst-template)**

This template provides a comprehensive Infrastructure as Code (IaC) solution built with SST for deploying and managing containerized applications on AWS ECS Fargate.

---

## What is SST?

SST (Serverless Stack) is an Infrastructure as Code framework that allows you to define and deploy AWS infrastructure using TypeScript. Unlike Terraform or CloudFormation, SST:

- Uses native TypeScript to define resources
- Provides hot-reload for local Lambda development
- Integrates natively with the Node.js ecosystem
- Enables high-level abstractions over AWS resources

## When to Use SST?

| Scenario | Use SST |
|----------|---------|
| Multi-tenant SaaS with replicable infrastructure | ✅ Yes |
| Multiple interconnected ECS services | ✅ Yes |
| Complex infrastructure with RDS, Redis, shared ALB | ✅ Yes |
| Automated monitoring with health checks | ✅ Yes |
| Single standard Nest.js API | ❌ Use Beanstalk |
| Simple frontend with basic API | ❌ Use Amplify + Beanstalk |

## Template Architecture

The template implements the following architecture:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              AWS Cloud                                       │
│                                                                              │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐       │
│  │   Route53 DNS    │───▶│   Application    │───▶│   ECS Fargate    │       │
│  │                  │    │   Load Balancer  │    │   Service        │       │
│  └──────────────────┘    └──────────────────┘    └────────┬─────────┘       │
│                                                           │                  │
│                          ┌────────────────────────────────┼────────────────┐│
│                          │                                │                ││
│                          ▼                                ▼                ││
│                  ┌──────────────┐                 ┌──────────────┐         ││
│                  │   RDS        │                 │   Redis      │         ││
│                  │   PostgreSQL │                 │   ElastiCache│         ││
│                  └──────────────┘                 └──────────────┘         ││
│                                                                            ││
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      ││
│  │  CloudWatch  │  │  AWS Backup  │  │   Secrets    │                      ││
│  │  Monitoring  │  │  Cross-Region│  │   Manager    │                      ││
│  └──────────────┘  └──────────────┘  └──────────────┘                      ││
└─────────────────────────────────────────────────────────────────────────────┘
```

## Conclusion

The SST template provides a solid foundation for projects that require:

- Replicable multi-tenant infrastructure
- Multiple ECS services with dedicated databases and cache
- Automated health monitoring
- Cross-region backups
- Centralized secrets management

For simpler projects, remember that **Elastic Beanstalk + Amplify** remains our preferred and recommended option.
