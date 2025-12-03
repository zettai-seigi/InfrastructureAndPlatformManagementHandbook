---
layout: default
title: "Chapter 7: High Availability and Disaster Recovery"
parent: "Part II: Architecture and Design"
nav_order: 7
permalink: /chapters/07-high-availability/
---

# Chapter 7: High Availability and Disaster Recovery

## Learning Objectives

After completing this chapter, you will be able to:
- Design infrastructure for high availability across multiple dimensions
- Define RTO and RPO requirements based on business needs
- Implement disaster recovery strategies appropriate for different workloads
- Select appropriate backup and replication approaches
- Test and validate recovery capabilities
- Calculate availability targets and design systems to meet them
- Implement resilience patterns for distributed systems

---

## Introduction

High availability (HA) ensures systems remain operational despite component failures. Disaster recovery (DR) enables restoration after catastrophic events. Together, they protect business operations from infrastructure failures, ensuring that critical services remain accessible and data remains safe.

This chapter covers the principles, patterns, and practices for building resilient infrastructure that can withstand failures at multiple levels—from individual components to entire regions.

---

## High Availability Fundamentals

### Availability Concepts

| Term | Definition | Example |
|------|------------|---------|
| **Availability** | Percentage of time a system is operational | 99.9% = 8.76 hours downtime/year |
| **Uptime** | Time system is functioning correctly | System running without errors |
| **Downtime** | Time system is not available | Planned maintenance, outages |
| **SPOF** | Single Point of Failure—component whose failure causes system failure | Single database server |
| **Redundancy** | Duplicate components to eliminate SPOFs | Multiple database replicas |
| **Failover** | Automatic switch to backup component | Database failover to standby |
| **Failback** | Return to primary component after recovery | Restore primary database |

### Availability Calculation

```
Availability = MTBF / (MTBF + MTTR)

Where:
MTBF = Mean Time Between Failures
MTTR = Mean Time To Recovery
```

**Availability Targets and Downtime**:

| Availability | Downtime/Year | Downtime/Month | Downtime/Week | Typical Use |
|--------------|---------------|----------------|---------------|-------------|
| 99% (two 9s) | 3.65 days | 7.3 hours | 1.68 hours | Development, testing |
| 99.9% (three 9s) | 8.76 hours | 43.8 minutes | 10.1 minutes | Internal business apps |
| 99.95% | 4.38 hours | 21.9 minutes | 5.04 minutes | Important business apps |
| 99.99% (four 9s) | 52.6 minutes | 4.38 minutes | 1.01 minutes | Critical apps, e-commerce |
| 99.999% (five 9s) | 5.26 minutes | 26.3 seconds | 6.05 seconds | Mission critical, financial |

### Calculating Combined Availability

**Serial Components** (all must work):
```
Combined Availability = A1 × A2 × A3 × ...

Example: Web Server (99.9%) → App Server (99.9%) → Database (99.9%)
Combined = 0.999 × 0.999 × 0.999 = 99.7%
```

**Parallel Components** (one must work):
```
Combined Availability = 1 - (1 - A1) × (1 - A2) × ...

Example: Two database servers, each 99.9%
Combined = 1 - (0.001 × 0.001) = 99.9999%
```

---

## High Availability Design Patterns

### Active-Active Pattern

Both (or all) instances actively handle traffic simultaneously.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ACTIVE-ACTIVE PATTERN                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌───────────────────┐                               │
│                         │   Load Balancer   │                               │
│                         └─────────┬─────────┘                               │
│                                   │                                          │
│                    ┌──────────────┴──────────────┐                          │
│                    │                             │                          │
│                    ▼                             ▼                          │
│           ┌─────────────────┐          ┌─────────────────┐                  │
│           │   Instance A    │          │   Instance B    │                  │
│           │   (Active)      │          │   (Active)      │                  │
│           │   50% Traffic   │          │   50% Traffic   │                  │
│           └─────────────────┘          └─────────────────┘                  │
│                                                                              │
│   Characteristics:                                                           │
│   • Full capacity utilization                                                │
│   • Seamless failover (no switchover time)                                  │
│   • Requires stateless design or shared state                               │
│   • More complex to implement for databases                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Use Cases**:
- Web servers
- API servers
- Stateless applications
- Read replicas

