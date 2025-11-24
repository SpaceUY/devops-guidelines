---
title: Platform Decision - AWS vs Supabase vs Convex
layout: default
nav_order: 4
---

# Decision Guide: NestJS on AWS vs Convex vs Supabase

This document provides practical criteria for choosing where and how to deploy a backend or fullstack application in the company, comparing three main paths:

- **Complete NestJS application hosted on AWS**
- **Convex**
- **Supabase**

This is not an "absolute truth", but rather a decision framework for the DevOps chapter. The goal is to reduce improvisation and ensure each new project starts with a conscious and well-reasoned choice.

## 1. Overview of each option

### 1.1 NestJS hosted on AWS

**What it is**

Complete backend in NestJS (Node.js), deployed on AWS using "classic" services:

- EC2 / ECS / Fargate / Lambda
- RDS (Postgres/MySQL)
- S3, CloudFront, API Gateway, etc.

This is the most flexible and controlled approach: "we are the platform".

**When we use it in the company (in general)**

Core projects, long-lived, where we need:

- High control over architecture
- Regulatory compliance (SOC2, ISO, etc.)
- Complex integrations with other AWS services
- Deep observability, private VPCs, peering, etc.

**Note:** As an AWS Partner, Space prefers AWS solutions when they meet project requirements, leveraging our expertise and partnership benefits.

### 1.2 Convex

**What it is**

Backend platform "everything in TypeScript":

- Proprietary database oriented to documents/light relational
- Queries, mutations, and actions written in TS
- Automatic realtime (queries are reactive, clients update when data changes)

Convex positions itself as "State management backend" for realtime apps: database + server + synchronization, all together.

**When we use it**

Products and features heavily interactive in real-time, with front-heavy teams (React/Next) that want to avoid:

- Managing websockets
- Cache invalidation
- A traditional backend

### 1.3 Supabase

**What it is**

BaaS platform on Postgres:

- Managed Postgres database
- Auth (email, OAuth, SSO)
- Realtime (channels, presence, table changes)
- Edge Functions (TypeScript/JS serverless)
- Storage (S3-like)

It's marketed as "Firebase but with SQL/Postgres".

**When we use it**

Projects that need:

- Serious relational database (auditing, reporting, integrations)
- BaaS features (Auth, Realtime, Storage)
- Without wanting to manage RDS, poolers, or websockets manually

## 2. NestJS on AWS

### 2.1 Description

We're talking about:

- Main backend in NestJS (modular monolith or microservices)
- Deployment on AWS:
  - EC2 + Auto Scaling Group behind an ALB
  - Or ECS/Fargate
  - Or Lambda (Nest adapted to serverless)
- Database in RDS (Postgres or MySQL), within private VPC
- Integrations: S3, SQS, SNS, Step Functions, Secrets Manager, KMS, etc.

### 2.2 Pros

**Maximum flexibility and control:**

- We can apply any architecture pattern (DDD, microservices, CQRS, etc.)
- Full control over network topology, security, VPC, IAM

**Best fit for SOC2/ISO:**

- Full control of logs, auditing, data flows
- Direct integration with compliance tools (CloudTrail, Config, GuardDuty, etc.)

**Complex integrations:**

- Easy to orchestrate calls to external services, queues, lambdas, complex workflows

**Controlled scalability:**

- We can size instances, DB tuning, autoscaling by specific metrics, etc.

**AWS Partnership benefits:**

- Access to AWS support and resources
- Potential cost optimizations through partnership programs
- Alignment with company's cloud strategy

### 2.3 Cons

**Higher operational cost:**

- More DevOps time (pipelines, infrastructure, monitoring)
- More things to maintain (EC2/ECS/RDS, patches, backups, etc.)

**Slower time to market:**

- Compared to BaaS (Supabase/Convex), setting up the entire stack takes longer

**Infrastructure learning curve:**

- Requires a level of AWS maturity to avoid breaking costs or security

### 2.4 Recommended use cases

**Use NestJS on AWS when:**

**Core / regulated / critical project**

- Fintech, exchange, fund custody, systems handling sensitive data
- Requires private VPC, strong integration with AWS services, and compliance

**Complex multi-service platforms**

- Orchestration with queues, lambdas, microservices, multi-region, failover
- Where a single BaaS falls short or would be too strong a lock-in

**Special performance requirements**

- You need fine tuning (e.g., connections, pooling, caches, dedicated game servers, etc.)

## 3. Convex

### 3.1 Description

Convex combines:

- Proprietary database (documents/light relational)
- Backend in TypeScript:
  - Reactive queries
  - Mutations (transactional)
  - Actions (to call external APIs, heavy work)
