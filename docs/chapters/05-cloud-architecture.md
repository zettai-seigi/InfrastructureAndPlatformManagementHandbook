---
layout: default
title: "Chapter 5: Cloud Platform Architecture"
parent: "Part II: Architecture and Design"
nav_order: 5
permalink: /chapters/05-cloud-architecture/
---

# Chapter 5: Cloud Platform Architecture

## Learning Objectives

After completing this chapter, you will be able to:
- Evaluate cloud deployment models for different use cases
- Design multi-cloud and hybrid cloud architectures
- Apply the shared responsibility model appropriately
- Select cloud services based on requirements
- Implement cloud-native design principles
- Apply the Well-Architected Framework to cloud solutions
- Design landing zones and account structures
- Implement cloud governance and compliance controls

---

## Introduction

Cloud platforms have transformed infrastructure management, offering unprecedented flexibility, scalability, and innovation. However, cloud success requires thoughtful architecture that leverages cloud benefits while managing complexity, cost, and risk.

This chapter covers cloud platform architecture across public, private, hybrid, and multi-cloud environments. We'll explore how to make strategic decisions about cloud adoption and design architectures that maximize cloud value while maintaining appropriate governance and control.

---

## Cloud Deployment Models

### Public Cloud

Infrastructure owned and operated by cloud providers, shared across customers with logical isolation.

| Characteristic | Description | Business Impact |
|----------------|-------------|-----------------|
| **Ownership** | Provider owns all infrastructure | No capital expenditure |
| **Tenancy** | Multi-tenant, logically isolated | Shared security responsibility |
| **Economics** | Pay-as-you-go, consumption-based | Operational expenditure model |
| **Scalability** | Virtually unlimited | Handle any growth |
| **Geographic Reach** | Global infrastructure | Worldwide deployment |
| **Innovation** | Continuous new services | Access to latest technology |

**Major Public Cloud Providers**:

| Provider | Strengths | Best For |
|----------|-----------|----------|
| **AWS** | Broadest services, largest market share | General workloads, innovation |
| **Azure** | Enterprise integration, Windows workloads | Microsoft shops, hybrid |
| **GCP** | Data/ML, Kubernetes heritage | Analytics, modern apps |
| **Oracle Cloud** | Database workloads | Oracle customers |
| **IBM Cloud** | Enterprise, mainframe integration | Regulated industries |

**When to Use Public Cloud**:
- Variable or unpredictable workloads
- Rapid scaling requirements
- Global deployment needs
- Innovation and experimentation
- Startup and development environments
- Disaster recovery and backup

### Private Cloud

Dedicated cloud infrastructure for a single organization, either on-premises or hosted.

| Characteristic | Description | Business Impact |
|----------------|-------------|-----------------|
| **Ownership** | Organization or provider owned | Capital investment required |
| **Tenancy** | Single tenant, dedicated | Full control over security |
| **Economics** | Capital or operational | More predictable costs |
| **Scalability** | Limited by capacity | Plan for growth |
| **Control** | Complete infrastructure control | Customization options |
| **Compliance** | Easier to meet strict requirements | Regulatory alignment |

**Private Cloud Technologies**:

| Technology | Description | Use Case |
|------------|-------------|----------|
| **VMware vSphere** | Enterprise virtualization | Existing VMware investment |
| **OpenStack** | Open-source cloud platform | Avoid vendor lock-in |
| **Azure Stack** | Azure services on-premises | Hybrid Azure |
| **AWS Outposts** | AWS services on-premises | Hybrid AWS |
| **Nutanix** | Hyper-converged infrastructure | Simplified management |

**When to Use Private Cloud**:
- Strict compliance requirements (HIPAA, PCI, government)
- Data sovereignty requirements
- Predictable, steady-state workloads
- Legacy applications with licensing constraints
- Ultra-low latency requirements
- High-security workloads

### Hybrid Cloud