### Active-Passive Pattern

One instance handles traffic; standby takes over on failure.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ACTIVE-PASSIVE PATTERN                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌───────────────────┐                               │
│                         │   Load Balancer   │                               │
│                         │   (Health Check)  │                               │
│                         └─────────┬─────────┘                               │
│                                   │                                          │
│                    ┌──────────────┴──────────────┐                          │
│                    │                             │                          │
│                    ▼                             ▼                          │
│           ┌─────────────────┐          ┌─────────────────┐                  │
│           │   Instance A    │          │   Instance B    │                  │
│           │   (Active)      │◄────────►│   (Standby)     │                  │
│           │   100% Traffic  │  Repl    │   0% Traffic    │                  │
│           └─────────────────┘          └─────────────────┘                  │
│                                                                              │
│   On Failure:                                                                │
│   • Health check detects primary failure                                     │
│   • Traffic automatically routes to standby                                  │
│   • Standby becomes new primary                                              │
│   • RTO depends on detection and switchover time                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Use Cases**:
- Databases
- Stateful applications
- License-limited software
- Cost optimization (standby can be smaller)

### Multi-AZ Deployment

Deploy across multiple Availability Zones within a region.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        MULTI-AZ DEPLOYMENT                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                    REGION: US-EAST-1                                         │
│                                                                              │
│                         ┌───────────────────┐                               │
│                         │       ALB         │                               │
│                         │  (Cross-AZ LB)    │                               │
│                         └─────────┬─────────┘                               │
│                                   │                                          │
│         ┌─────────────────────────┼─────────────────────────┐               │
│         │                         │                         │               │
│         ▼                         ▼                         ▼               │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │      AZ-A       │    │      AZ-B       │    │      AZ-C       │         │
│  │  (us-east-1a)   │    │  (us-east-1b)   │    │  (us-east-1c)   │         │
│  ├─────────────────┤    ├─────────────────┤    ├─────────────────┤         │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │         │
│  │ │ Web Tier    │ │    │ │ Web Tier    │ │    │ │ Web Tier    │ │         │
│  │ │ (ASG)       │ │    │ │ (ASG)       │ │    │ │ (ASG)       │ │         │
│  │ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │         │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │         │
│  │ │ App Tier    │ │    │ │ App Tier    │ │    │ │ App Tier    │ │         │
│  │ │ (ASG)       │ │    │ │ (ASG)       │ │    │ │ (ASG)       │ │         │
│  │ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │         │
│  │ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │         │
│  │ │ RDS Primary │ │◄──►│ │ RDS Standby │ │    │ │ RDS Read    │ │         │
│  │ └─────────────┘ │Sync│ └─────────────┘ │    │ │ Replica     │ │         │
│  └─────────────────┘Repl└─────────────────┘    │ └─────────────┘ │         │
│                                                 └─────────────────┘         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Multi-Region Deployment

Deploy across multiple geographic regions for disaster recovery.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        MULTI-REGION DEPLOYMENT                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌───────────────────┐                               │
│                         │   Route 53 / CDN  │                               │
│                         │   (Global DNS)    │                               │
│                         └─────────┬─────────┘                               │
│                                   │                                          │
│              ┌────────────────────┼────────────────────┐                    │
│              │                    │                    │                    │
│              ▼                    ▼                    ▼                    │
│  ┌─────────────────────┐ ┌─────────────────────┐ ┌────────────────────┐    │
│  │     US-EAST-1       │ │     EU-WEST-1       │ │    AP-SOUTHEAST-1  │    │
│  │     (Primary)       │ │    (Secondary)      │ │    (Secondary)     │    │
│  ├─────────────────────┤ ├─────────────────────┤ ├────────────────────┤    │
│  │ • Full application  │ │ • Full application  │ │ • Full application │    │
│  │ • Active traffic    │ │ • DR standby or     │ │ • Regional traffic │    │
│  │ • Primary database  │ │   active traffic    │ │ • Read replica     │    │
│  │                     │ │ • Replicated data   │ │ • Replicated data  │    │
│  └─────────────────────┘ └─────────────────────┘ └────────────────────┘    │
│              │                    │                    │                    │
│              └────────────────────┴────────────────────┘                    │
│                       Database Replication                                   │
│                    (Async or Global Tables)                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Disaster Recovery

