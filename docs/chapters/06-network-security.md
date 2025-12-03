---
layout: default
title: "Chapter 6: Network and Security Architecture"
parent: "Part II: Architecture and Design"
nav_order: 6
permalink: /chapters/06-network-security/
---

# Chapter 6: Network and Security Architecture

## Learning Objectives

After completing this chapter, you will be able to:
- Design secure network architectures with appropriate segmentation
- Implement defense-in-depth security strategies
- Apply Zero Trust architecture principles
- Configure network security controls effectively
- Design encryption strategies for data protection
- Implement identity and access management best practices
- Design network connectivity for hybrid and multi-cloud environments

---

## Introduction

Network and security architecture form the protective foundation of infrastructure. A well-designed network enables connectivity while limiting blast radius when incidents occur. Security architecture ensures that infrastructure protects organizational assets against evolving threats.

This chapter covers network design principles, security architecture patterns, and the controls necessary to protect modern infrastructure environments.

---

## Network Architecture Fundamentals

### Network Design Principles

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Segmentation** | Divide networks to contain threats | VLANs, subnets, security zones |
| **Least Connectivity** | Only allow necessary connections | Default deny, explicit allow |
| **Defense in Depth** | Multiple security layers | Perimeter, network, host controls |
| **Visibility** | Monitor all network traffic | Flow logs, packet capture, SIEM |
| **Simplicity** | Minimize complexity | Standard patterns, documentation |
| **Resilience** | No single points of failure | Redundant paths, failover |

### Network Segmentation

Divide networks into segments to contain threats and control traffic flow:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ENTERPRISE NETWORK SEGMENTATION                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                              INTERNET                                        │
│                                  │                                           │
│                                  ▼                                           │
│                    ┌─────────────────────────┐                              │
│                    │     Edge Firewall       │                              │
│                    │   WAF │ DDoS │ IPS      │                              │
│                    └───────────┬─────────────┘                              │
│                                │                                             │
│  ┌─────────────────────────────┼─────────────────────────────────┐          │
│  │                         DMZ ZONE                               │          │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │          │
│  │  │ Load        │  │ Reverse     │  │ Bastion     │            │          │
│  │  │ Balancers   │  │ Proxy       │  │ Hosts       │            │          │
│  │  └─────────────┘  └─────────────┘  └─────────────┘            │          │
│  └─────────────────────────────┬─────────────────────────────────┘          │
│                                │                                             │
│                    ┌───────────┴───────────┐                                │
│                    │   Internal Firewall    │                                │
│                    └───────────┬───────────┘                                │
│                                │                                             │
│  ┌────────────────┬────────────┼────────────┬─────────────────┐             │
│  │                │            │            │                 │             │
│  ▼                ▼            ▼            ▼                 ▼             │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐       │
│ │   WEB    │ │   APP    │ │   DATA   │ │   MGMT   │ │   SHARED     │       │
│ │   ZONE   │ │   ZONE   │ │   ZONE   │ │   ZONE   │ │   SERVICES   │       │
│ ├──────────┤ ├──────────┤ ├──────────┤ ├──────────┤ ├──────────────┤       │
│ │ • Web    │ │ • App    │ │ • DBs    │ │ • Jump   │ │ • DNS        │       │
│ │   Servers│ │   Servers│ │ • Cache  │ │   Boxes  │ │ • Directory  │       │
│ │ • CDN    │ │ • APIs   │ │ • Storage│ │ • SIEM   │ │ • LDAP       │       │
│ │   Origin │ │ • Workers│ │ • Backup │ │ • Logging│ │ • NTP        │       │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Zone-Based Architecture

| Zone | Purpose | Access Rules | Examples |
|------|---------|--------------|----------|
| **Public/Edge** | Internet-facing entry points | Restricted inbound, monitored | CDN, WAF, DDoS protection |
| **DMZ** | Buffer between public and private | Controlled bidirectional | Load balancers, reverse proxies |
| **Application** | Application workloads | Internal access only | App servers, APIs, microservices |
| **Data** | Databases and storage | Highly restricted | Databases, file storage, caches |
| **Management** | Administrative and monitoring | Privileged access only | Jump boxes, SIEM, monitoring |
| **Shared Services** | Common infrastructure | Controlled from all zones | DNS, directory, NTP |