Combination of on-premises infrastructure (or private cloud) and public cloud resources, operating as a unified environment.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       HYBRID CLOUD ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ON-PREMISES / PRIVATE CLOUD           PUBLIC CLOUD                          │
│  ┌───────────────────────────┐         ┌───────────────────────────┐        │
│  │                           │         │                           │        │
│  │  Core Systems:            │         │  Extended Capabilities:   │        │
│  │  ┌─────────────────────┐  │         │  ┌─────────────────────┐  │        │
│  │  │ Core Databases      │  │         │  │ Web Applications    │  │        │
│  │  │ (Customer Data)     │  │         │  │ (Customer-facing)   │  │        │
│  │  └─────────────────────┘  │         │  └─────────────────────┘  │        │
│  │  ┌─────────────────────┐  │         │  ┌─────────────────────┐  │        │
│  │  │ Legacy ERP/Finance  │  │◄───────►│  │ Dev/Test Envs       │  │        │
│  │  │ Systems             │  │   VPN   │  │ (Rapid provisioning)│  │        │
│  │  └─────────────────────┘  │   or    │  └─────────────────────┘  │        │
│  │  ┌─────────────────────┐  │ Direct  │  ┌─────────────────────┐  │        │
│  │  │ Regulated Workloads │  │ Connect │  │ Analytics/BigData   │  │        │
│  │  │ (Compliance)        │  │         │  │ (Scale computing)   │  │        │
│  │  └─────────────────────┘  │         │  └─────────────────────┘  │        │
│  │  ┌─────────────────────┐  │         │  ┌─────────────────────┐  │        │
│  │  │ Identity (AD/LDAP)  │  │         │  │ DR/Backup           │  │        │
│  │  │ Master              │  │         │  │ (Off-site copy)     │  │        │
│  │  └─────────────────────┘  │         │  └─────────────────────┘  │        │
│  │                           │         │                           │        │
│  └───────────────────────────┘         └───────────────────────────┘        │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    UNIFIED MANAGEMENT LAYER                          │    │
│  │  Identity │ Networking │ Security │ Monitoring │ Governance          │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Hybrid Connectivity Options**:

| Option | Bandwidth | Latency | Cost | Use Case |
|--------|-----------|---------|------|----------|
| **Site-to-Site VPN** | Limited | Variable | Low | Development, backup |
| **Direct Connect/ExpressRoute** | High | Low | Medium | Production workloads |
| **SD-WAN** | Variable | Optimized | Medium | Multiple locations |
| **Dedicated Fiber** | Very High | Very Low | High | Critical workloads |

**Hybrid Use Cases**:
- Cloud bursting for peak capacity
- Disaster recovery to cloud
- Gradual cloud migration
- Development in cloud, production on-premises
- Compliance requiring on-premises data

### Multi-Cloud

Using multiple public cloud providers strategically.

| Strategy | Description | Benefits | Challenges |
|----------|-------------|----------|------------|
| **Best of Breed** | Different clouds for different workloads | Optimal service selection | Complexity, multiple skills |
| **Redundancy** | Same workload across clouds | High availability | Cost, synchronization |
| **Avoid Lock-in** | Preserve portability | Negotiating leverage | Lowest common denominator |
| **Geographic** | Clouds for specific regions | Data residency | Compliance management |
| **Cost Arbitrage** | Use cheapest provider | Cost optimization | Constant comparison |

**Multi-Cloud Architecture Pattern**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       MULTI-CLOUD ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    ABSTRACTION LAYER                                 │    │
│  │  Kubernetes │ Terraform │ HashiCorp Vault │ Prometheus/Grafana      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                              │                                               │
│           ┌──────────────────┼──────────────────┐                           │
│           │                  │                  │                           │
│           ▼                  ▼                  ▼                           │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐               │
│  │       AWS       │ │      Azure      │ │       GCP       │               │
│  ├─────────────────┤ ├─────────────────┤ ├─────────────────┤               │
│  │ • EKS clusters  │ │ • AKS clusters  │ │ • GKE clusters  │               │
│  │ • Data lakes    │ │ • M365 workloads│ │ • ML workloads  │               │
│  │ • IoT workloads │ │ • AD integration│ │ • BigQuery      │               │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Shared Responsibility Model

Understanding the shared responsibility model is critical for cloud security and operations.

### Responsibility by Service Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SHARED RESPONSIBILITY MODEL                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                      IaaS          PaaS          SaaS                       │
│                                                                              │
│  Data              ██████████    ██████████    ██████████   Customer        │
│  Application       ██████████    ██████████    ░░░░░░░░░░   manages         │
│  Runtime           ██████████    ░░░░░░░░░░    ░░░░░░░░░░                   │
│  Middleware        ██████████    ░░░░░░░░░░    ░░░░░░░░░░                   │
│  Operating System  ██████████    ░░░░░░░░░░    ░░░░░░░░░░                   │
│  Virtualization    ░░░░░░░░░░    ░░░░░░░░░░    ░░░░░░░░░░   Provider        │
│  Servers           ░░░░░░░░░░    ░░░░░░░░░░    ░░░░░░░░░░   manages         │
│  Storage           ░░░░░░░░░░    ░░░░░░░░░░    ░░░░░░░░░░                   │
│  Networking        ░░░░░░░░░░    ░░░░░░░░░░    ░░░░░░░░░░                   │
│  Physical Security ░░░░░░░░░░    ░░░░░░░░░░    ░░░░░░░░░░                   │
│                                                                              │
│  ██████████ = Customer Responsibility                                        │
│  ░░░░░░░░░░ = Provider Responsibility                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Detailed Responsibility Matrix