### RTO and RPO Definitions

| Metric | Definition | Question Answered |
|--------|------------|-------------------|
| **RTO (Recovery Time Objective)** | Maximum acceptable time to restore service after disaster | How long can we be down? |
| **RPO (Recovery Point Objective)** | Maximum acceptable data loss measured in time | How much data can we lose? |

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        RTO AND RPO ILLUSTRATED                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  TIMELINE:                                                                   │
│                                                                              │
│  Last Good     Disaster        Service                                       │
│  Backup        Occurs          Restored                                      │
│     │             │               │                                          │
│     ▼             ▼               ▼                                          │
│  ───●─────────────●───────────────●─────────────────────►  Time             │
│     │             │               │                                          │
│     │◄───RPO────►│               │                                          │
│     │    (Data Loss)             │                                          │
│                   │◄────RTO─────►│                                          │
│                   │   (Downtime)                                             │
│                                                                              │
│  RPO = 4 hours means we can lose up to 4 hours of data                      │
│  RTO = 2 hours means we must be back online within 2 hours                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### DR Strategy Selection

| Strategy | RTO | RPO | Cost | Infrastructure |
|----------|-----|-----|------|----------------|
| **Backup & Restore** | Hours-Days | Hours | $ | No standby infrastructure |
| **Pilot Light** | Minutes-Hours | Minutes | $$ | Minimal core running |
| **Warm Standby** | Minutes | Seconds-Minutes | $$$ | Scaled-down copy running |
| **Hot Standby / Multi-Site** | Seconds | Near-zero | $$$$ | Full capacity in DR site |