### VPC/Virtual Network Design

| Component | Purpose | Best Practices |
|-----------|---------|----------------|
| **CIDR Planning** | IP address allocation | Plan for growth, avoid overlaps with on-premises |
| **Subnets** | Network segments | Public and private subnets per AZ |
| **Route Tables** | Traffic routing | Explicit routes, default deny |
| **NAT Gateway** | Outbound internet for private subnets | Highly available, multi-AZ |
| **Internet Gateway** | Inbound internet access | Attach to VPC, use with public subnets |
| **VPC Peering** | Connect VPCs | Non-overlapping CIDRs, transitive routing considerations |
| **Transit Gateway** | Hub-and-spoke connectivity | Centralize routing, simplify architecture |

**VPC Architecture Example**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         VPC ARCHITECTURE                                     │
│                         CIDR: 10.0.0.0/16                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │                    AVAILABILITY ZONE A                              │     │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │     │
│  │  │ Public Subnet    │  │ Private Subnet   │  │ Data Subnet      │  │     │
│  │  │ 10.0.0.0/24      │  │ 10.0.10.0/24     │  │ 10.0.20.0/24     │  │     │
│  │  │ • NAT Gateway    │  │ • App Servers    │  │ • RDS Primary    │  │     │
│  │  │ • Bastion Host   │  │ • EKS Nodes      │  │ • ElastiCache    │  │     │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────┘  │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │                    AVAILABILITY ZONE B                              │     │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │     │
│  │  │ Public Subnet    │  │ Private Subnet   │  │ Data Subnet      │  │     │
│  │  │ 10.0.1.0/24      │  │ 10.0.11.0/24     │  │ 10.0.21.0/24     │  │     │
│  │  │ • NAT Gateway    │  │ • App Servers    │  │ • RDS Standby    │  │     │
│  │  │ • ALB Nodes      │  │ • EKS Nodes      │  │ • ElastiCache    │  │     │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────┘  │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │                    AVAILABILITY ZONE C                              │     │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐  │     │
│  │  │ Public Subnet    │  │ Private Subnet   │  │ Data Subnet      │  │     │
│  │  │ 10.0.2.0/24      │  │ 10.0.12.0/24     │  │ 10.0.22.0/24     │  │     │
│  │  │ • NAT Gateway    │  │ • App Servers    │  │ • RDS Read       │  │     │
│  │  │ • ALB Nodes      │  │ • EKS Nodes      │  │   Replica        │  │     │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────┘  │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Security Architecture

### Defense in Depth

Multiple layers of security controls protect against different attack vectors:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DEFENSE IN DEPTH LAYERS                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ LAYER 1: PERIMETER                                                   │    │
│  │ • DDoS Protection (Shield, Cloudflare)                               │    │
│  │ • Web Application Firewall (WAF)                                     │    │
│  │ • CDN with security features                                         │    │
│  │ • Rate limiting, bot protection                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ LAYER 2: NETWORK                                                     │    │
│  │ • Segmentation (VPCs, subnets, security groups)                      │    │
│  │ • Network ACLs                                                       │    │
│  │ • Network firewall/IDS/IPS                                           │    │
│  │ • VPN, Private Link, Direct Connect                                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ LAYER 3: IDENTITY                                                    │    │
│  │ • IAM policies (least privilege)                                     │    │
│  │ • Multi-factor authentication                                        │    │
│  │ • SSO and federation                                                 │    │
│  │ • Privileged access management                                       │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ LAYER 4: COMPUTE                                                     │    │
│  │ • Hardened AMIs/images                                               │    │
│  │ • Patch management                                                   │    │
│  │ • Endpoint protection                                                │    │
│  │ • Vulnerability scanning                                             │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ LAYER 5: APPLICATION                                                 │    │
│  │ • Secure coding practices                                            │    │
│  │ • Input validation                                                   │    │
│  │ • Authentication and authorization                                   │    │
│  │ • API security                                                       │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ LAYER 6: DATA                                                        │    │
│  │ • Encryption at rest and in transit                                  │    │
│  │ • Data classification                                                │    │
│  │ • Access controls                                                    │    │
│  │ • Data loss prevention                                               │    │
│  │ • Backup and recovery                                                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Zero Trust Architecture