| Responsibility Area | IaaS Customer | PaaS Customer | SaaS Customer | Provider |
|---------------------|---------------|---------------|---------------|----------|
| **Data Classification** | Yes | Yes | Yes | No |
| **Data Encryption (at rest)** | Configure | Configure | Limited | Enable option |
| **Data Encryption (in transit)** | Configure | Configure | Limited | Enable option |
| **Identity Management** | Configure IAM | Configure IAM | Configure SSO | Provide service |
| **Application Security** | Full | Partial | No | Varies |
| **Network Configuration** | Configure | Limited | No | Provide/manage |
| **OS Patching** | Full | No | No | Full |
| **Physical Security** | No | No | No | Full |
| **Compliance Certifications** | Leverage | Leverage | Leverage | Obtain/maintain |

### Security Responsibility Details

| Security Domain | Customer Responsibilities | Provider Responsibilities |
|-----------------|---------------------------|---------------------------|
| **Identity & Access** | Create/manage users, assign permissions, enable MFA, review access | Provide IAM services, secure authentication infrastructure |
| **Data Protection** | Classify data, enable encryption, manage keys, implement DLP | Provide encryption options, secure storage, HSM services |
| **Network Security** | Configure security groups, NACLs, WAF rules | Secure physical network, DDoS protection infrastructure |
| **Compute Security** | Harden images, patch OS (IaaS), secure applications | Secure hypervisors, physical hosts |
| **Logging & Monitoring** | Enable logging, configure alerts, review logs | Provide logging services, secure log infrastructure |
| **Incident Response** | Respond to incidents in your systems | Respond to infrastructure incidents, notify customers |

---

## Cloud Service Selection

### Compute Services

| Service Type | Description | Best For | Examples |
|--------------|-------------|----------|----------|
| **Virtual Machines** | Full OS control, traditional apps | Lift-and-shift, custom requirements | EC2, Azure VMs, GCE |
| **Containers (Managed)** | Kubernetes orchestration | Microservices, portability | EKS, AKS, GKE |
| **Containers (Serverless)** | Containers without managing servers | Event-driven containers | Fargate, Container Instances |
| **Serverless Functions** | Event-driven code execution | APIs, automation, triggers | Lambda, Functions, Cloud Functions |
| **Bare Metal** | Dedicated physical servers | High performance, compliance | EC2 Metal, Bare Metal Solution |

**Compute Selection Decision Tree**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COMPUTE SERVICE SELECTION                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Start: What are your requirements?                                          │
│                                                                              │
│  Need full OS control? ──Yes──► Virtual Machines                            │
│         │                                                                    │
│         No                                                                   │
│         ▼                                                                    │
│  Container workload? ──Yes──► Need cluster management? ──Yes──► Managed K8s │
│         │                              │                                     │
│         │                              No                                    │
│         │                              ▼                                     │
│         │                      Serverless Containers                         │
│         No                                                                   │
│         ▼                                                                    │
│  Event-driven/Short-lived? ──Yes──► Serverless Functions                    │
│         │                                                                    │
│         No                                                                   │
│         ▼                                                                    │
│  Need bare metal performance? ──Yes──► Bare Metal                           │
│         │                                                                    │
│         No                                                                   │
│         ▼                                                                    │
│  Default: Virtual Machines or Containers                                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Storage Services

| Service Type | Characteristics | Use Cases | Examples |
|--------------|-----------------|-----------|----------|
| **Block Storage** | Low latency, high IOPS | Databases, boot volumes | EBS, Azure Disk, Persistent Disk |
| **File Storage** | Shared access, POSIX | Legacy apps, shared data | EFS, Azure Files, Filestore |
| **Object Storage** | Scalable, cheap, HTTP access | Backups, data lakes, static content | S3, Blob Storage, Cloud Storage |