### Backup and Restore Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        BACKUP AND RESTORE                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   PRIMARY REGION                              DR REGION                      │
│   ┌───────────────────────┐                  ┌───────────────────────┐      │
│   │                       │     Backup       │                       │      │
│   │   Production          │ ──────────────►  │   S3 / Blob Storage   │      │
│   │   Infrastructure      │    (Nightly)     │   (Backups, AMIs)     │      │
│   │                       │                  │                       │      │
│   │   • EC2 Instances     │                  │   On Disaster:        │      │
│   │   • RDS Database      │                  │   • Restore from      │      │
│   │   • Application Data  │                  │     backup            │      │
│   │                       │                  │   • Launch new        │      │
│   └───────────────────────┘                  │     infrastructure    │      │
│                                              │   • Restore data      │      │
│                                              └───────────────────────┘      │
│                                                                              │
│   RPO: Hours (depends on backup frequency)                                   │
│   RTO: Hours-Days (depends on restoration time)                              │
│   Cost: $ (storage costs only)                                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Pilot Light Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        PILOT LIGHT                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   PRIMARY REGION                              DR REGION                      │
│   ┌───────────────────────┐                  ┌───────────────────────┐      │
│   │                       │                  │                       │      │
│   │   Production          │  Continuous      │   Minimal Running:    │      │
│   │   (Full Scale)        │  Replication     │                       │      │
│   │                       │ ──────────────►  │   ┌─────────────────┐ │      │
│   │   ┌─────────────────┐ │                  │   │ RDS Replica     │ │      │
│   │   │ Web (ASG: 10)   │ │                  │   │ (Running)       │ │      │
│   │   └─────────────────┘ │                  │   └─────────────────┘ │      │
│   │   ┌─────────────────┐ │                  │                       │      │
│   │   │ App (ASG: 20)   │ │                  │   ┌─────────────────┐ │      │
│   │   └─────────────────┘ │                  │   │ AMIs (Ready)    │ │      │
│   │   ┌─────────────────┐ │                  │   └─────────────────┘ │      │
│   │   │ RDS (Primary)   │ │                  │                       │      │
│   │   └─────────────────┘ │                  │   On Disaster:        │      │
│   │                       │                  │   Scale up from AMIs  │      │
│   └───────────────────────┘                  └───────────────────────┘      │
│                                                                              │
│   RPO: Minutes (continuous DB replication)                                   │
│   RTO: Minutes-Hours (time to scale up)                                      │
│   Cost: $$ (running database replica)                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Warm Standby Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        WARM STANDBY                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   PRIMARY REGION                              DR REGION                      │
│   ┌───────────────────────┐                  ┌───────────────────────┐      │
│   │                       │  Continuous      │   Scaled-Down Copy:   │      │
│   │   Production          │  Replication     │                       │      │
│   │   (Full Scale)        │ ──────────────►  │   ┌─────────────────┐ │      │
│   │                       │                  │   │ Web (ASG: 2)    │ │      │
│   │   ┌─────────────────┐ │                  │   │ (Running)       │ │      │
│   │   │ Web (ASG: 10)   │ │                  │   └─────────────────┘ │      │
│   │   └─────────────────┘ │                  │   ┌─────────────────┐ │      │
│   │   ┌─────────────────┐ │                  │   │ App (ASG: 4)    │ │      │
│   │   │ App (ASG: 20)   │ │                  │   │ (Running)       │ │      │
│   │   └─────────────────┘ │                  │   └─────────────────┘ │      │
│   │   ┌─────────────────┐ │                  │   ┌─────────────────┐ │      │
│   │   │ RDS (Primary)   │ │                  │   │ RDS Replica     │ │      │
│   │   └─────────────────┘ │                  │   │ (Running)       │ │      │
│   │                       │                  │   └─────────────────┘ │      │
│   └───────────────────────┘                  └───────────────────────┘      │
│                                                                              │
│   RPO: Seconds-Minutes (synchronous or near-sync replication)                │
│   RTO: Minutes (scale out existing infrastructure)                           │
│   Cost: $$$ (running scaled-down infrastructure)                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Hot Standby / Multi-Site Active-Active

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        HOT STANDBY / ACTIVE-ACTIVE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌───────────────────┐                               │
│                         │   Global DNS      │                               │
│                         │   (Active-Active) │                               │
│                         └─────────┬─────────┘                               │
│                                   │                                          │
│                    ┌──────────────┴──────────────┐                          │
│                    │                             │                          │
│                    ▼                             ▼                          │
│   ┌───────────────────────┐       ┌───────────────────────┐                │
│   │   PRIMARY REGION      │       │   SECONDARY REGION    │                │
│   │   (Full Scale)        │       │   (Full Scale)        │                │
│   │                       │       │                       │                │
│   │   ┌─────────────────┐ │       │   ┌─────────────────┐ │                │
│   │   │ Web (ASG: 10)   │ │       │   │ Web (ASG: 10)   │ │                │
│   │   └─────────────────┘ │       │   └─────────────────┘ │                │
│   │   ┌─────────────────┐ │       │   ┌─────────────────┐ │                │
│   │   │ App (ASG: 20)   │ │       │   │ App (ASG: 20)   │ │                │
│   │   └─────────────────┘ │◄─────►│   └─────────────────┘ │                │
│   │   ┌─────────────────┐ │ Bi-   │   ┌─────────────────┐ │                │
│   │   │ Database        │ │ Dir   │   │ Database        │ │                │
│   │   │ (Active)        │ │ Repl  │   │ (Active)        │ │                │
│   │   └─────────────────┘ │       │   └─────────────────┘ │                │
│   └───────────────────────┘       └───────────────────────┘                │
│                                                                              │
│   RPO: Near-zero (synchronous replication)                                   │
│   RTO: Seconds (automatic failover)                                          │
│   Cost: $$$$ (full duplicate infrastructure)                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Backup Strategies