**Principles**:

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Never Trust, Always Verify** | No implicit trust based on network location | Authenticate all access requests |
| **Least Privilege Access** | Minimum necessary permissions | Role-based access, just-in-time access |
| **Assume Breach** | Design for compromise containment | Micro-segmentation, blast radius limits |
| **Explicit Verification** | Verify user, device, and context | MFA, device compliance, conditional access |
| **Continuous Validation** | Ongoing authentication and authorization | Session management, anomaly detection |

**Zero Trust Architecture Components**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ZERO TRUST ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    POLICY DECISION POINT                             │    │
│  │  • Identity verification    • Device compliance                      │    │
│  │  • Context evaluation       • Policy enforcement                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                   │                                          │
│                                   ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    POLICY ENFORCEMENT POINT                          │    │
│  │  • API Gateway              • Service Mesh                           │    │
│  │  • Reverse Proxy            • Identity-Aware Proxy                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                   │                                          │
│       ┌───────────────┬──────────┴──────────┬───────────────┐               │
│       │               │                      │               │               │
│       ▼               ▼                      ▼               ▼               │
│  ┌─────────┐    ┌─────────┐           ┌─────────┐    ┌─────────┐           │
│  │ Service │    │ Service │           │ Service │    │  Data   │           │
│  │    A    │    │    B    │           │    C    │    │  Store  │           │
│  └─────────┘    └─────────┘           └─────────┘    └─────────┘           │
│                                                                              │
│  Supporting Components:                                                      │
│  • Identity Provider (IdP)     • Threat Intelligence                        │
│  • Device Management           • Security Analytics                         │
│  • Encryption Services         • Audit Logging                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Network Security Controls

### Security Group vs NACL Comparison

| Aspect | Security Groups | Network ACLs |
|--------|-----------------|--------------|
| **Scope** | Instance/ENI level | Subnet level |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow and deny |
| **Processing** | All rules evaluated | Rules processed in order |
| **Default** | Deny all inbound, allow all outbound | Allow all |
| **Use Case** | Fine-grained instance control | Subnet-level blocking |

### Security Control Implementation

| Control | Layer | Purpose | Examples |
|---------|-------|---------|----------|
| **WAF** | Application | Protect web apps | AWS WAF, Cloudflare, Azure WAF |
| **Firewall** | Network | Control traffic flow | AWS Network Firewall, Palo Alto |
| **Security Groups** | Instance | Instance-level firewall | Cloud-native SGs |
| **NACLs** | Subnet | Subnet-level controls | Stateless allow/deny |
| **IDS/IPS** | Network | Intrusion detection/prevention | Suricata, Snort |
| **DLP** | Data | Prevent data leakage | Symantec, Digital Guardian |

### Firewall Rules Best Practices

| Practice | Description |
|----------|-------------|
| **Default Deny** | Block all traffic by default, explicitly allow |
| **Least Privilege** | Only allow necessary ports and protocols |
| **Specific Sources** | Avoid 0.0.0.0/0 except for public services |
| **Documentation** | Comment rules with purpose and ticket number |
| **Regular Review** | Audit rules quarterly, remove unused |
| **Logging** | Log denied traffic for analysis |

---

## Encryption Standards

### Data at Rest Encryption

| Data Type | Encryption Method | Key Management |
|-----------|------------------|----------------|
| **Storage Volumes** | AES-256 | Provider or customer-managed keys |
| **Databases** | TDE (Transparent Data Encryption) | Database-native or KMS |
| **Object Storage** | Server-side encryption | SSE-S3, SSE-KMS, SSE-C |
| **Backups** | Encrypted at source | Inherit from source or separate key |
| **Archives** | Long-term encrypted storage | Key retention planning |

### Data in Transit Encryption

| Connection Type | Encryption Standard | Minimum Version |
|-----------------|---------------------|-----------------|
| **External Web** | TLS | 1.2 (prefer 1.3) |
| **Internal Services** | TLS/mTLS | 1.2 |
| **Database Connections** | TLS | 1.2 |
| **VPN** | IPSec | IKEv2 |
| **Direct Connect** | MACsec or VPN overlay | Layer 2/3 encryption |