**Storage Performance Tiers**:

| Tier | IOPS | Throughput | Latency | Cost | Use Case |
|------|------|------------|---------|------|----------|
| **Premium/Provisioned** | 64,000+ | 1,000+ MB/s | <1ms | $$$ | High-performance databases |
| **General Purpose** | 3,000-16,000 | 125-1,000 MB/s | 1-10ms | $$ | Most workloads |
| **Throughput Optimized** | Lower | Higher | Variable | $$ | Big data, logs |
| **Cold/Archive** | N/A | Limited | Minutes-hours | $ | Backup, archive |

### Database Services

| Category | Service Type | Best For | Examples |
|----------|--------------|----------|----------|
| **Relational** | Managed RDBMS | OLTP, structured data | RDS, Azure SQL, Cloud SQL |
| **NoSQL Document** | Document stores | Flexible schema, content | DocumentDB, Cosmos DB, Firestore |
| **NoSQL Key-Value** | Key-value stores | Caching, sessions | DynamoDB, Table Storage |
| **In-Memory** | In-memory databases | Caching, real-time | ElastiCache, Memorystore |
| **Time Series** | Time-series data | IoT, metrics | Timestream, Influx |
| **Graph** | Graph databases | Relationships | Neptune, Cosmos DB (Gremlin) |
| **Data Warehouse** | OLAP, analytics | BI, reporting | Redshift, Synapse, BigQuery |

### Networking Services

| Service Category | Services | Use Cases |
|------------------|----------|-----------|
| **Virtual Networks** | VPC, VNet | Network isolation, segmentation |
| **Load Balancing** | ALB, NLB, Azure LB | Traffic distribution, HA |
| **DNS** | Route 53, Azure DNS | Name resolution, routing |
| **CDN** | CloudFront, Azure CDN | Content delivery, edge caching |
| **Connectivity** | Direct Connect, ExpressRoute | Hybrid connectivity |
| **API Gateway** | API Gateway | API management, authentication |

---

## Cloud Landing Zone Design

A landing zone is a pre-configured, secure, multi-account cloud environment.

### Landing Zone Components

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       CLOUD LANDING ZONE                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    MANAGEMENT ACCOUNT/SUBSCRIPTION                   │    │
│  │   Identity │ Billing │ Organizations │ SSO │ Control Tower          │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌──────────────────────┐  ┌──────────────────────┐                        │
│  │    SECURITY OU       │  │     SHARED OU        │                        │
│  │  ┌────────────────┐  │  │  ┌────────────────┐  │                        │
│  │  │ Security       │  │  │  │ Shared Services│  │                        │
│  │  │ Account        │  │  │  │ Account        │  │                        │
│  │  │ - GuardDuty    │  │  │  │ - Transit GW   │  │                        │
│  │  │ - Security Hub │  │  │  │ - DNS          │  │                        │
│  │  │ - Config       │  │  │  │ - Directory    │  │                        │
│  │  └────────────────┘  │  │  └────────────────┘  │                        │
│  │  ┌────────────────┐  │  │  ┌────────────────┐  │                        │
│  │  │ Log Archive    │  │  │  │ Network Hub    │  │                        │
│  │  │ Account        │  │  │  │ Account        │  │                        │
│  │  │ - CloudTrail   │  │  │  │ - Transit GW   │  │                        │
│  │  │ - Centralized  │  │  │  │ - Inspection   │  │                        │
│  │  └────────────────┘  │  │  └────────────────┘  │                        │
│  └──────────────────────┘  └──────────────────────┘                        │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                        WORKLOAD OUs                                   │   │
│  │                                                                       │   │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌─────────────┐  │   │
│  │  │ Production   │ │ Non-Prod     │ │ Sandbox      │ │ Restricted  │  │   │
│  │  │ OU           │ │ OU           │ │ OU           │ │ OU          │  │   │
│  │  │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌─────────┐ │  │   │
│  │  │ │ Prod-App1│ │ │ │ Dev-App1 │ │ │ │ Sandbox1 │ │ │ │ PCI-Env │ │  │   │
│  │  │ │ Account  │ │ │ │ Account  │ │ │ │ Account  │ │ │ │ Account │ │  │   │
│  │  │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │ │ └─────────┘ │  │   │
│  │  │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌─────────┐ │  │   │
│  │  │ │ Prod-App2│ │ │ │ Staging  │ │ │ │ Sandbox2 │ │ │ │ HIPAA   │ │  │   │
│  │  │ │ Account  │ │ │ │ Account  │ │ │ │ Account  │ │ │ │ Account │ │  │   │
│  │  │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │ │ └─────────┘ │  │   │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └─────────────┘  │   │
│  │                                                                       │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Account/Subscription Strategy