- Highly integrated React/Next client
- Automatic realtime: when data changes, subscribed queries re-execute and the client updates

### 3.2 Pros

**Very high development speed:**

- Fullstack JS/TS, no SQL, no websockets, no worrying about cache invalidations

**Realtime by default:**

- All views can be "live" without extra effort

**Minimal infrastructure:**

- You don't manage servers, DB, poolers, or sockets

### 3.3 Cons

**Proprietary DB (lock-in):**

- BI/analytics integrations, classic SQL, complex reporting → more work

**Less standard for compliance:**

- If you need SQL, traditional auditing, or replication to warehouses, it's not as direct as Postgres

**Different mental model:**

- You have to adopt the Convex way of thinking about queries and mutations

### 3.4 Recommended use cases

**Use Convex when:**

**Collaborative apps / intensive realtime**

- Notion-lite type, Trello/kanban collaborative, planning poker, internal tools where multiple people edit the same data live

**Internal tools or early-stage SaaS**

- Side projects or early products, with front-heavy teams, where the goal is to iterate quickly

**AI-assisted apps with shared state**

- Dashboards where multiple users see results that change in real-time
- Workflows where an AI agent modifies data live

**Don't use Convex when:**

- The project has strong auditing, SQL reporting, integration with other databases/warehouses requirements
- Or we already know the platform will require large multi-tenant with integration into AWS/BI ecosystem

## 4. Supabase

### 4.1 Description

Supabase offers:

- Managed Postgres (core of everything)
- Ready Auth (email, OAuth, SSO)
- Realtime:
  - Broadcast channels
  - Presence
  - Table change streams
- Edge Functions (serverless TypeScript)
- Storage (similar to S3, with integrated permissions)

In practice, it's "Postgres + modern BaaS".

### 4.2 Pros

**Solid data model (Postgres):**

- Relational, transactions, indexes, views, constraints
- Perfect for auditing, reporting, BI, integrations

**Auth + Realtime + DB integrated:**

- Saves a lot of infrastructure boilerplate

**Row-Level Security (RLS):**

- Declarative policies to control who can see/edit each row

**Auto-scalable Edge Functions:**

- Very convenient for business APIs that don't warrant a full cluster on AWS

### 4.3 Cons

**Less control than "Nest full AWS":**

- You don't choose RDS, nor ultra-low-level tuning (though you can do some)

**Moderate lock-in:**

- Although the DB is standard Postgres, certain features (Auth, specific Realtime) are platform-specific

**Project/plan limits:**

- Need to carefully review Pro/Team quotas to avoid cost surprises

### 4.4 Recommended use cases

**Use Supabase when:**

**SaaS product with strong CRUD + reporting component**

- Multi-tenant, roles, ACLs, dashboards, reports
- Need serious SQL but don't want to manage RDS

**Realtime apps with good portion of relational data**

- Collaborative apps, simple financial apps, games with transactional logic (poker type without "hardcore" regulation)

**MVPs that could grow into serious products**

- Start quickly, but already with Postgres base and RLS, which serves if the project scales and needs to integrate with BI/warehouse

## 5. Cloudflare (Additional Information)

### 5.1 What it is

Cloudflare provides:

- **Workers:** Serverless functions distributed globally, auto-scalable, ideal for APIs, lightweight logic, and middleware
- **Pages:** Static/hybrid hosting for frontends (React, Next static/hybrid, etc.)
- **Durable Objects:** Objects with consistent and unique state per ID, perfect for rooms, sessions, queues, etc.
- **D1 / KV / R2:** SQL database (serverless SQLite) and storage

### 5.2 When it's considered

**Use Cloudflare as a platform when:**

- **Edge-first applications:** Simple public APIs with very high global traffic
- **Static frontends with lightweight logic:** JAMstack with some logic in Workers (AB testing, feature flags, preprocessing)
- **High read traffic with simple logic:** Feature flags, redirection, AB testing
- **Light multiplayer games:** Small multiplayer games with low need for historical auditing and "room = Durable Object" pattern

**Important note for Space:**

As an **AWS Partner**, Space prefers AWS solutions over Cloudflare when both can meet project requirements. AWS is our primary cloud platform, and we leverage our partnership benefits, expertise, and integration capabilities. Cloudflare may be considered for specific edge use cases or as a CDN/WAF layer in front of AWS infrastructure, but AWS remains our preferred choice for core infrastructure.

### 5.3 Pros and Cons

**Pros:**