### Key Management Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        KEY MANAGEMENT ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    KEY MANAGEMENT SERVICE (KMS)                      │    │
│  │  • Key creation and storage    • Key rotation                        │    │
│  │  • Access policies             • Audit logging                       │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                   │                                          │
│           ┌───────────────────────┼───────────────────────┐                 │
│           │                       │                       │                 │
│           ▼                       ▼                       ▼                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │  Customer       │    │   Service       │    │    HSM          │         │
│  │  Managed Keys   │    │   Keys          │    │    Keys         │         │
│  │  (CMK)          │    │   (AWS Managed) │    │    (CloudHSM)   │         │
│  └────────┬────────┘    └────────┬────────┘    └────────┬────────┘         │
│           │                      │                      │                   │
│           └──────────────────────┴──────────────────────┘                   │
│                                  │                                          │
│                                  ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    DATA ENCRYPTION KEYS (DEK)                        │    │
│  │  Generated per-resource, encrypted with CMK (envelope encryption)    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Identity and Access Management

### IAM Principles

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Least Privilege** | Minimum necessary permissions | Start with no permissions, add as needed |
| **Separation of Duties** | Prevent single point of control | Multiple approvals for critical actions |
| **Just-in-Time Access** | Temporary elevated permissions | Time-limited access, approval workflow |
| **Regular Review** | Periodic access certification | Quarterly access reviews |
| **Strong Authentication** | Multi-factor authentication | MFA required for all access |

### IAM Architecture

| Component | Purpose | Examples |
|-----------|---------|----------|
| **Identity Provider** | User authentication | Okta, Azure AD, Google Workspace |
| **Federation** | Connect identity systems | SAML, OIDC |
| **IAM Service** | Cloud resource authorization | AWS IAM, Azure RBAC, GCP IAM |
| **PAM** | Privileged access management | CyberArk, HashiCorp Vault |
| **SSO** | Single sign-on | Centralized authentication |

### Role-Based Access Control (RBAC)

| Role Type | Scope | Use Case |
|-----------|-------|----------|
| **Organization Roles** | Across organization | Billing admin, security auditor |
| **Account Roles** | Single account | Account admin, developer |
| **Service Roles** | Specific service | EC2 admin, S3 read-only |
| **Resource Roles** | Specific resources | Database admin for specific DB |

---

## Hybrid and Multi-Cloud Connectivity

### Connectivity Options

| Option | Bandwidth | Latency | Security | Use Case |
|--------|-----------|---------|----------|----------|
| **Site-to-Site VPN** | Up to 1.25 Gbps | Variable | Encrypted tunnel | Development, backup |
| **Direct Connect** | 1-100 Gbps | Low, consistent | Private connection | Production workloads |
| **ExpressRoute** | 50 Mbps - 10 Gbps | Low | Private connection | Azure connectivity |
| **Cloud Interconnect** | 10-100 Gbps | Low | Private connection | GCP connectivity |
| **SD-WAN** | Variable | Optimized | Encrypted overlay | Multiple locations |

### Hub-and-Spoke Network Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        HUB-AND-SPOKE ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                    ┌─────────────────────────────────┐                      │
│                    │           HUB VPC               │                      │
│                    │  ┌─────────────────────────┐    │                      │
│                    │  │    Transit Gateway      │    │                      │
│                    │  └───────────┬─────────────┘    │                      │
│                    │              │                   │                      │
│                    │  ┌───────────┴─────────────┐    │                      │
│                    │  │    Shared Services      │    │                      │
│                    │  │  • Firewall             │    │                      │
│                    │  │  • DNS                  │    │                      │
│                    │  │  • NAT                  │    │                      │
│                    │  │  • VPN Endpoint         │    │                      │
│                    │  └─────────────────────────┘    │                      │
│                    └───────────────┬─────────────────┘                      │
│                                    │                                         │
│         ┌──────────────┬───────────┼───────────┬──────────────┐             │
│         │              │           │           │              │             │
│         ▼              ▼           │           ▼              ▼             │
│  ┌────────────┐ ┌────────────┐    │    ┌────────────┐ ┌────────────┐       │
│  │ Production │ │    Dev     │    │    │  Staging   │ │  Security  │       │
│  │    VPC     │ │    VPC     │    │    │    VPC     │ │    VPC     │       │
│  └────────────┘ └────────────┘    │    └────────────┘ └────────────┘       │
│                                   │                                         │
│                                   │                                         │
│                    ┌──────────────┴──────────────┐                          │
│                    │     On-Premises Network     │                          │
│                    │  via Direct Connect / VPN   │                          │
│                    └─────────────────────────────┘                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Security Monitoring and Detection