### Backup Types

| Type | Description | Frequency | Storage | Use Case |
|------|-------------|-----------|---------|----------|
| **Full Backup** | Complete copy of all data | Weekly | High | Baseline for recovery |
| **Incremental** | Changes since last backup (any type) | Daily | Low | Frequent backup |
| **Differential** | Changes since last full backup | Daily | Medium | Balance of speed and space |
| **Snapshot** | Point-in-time volume/database capture | Hourly-Daily | Varies | Quick recovery |
| **Continuous** | Transaction log shipping | Real-time | Low | Minimal data loss |

### 3-2-1 Backup Rule

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        3-2-1 BACKUP RULE                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   3 COPIES of data:                                                          │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │
│   │   Original   │  │   Backup 1   │  │   Backup 2   │                      │
│   │   Data       │  │   (Local)    │  │   (Offsite)  │                      │
│   └──────────────┘  └──────────────┘  └──────────────┘                      │
│                                                                              │
│   2 DIFFERENT storage types:                                                 │
│   ┌──────────────┐  ┌──────────────┐                                        │
│   │   Primary    │  │   Backup     │                                        │
│   │   Storage    │  │   Storage    │                                        │
│   │   (SSD/EBS)  │  │   (S3/Tape)  │                                        │
│   └──────────────┘  └──────────────┘                                        │
│                                                                              │
│   1 COPY offsite/cloud:                                                      │
│   ┌──────────────┐                                                          │
│   │   Cloud      │                                                          │
│   │   Storage    │                                                          │
│   │   (S3 Cross- │                                                          │
│   │   Region)    │                                                          │
│   └──────────────┘                                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Backup Retention Policies

| Data Type | Frequency | Retention | Storage Tier |
|-----------|-----------|-----------|--------------|
| **Critical Database** | Hourly snapshots | 7 days | Standard |
| **Critical Database** | Daily backups | 30 days | Standard |
| **Critical Database** | Weekly backups | 1 year | Infrequent Access |
| **Critical Database** | Monthly backups | 7 years | Archive |
| **Application Data** | Daily backups | 30 days | Standard |
| **Configuration** | On change | 90 days | Standard |
| **Logs** | Continuous | 90 days-1 year | Infrequent Access |

---

## Data Replication

### Replication Modes

| Mode | Description | RPO | Latency Impact |
|------|-------------|-----|----------------|
| **Synchronous** | Write confirmed after replica acknowledges | Zero | Higher (wait for replica) |
| **Asynchronous** | Write confirmed immediately, replica follows | Minutes | None |
| **Semi-synchronous** | Write confirmed after one replica acknowledges | Low | Moderate |

### Database Replication Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Primary-Standby** | One primary, one standby for failover | High availability |
| **Primary-Read Replicas** | One primary, multiple read replicas | Read scaling |
| **Multi-Master** | Multiple writable nodes | Active-active across regions |
| **Streaming Replication** | Continuous WAL/log shipping | Low RPO |

---

## Testing and Validation

### DR Test Types

| Test Type | Description | Impact | Frequency |
|-----------|-------------|--------|-----------|
| **Walkthrough** | Review procedures on paper | None | Quarterly |
| **Tabletop Exercise** | Simulate scenario with team discussion | None | Semi-annually |
| **Simulation** | Simulate failure without actual failover | Minimal | Quarterly |
| **Parallel Test** | Bring up DR while production runs | Low | Semi-annually |
| **Full Interruption** | Actual failover to DR | High | Annually |

### Testing Checklist

| Area | Test Item | Success Criteria |
|------|-----------|------------------|
| **Backup** | Verify backup completion | All systems backed up successfully |
| **Restore** | Restore from backup | Data restored within RTO |
| **Failover** | Trigger database failover | Failover completes within target time |
| **DNS** | Verify DNS failover | DNS propagates within expected time |
| **Application** | Validate application function | All features work in DR |
| **Connectivity** | Test network connectivity | All connections established |
| **Performance** | Validate performance in DR | Meets performance SLAs |
| **Data Integrity** | Verify data consistency | No data loss beyond RPO |