| Account Type | Purpose | Governance |
|--------------|---------|------------|
| **Management** | Organization root, billing, SSO | Most restrictive |
| **Security** | Security services, GuardDuty, Security Hub | Security team only |
| **Log Archive** | Centralized logging, audit logs | Read-only, immutable |
| **Shared Services** | Transit networking, directory, DNS | Infrastructure team |
| **Production** | Production workloads | Change-controlled |
| **Non-Production** | Dev, test, staging | Less restrictive |
| **Sandbox** | Experimentation, learning | Self-service with limits |

### Guardrails and Policies

| Guardrail Type | Description | Examples |
|----------------|-------------|----------|
| **Preventive** | Stop non-compliant actions | SCP blocking public S3 |
| **Detective** | Alert on non-compliance | Config rules |
| **Proactive** | Validate before deployment | CloudFormation hooks |

**Common Guardrails**:

| Category | Guardrails |
|----------|------------|
| **Identity** | Require MFA, no root access, federated identity |
| **Data** | Encrypt by default, no public storage, DLP |
| **Network** | No public access, approved CIDRs only, flow logs |
| **Compute** | Approved AMIs only, no public instances, patching |
| **Logging** | CloudTrail enabled, logs centralized, immutable |

---

## Cloud-Native Design Principles

### Twelve-Factor App Principles for Infrastructure

| Factor | Traditional | Cloud-Native |
|--------|-------------|--------------|
| **Codebase** | Manual changes, undocumented | IaC in version control |
| **Dependencies** | Implicit, manual installs | Explicitly declared, automated |
| **Configuration** | Embedded in code | Environment variables, external |
| **Backing Services** | Tightly coupled | Attached resources, loosely coupled |
| **Build, Release, Run** | Manual deployments | CI/CD pipelines |
| **Processes** | Stateful servers | Stateless, externalized state |
| **Port Binding** | Fixed ports, manual config | Self-contained, dynamic |
| **Concurrency** | Vertical scaling | Horizontal scaling |
| **Disposability** | Long-running, hard to replace | Fast start, graceful shutdown |
| **Dev/Prod Parity** | Significant differences | Maximum similarity |
| **Logs** | Local files | Event streams, centralized |
| **Admin Processes** | Manual operations | One-off containers, automation |

### Well-Architected Framework

| Pillar | Description | Key Practices |
|--------|-------------|---------------|
| **Operational Excellence** | Run and monitor systems effectively | IaC, monitoring, runbooks, automation |
| **Security** | Protect information and systems | IAM, encryption, detection, response |
| **Reliability** | Recover from failures, meet demand | Multi-AZ, backups, testing, scaling |
| **Performance Efficiency** | Use resources efficiently | Right-sizing, caching, CDN |
| **Cost Optimization** | Avoid unnecessary costs | Reserved, right-size, cleanup |
| **Sustainability** | Minimize environmental impact | Efficient resources, right region |

### Cloud-Native Architecture Patterns

| Pattern | Description | Benefits |
|---------|-------------|----------|
| **Microservices** | Small, independently deployable services | Agility, scalability, resilience |
| **Event-Driven** | Asynchronous, loosely coupled | Scalability, decoupling |
| **Serverless-First** | Functions and managed services | Reduced operations, pay-per-use |
| **Container-Based** | Containerized workloads | Portability, consistency |
| **API-First** | Design APIs before implementation | Integration, reuse |

---

## Cloud Cost Management

### FinOps Practices

| Practice | Description | Implementation |
|----------|-------------|----------------|
| **Visibility** | See all cloud spending | Cost Explorer, Kubecost |
| **Allocation** | Attribute costs to owners | Tagging, chargeback |
| **Optimization** | Reduce waste, improve efficiency | Right-sizing, reserved capacity |
| **Governance** | Control spending | Budgets, alerts, approval workflows |

### Cost Optimization Strategies

| Strategy | Savings Potential | Effort | Risk |
|----------|-------------------|--------|------|
| **Shut Down Unused** | 20-40% | Low | Low |
| **Right-Size** | 15-35% | Medium | Low |
| **Reserved Instances** | 30-72% | Medium | Medium |
| **Spot/Preemptible** | 60-90% | High | Higher |
| **Modern Architecture** | 20-50% | High | Medium |