### Security Monitoring Components

| Component | Purpose | Examples |
|-----------|---------|----------|
| **SIEM** | Security event correlation | Splunk, Elastic, Sentinel |
| **Flow Logs** | Network traffic analysis | VPC Flow Logs, NSG Flow Logs |
| **CloudTrail/Activity Logs** | API activity logging | AWS CloudTrail, Azure Activity Log |
| **Threat Detection** | Automated threat identification | GuardDuty, Defender |
| **Vulnerability Scanning** | Identify vulnerabilities | Inspector, Qualys, Tenable |

### Security Logging Requirements

| Log Type | Retention | Purpose |
|----------|-----------|---------|
| **Authentication Logs** | 2+ years | Access audit, incident investigation |
| **API/CloudTrail Logs** | 1+ year | Change tracking, compliance |
| **Flow Logs** | 90 days | Network forensics |
| **Application Logs** | 90 days | Application security events |
| **Security Tool Logs** | 1+ year | Threat detection history |

---

## Compliance and Standards

### Security Frameworks

| Framework | Focus | Requirements |
|-----------|-------|--------------|
| **CIS Benchmarks** | Hardening | Configuration baselines |
| **NIST 800-53** | Federal | Comprehensive controls |
| **ISO 27001** | Information Security | Management system |
| **SOC 2** | Service Organizations | Trust principles |
| **PCI DSS** | Payment Cards | Card data protection |
| **HIPAA** | Healthcare | PHI protection |

### Compliance Controls Mapping

| Control Area | CIS | NIST | PCI | Implementation |
|--------------|-----|------|-----|----------------|
| **Access Control** | 5.x | AC-x | 7.x, 8.x | IAM, MFA, RBAC |
| **Encryption** | 3.x | SC-x | 3.4, 4.x | TLS, KMS, encryption |
| **Logging** | 6.x | AU-x | 10.x | CloudTrail, SIEM |
| **Network** | 4.x | SC-x | 1.x | Segmentation, firewalls |
| **Vulnerability** | 7.x | RA-5 | 6.x, 11.x | Scanning, patching |

---

## Review Questions

1. **Network Segmentation**: Design a network segmentation strategy for a company with web applications, internal applications, databases, and a management network. What zones would you create and what traffic would you allow between them?

2. **Zero Trust**: A company wants to implement Zero Trust. They currently have a perimeter-based security model with VPN access. What steps would you recommend to transition to Zero Trust?

3. **Encryption Strategy**: Design an encryption strategy for a healthcare application that stores PHI. What data would you encrypt, what encryption methods would you use, and how would you manage keys?

4. **Security Group Design**: You have a three-tier web application (web, app, database). Design the security group rules for each tier. What ports would you allow, and from which sources?

5. **Hybrid Connectivity**: A company needs to connect their on-premises data center to AWS with consistent, low-latency connectivity for database replication. What connectivity option would you recommend?

6. **Incident Detection**: How would you design a security monitoring architecture to detect a compromised EC2 instance that is exfiltrating data?

---

## Key Takeaways

- **Network segmentation** limits blast radius and enables granular access control
- **Defense in depth** provides multiple security layers—no single point of failure
- **Zero Trust** assumes no implicit trust, verifying every access request
- **Encryption** protects data at rest and in transit—use TLS 1.2+ minimum
- **Key management** is critical—use KMS with proper access controls
- **IAM** follows least privilege—start with no permissions, add as needed
- **Monitoring and detection** enable rapid response to security incidents

---

## Summary

Network and security architecture form the foundation of infrastructure protection. Effective security requires defense in depth with multiple layers of controls, Zero Trust principles that verify every access request, proper encryption for data protection, and comprehensive monitoring for threat detection.

The patterns and controls in this chapter apply across deployment models and cloud providers. The key is applying them consistently and comprehensively, following the principle that security is everyone's responsibility.

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 5: Cloud Architecture](/InfrastructureAndPlatformManagementHandbook/chapters/05-cloud-architecture/) | [Chapter 7: High Availability and Disaster Recovery](/InfrastructureAndPlatformManagementHandbook/chapters/07-high-availability/) |