---

## Resilience Patterns

### Circuit Breaker Pattern

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CIRCUIT BREAKER PATTERN                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                    ┌───────────────────┐                                    │
│                    │      CLOSED       │                                    │
│                    │  (Normal flow)    │                                    │
│                    └─────────┬─────────┘                                    │
│                              │                                               │
│                    Failures exceed                                           │
│                      threshold                                               │
│                              │                                               │
│                              ▼                                               │
│                    ┌───────────────────┐                                    │
│                    │       OPEN        │                                    │
│                    │ (Fast fail, no    │                                    │
│                    │  requests sent)   │                                    │
│                    └─────────┬─────────┘                                    │
│                              │                                               │
│                    After timeout                                             │
│                    period expires                                            │
│                              │                                               │
│                              ▼                                               │
│                    ┌───────────────────┐                                    │
│                    │    HALF-OPEN      │                                    │
│                    │ (Test requests)   │───Success───► CLOSED               │
│                    │                   │───Failure───► OPEN                 │
│                    └───────────────────┘                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Retry with Exponential Backoff

| Attempt | Wait Time | Total Time |
|---------|-----------|------------|
| 1 | 0 | 0 |
| 2 | 1 second | 1 second |
| 3 | 2 seconds | 3 seconds |
| 4 | 4 seconds | 7 seconds |
| 5 | 8 seconds | 15 seconds |
| 6 | 16 seconds | 31 seconds |

### Bulkhead Pattern

Isolate failures to prevent cascade:

| Resource | Bulkhead Implementation |
|----------|------------------------|
| Thread pools | Separate pools per dependency |
| Connection pools | Separate pools per service |
| Instances | Separate instance groups |
| Availability zones | Distribute across AZs |

---

## Review Questions

1. **Availability Calculation**: A system has three components in series: Component A (99.9%), Component B (99.9%), and Component C (99.95%). What is the overall system availability? How would adding redundancy to Component A change this?

2. **RTO/RPO Selection**: A financial trading application requires no more than 5 minutes of data loss and must be restored within 15 minutes. Which DR strategy would you recommend and why?

3. **Multi-AZ Design**: Design a highly available architecture for a web application with the following requirements: 99.95% availability, handle AZ failure gracefully, database with multi-AZ failover.

4. **Backup Strategy**: Design a backup strategy for a healthcare application with 7-year retention requirements for patient data. Include backup types, frequencies, and storage tiers.

5. **DR Testing**: Your organization has never tested DR. Propose a phased testing approach that minimizes risk while validating DR capabilities.

6. **Resilience Patterns**: A microservice is experiencing intermittent failures from a downstream dependency. Which resilience patterns would you implement and why?

---

## Key Takeaways

- **High availability** eliminates single points of failure through redundancy
- **Availability targets** should be based on business requirements, not technical aspirations
- **RTO and RPO** define recovery requirements and drive DR strategy selection
- **DR strategies** range from backup/restore ($) to multi-site active-active ($$$$)
- **3-2-1 backup rule** ensures data survives multiple failure scenarios
- **Regular testing** validates that DR actually works when needed
- **Resilience patterns** (circuit breakers, retries, bulkheads) protect against cascading failures

---

## Summary

High availability and disaster recovery are foundational to reliable infrastructure. Success requires understanding business requirements (RTO/RPO), selecting appropriate strategies, implementing redundancy at multiple levels, and regularly testing recovery capabilities.

The investment in HA/DR should be proportional to the business impact of downtime. Critical systems warrant higher investment in redundancy and faster recovery capabilities, while less critical systems can accept longer recovery times and higher data loss.

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 6: Network and Security](/InfrastructureAndPlatformManagementHandbook/chapters/06-network-security/) | [Chapter 8: Infrastructure as Code](/InfrastructureAndPlatformManagementHandbook/chapters/08-infrastructure-as-code/) |