### Cloud Cost Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       CLOUD COST COMPONENTS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Total Cost = Compute + Storage + Network + Services + Support              │
│                                                                              │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────┐  │
│  │  COMPUTE   │ │  STORAGE   │ │  NETWORK   │ │  SERVICES  │ │ SUPPORT  │  │
│  │            │ │            │ │            │ │            │ │          │  │
│  │ • VM hours │ │ • Capacity │ │ • Egress   │ │ • Database │ │ • Plans  │  │
│  │ • vCPU     │ │ • IOPS     │ │ • Data     │ │ • Analytics│ │ • TAM    │  │
│  │ • Memory   │ │ • Requests │ │   transfer │ │ • Security │ │          │  │
│  │ • GPU      │ │ • Retrieval│ │ • Load     │ │ • AI/ML    │ │          │  │
│  │            │ │            │ │   balancers│ │            │ │          │  │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘ └──────────┘  │
│                                                                              │
│  Optimization Levers:                                                        │
│  • Reserved capacity (committed spend)                                       │
│  • Right-sizing (match to actual usage)                                      │
│  • Spot/preemptible (spare capacity)                                         │
│  • Storage tiering (lifecycle policies)                                      │
│  • Architecture optimization (reduce data movement)                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Cloud Migration Strategies

### The 7 Rs of Migration

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| **Retire** | Decommission, no longer needed | Unused applications |
| **Retain** | Keep in current location | Not ready, compliance |
| **Relocate** | Move with minimal changes | VMware to VMware Cloud |
| **Rehost** | "Lift and shift" to cloud | Quick migration, legacy |
| **Replatform** | Minor modifications | Database to RDS |
| **Refactor** | Re-architect for cloud-native | Strategic applications |
| **Repurchase** | Move to SaaS | Replace with cloud service |

### Migration Phases

| Phase | Activities | Outputs |
|-------|------------|---------|
| **Assess** | Discover, analyze, plan | Migration strategy, wave plan |
| **Mobilize** | Set up landing zone, train teams | Ready environment |
| **Migrate** | Move workloads in waves | Migrated applications |
| **Modernize** | Optimize for cloud | Cloud-native applications |

---

## Review Questions

1. **Deployment Model Selection**: A healthcare organization needs to deploy a patient portal. They have strict HIPAA requirements but want cloud scalability. Which deployment model would you recommend and why?

2. **Shared Responsibility**: A company using AWS Lambda had a data breach because their function logged sensitive data to CloudWatch. Was this AWS's responsibility or the customer's? Explain using the shared responsibility model.

3. **Service Selection**: You need to choose a database for a new application with the following requirements: ACID transactions, scale to 1M reads/second, global distribution. Which cloud database service would you recommend?

4. **Landing Zone Design**: Design an account structure for a company with 5 business units, each with dev/test/prod environments, plus shared services. How many accounts would you create?

5. **Cost Optimization**: Your cloud bill shows 40% of costs are from EC2, with average utilization of 15%. What optimization strategies would you recommend?

6. **Migration Strategy**: You have a legacy monolithic application on VMware. It has complex database dependencies and custom middleware. Which migration strategy would you recommend?

---

## Key Takeaways

- **Deployment models** (public, private, hybrid, multi-cloud) each serve different needs
- **Shared responsibility** clearly defines security boundaries between customer and provider
- **Service selection** should align with workload requirements, not vendor preference
- **Landing zones** provide the foundation for secure, scalable cloud environments
- **Cloud-native principles** maximize cloud benefits and reduce operational burden
- **Cost management** requires visibility, allocation, optimization, and governance
- **Migration strategies** vary based on application characteristics and business goals

---

## Summary

Cloud platform architecture requires understanding deployment models, the shared responsibility model, service options, and cloud-native design principles. Success in the cloud depends on making informed decisions about where and how to deploy workloads, establishing proper governance through landing zones and guardrails, and continuously optimizing for cost and performance.

This chapter provided the foundation for cloud architecture decisions. The following chapters will dive deeper into specific aspects of infrastructure architecture, starting with network and security architecture.

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 4: Infrastructure Architecture](/InfrastructureAndPlatformManagementHandbook/chapters/04-infrastructure-architecture/) | [Chapter 6: Network and Security Architecture](/InfrastructureAndPlatformManagementHandbook/chapters/06-network-security/) |