- Very low global latency (Workers run close to users)
- Almost non-existent infrastructure (no servers to manage, no VPC, no RDS)
- Automatic and granular scaling (pay strictly per requests and CPU)
- Durable Objects facilitate patterns like "one room = one object with state", with guaranteed consistency

**Cons:**

- Limited data model (D1/SQLite is sufficient for small/medium things, but falls short for serious data warehouse/reporting)
- Less fit for classic compliance (can do serious security, but not the typical stack a financial regulator audits compared to RDS in VPC)
- More design work (no Auth + Realtime + DB already packaged; you have to compose pieces)

## 6. Practical decision rules for the DevOps chapter

Here's a simplified "decision tree":

### 6.1 Decision tree

**Is the project core, financial, regulated, or critical for the company/client?**

- **Yes** → start with NestJS on AWS (within VPC, RDS, etc.)
- **No** → continue to point 2

**Do we need strong SQL, reporting/auditing, data integrations (BI)?**

- **Yes** → prefer Supabase as main backend (or Nest+RDS if already on AWS)
- **No, it's more of a UI/collaboration app** → consider Convex

**Is the app highly interactive/realtime, with lots of live shared state between users (Notion, Trello, live dashboards type)?**

- **Yes** → Convex is a good candidate (speed > compliance)
- **Not necessarily** → continue to point 4

**Is the development team very front-heavy (JS/TS) and the project is MVP/experiment?**

- **Yes:**
  - If SQL/reporting is needed → Supabase
  - If it's more shared realtime and without strong compliance → Convex
- **No:**
  - We can go more directly to Nest on AWS or Supabase, thinking long-term

### 6.2 Quick checklist

- Is there strong compliance? → **Nest on AWS**
- Is SQL/reporting needed? → **Supabase or Nest+RDS**
- Is it collaborative realtime and "quick to ship"? → **Convex**
- Is it an MVP with serious data? → **Supabase**

### 6.3 Default option

- **For core products:** NestJS on AWS
- **For MVPs with serious data:** Supabase
- **For highly interactive internal tools:** Convex

## 7. "Type" examples by platform

### 7.1 NestJS on AWS

- Exchange / Fintech with custody, orders, KYC, AML, etc.
- Backend integrators between on-premise systems and various providers (banks, KYC, payments)
- Platforms already deeply tied to AWS (KMS, Secrets Manager, VPC peering, etc.)

### 7.2 Convex

- Internal tools like collaborative kanban, planner, shared docs
- Early-stage collaboration SaaS (whiteboards, comments, live metrics dashboards)
- Small casual multiplayer games or "planning poker" for squads

### 7.3 Supabase

- Online poker application / games with relational logic (tables, hands, bets, chips) with history
- Multi-tenant backoffice for clients (e.g., CRM-type platforms)
- Marketplace / e-commerce lite where we want SQL but without managing RDS

### 7.4 Cloudflare

- Simple public API with very high global traffic
- JAMstack frontends with some logic in Workers (AB testing, feature flags, preprocessing)
- Edge microservices acting as intelligent gateway toward backends on AWS/Supabase

**Note:** For Space, Cloudflare is typically used as a CDN/WAF layer in front of AWS infrastructure rather than as the primary platform.

## 8. Operational recommendations for the DevOps chapter

To standardize decisions, we propose:

### 8.1 Project kickoff document

Always include:

- Nature of the project (core / peripheral / experiment)
- Compliance and auditing requirements
- Team profile (Front-heavy, Full-stack, etc.)
- Traffic expectations and geographic scope

### 8.2 Decision process

1. Review the quick checklist (section 6.2)
2. Apply the decision tree (section 6.1)
3. Document the decision in the project kickoff
4. Periodically review if the decision is still correct as the project evolves

### 8.3 Additional considerations

- **Future migration:** If there are doubts about growth, prefer options that allow easier migration (Supabase with standard Postgres vs Convex with proprietary DB)
- **Long-term cost:** Evaluate not only initial cost but operational and maintenance costs
- **Team capabilities:** Consider the learning curve and expertise available in the team
- **AWS Partnership:** Leverage AWS partnership benefits when AWS solutions meet requirements

## 9. Conclusion

This decision guide seeks to reduce improvisation and standardize the platform selection process for each new project. There is no single solution that works for all cases, but with these criteria we can make more informed and consistent decisions.

The key is to:

- Understand the real needs of the project (compliance, SQL, realtime, etc.)
- Evaluate the trade-off between development speed and control/flexibility
- Consider the team context and available resources
- Think long-term, not just the MVP
- Leverage AWS partnership benefits when appropriate

Let's continue building reliable, scalable, and well-managed infrastructure, choosing the right platform for each use case.
