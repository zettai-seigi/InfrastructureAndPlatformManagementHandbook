---
layout: default
title: "Chapter 9: Container Platforms and Orchestration"
parent: "Part III: Build and Deployment"
nav_order: 9
permalink: /chapters/09-container-platforms/
---

# Chapter 9: Container Platforms and Orchestration

## Learning Objectives

After completing this chapter, you will be able to:
- Understand container fundamentals and their benefits for infrastructure
- Design Kubernetes cluster architectures for production workloads
- Implement container platform best practices
- Configure container networking, storage, and security
- Select appropriate container platforms for different use cases
- Operate and troubleshoot containerized environments

---

## Introduction

Containers have revolutionized how applications are packaged, deployed, and managed. By encapsulating applications with their dependencies, containers enable consistency across environments, efficient resource utilization, and rapid deployment. Kubernetes has emerged as the standard platform for orchestrating containers at scale.

This chapter covers container fundamentals, Kubernetes architecture, and best practices for running containerized workloads in production.

---

## Container Fundamentals

### What Are Containers?

Containers are lightweight, standalone packages that include everything needed to run an application: code, runtime, system tools, libraries, and settings.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VMs vs CONTAINERS                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   VIRTUAL MACHINES                          CONTAINERS                       │
│   ┌───────────────────────────┐            ┌───────────────────────────┐    │
│   │    App A    │    App B    │            │  App A  │  App B  │ App C │    │
│   ├─────────────┼─────────────┤            ├─────────┼─────────┼───────┤    │
│   │   Guest OS  │   Guest OS  │            │   Bins/Libs  │ Bins/Libs │    │
│   ├─────────────┴─────────────┤            ├───────────────────────────┤    │
│   │        Hypervisor         │            │      Container Runtime    │    │
│   ├───────────────────────────┤            ├───────────────────────────┤    │
│   │         Host OS           │            │          Host OS          │    │
│   ├───────────────────────────┤            ├───────────────────────────┤    │
│   │       Infrastructure      │            │       Infrastructure      │    │
│   └───────────────────────────┘            └───────────────────────────┘    │
│                                                                              │
│   • Full OS per VM                         • Shared OS kernel               │
│   • Minutes to start                       • Seconds to start               │
│   • GB of memory                           • MB of memory                   │
│   • Strong isolation                       • Process isolation              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Container vs. Virtual Machine

| Aspect | Container | Virtual Machine |
|--------|-----------|-----------------|
| **Isolation** | Process-level (namespaces, cgroups) | Hardware-level (hypervisor) |
| **Size** | Megabytes | Gigabytes |
| **Start Time** | Seconds | Minutes |
| **Resource Usage** | Efficient (shared kernel) | More overhead (full OS) |
| **Portability** | High (OCI standard) | Medium (hypervisor dependent) |
| **Security** | Shared kernel attack surface | Stronger isolation |
| **Density** | High (hundreds per host) | Lower (tens per host) |

### Container Benefits

| Benefit | Description | Impact |
|---------|-------------|--------|
| **Consistency** | Same container runs anywhere | Dev/prod parity, fewer bugs |
| **Portability** | Run on any container runtime | Cloud agnostic |
| **Efficiency** | Share OS kernel | Higher density, lower cost |
| **Speed** | Sub-second startup | Fast scaling, CI/CD |
| **Isolation** | Process-level separation | Multi-tenant safety |
| **Immutability** | Replace, don't patch | Predictable deployments |
| **Microservices** | Enables distributed architectures | Scalability, agility |

### Container Architecture Components

| Component | Description | Examples |
|-----------|-------------|----------|
| **Container Runtime** | Runs containers | containerd, CRI-O, Docker |
| **Container Image** | Immutable package with application | Built from Dockerfile |
| **Registry** | Stores and distributes images | ECR, GCR, Docker Hub, Harbor |
| **Orchestrator** | Manages containers at scale | Kubernetes, ECS, Nomad |

---

## Kubernetes Architecture

### Kubernetes Overview

Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    KUBERNETES ARCHITECTURE                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         CONTROL PLANE                                   │ │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐           │ │
│  │  │   API Server   │  │   Scheduler    │  │ Controller Mgr │           │ │
│  │  │   (kube-api)   │  │                │  │                │           │ │
│  │  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘           │ │
│  │          │                   │                   │                     │ │
│  │          └───────────────────┼───────────────────┘                     │ │
│  │                              │                                          │ │
│  │                   ┌──────────┴──────────┐                              │ │
│  │                   │        etcd         │                              │ │
│  │                   │  (Cluster State)    │                              │ │
│  │                   └─────────────────────┘                              │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                   │                                          │
│                                   ▼                                          │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         WORKER NODES                                    │ │
│  │  ┌─────────────────────────┐    ┌─────────────────────────┐           │ │
│  │  │        Node 1           │    │        Node 2           │           │ │
│  │  │  ┌───────────────────┐  │    │  ┌───────────────────┐  │           │ │
│  │  │  │     kubelet       │  │    │  │     kubelet       │  │           │ │
│  │  │  └───────────────────┘  │    │  └───────────────────┘  │           │ │
│  │  │  ┌───────────────────┐  │    │  ┌───────────────────┐  │           │ │
│  │  │  │    kube-proxy     │  │    │  │    kube-proxy     │  │           │ │
│  │  │  └───────────────────┘  │    │  └───────────────────┘  │           │ │
│  │  │  ┌───────────────────┐  │    │  ┌───────────────────┐  │           │ │
│  │  │  │ Container Runtime │  │    │  │ Container Runtime │  │           │ │
│  │  │  └───────────────────┘  │    │  └───────────────────┘  │           │ │
│  │  │  ┌─────┐┌─────┐┌─────┐  │    │  ┌─────┐┌─────┐┌─────┐  │           │ │
│  │  │  │ Pod ││ Pod ││ Pod │  │    │  │ Pod ││ Pod ││ Pod │  │           │ │
│  │  │  └─────┘└─────┘└─────┘  │    │  └─────┘└─────┘└─────┘  │           │ │
│  │  └─────────────────────────┘    └─────────────────────────┘           │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Control Plane Components

| Component | Purpose | Function |
|-----------|---------|----------|
| **API Server** | Frontend for cluster | All communication goes through API |
| **etcd** | Distributed key-value store | Stores all cluster state |
| **Scheduler** | Places pods on nodes | Considers resources, constraints |
| **Controller Manager** | Runs controllers | Ensures desired state |
| **Cloud Controller** | Cloud provider integration | Manages cloud resources |

### Worker Node Components

| Component | Purpose | Function |
|-----------|---------|----------|
| **kubelet** | Node agent | Ensures containers running |
| **kube-proxy** | Network proxy | Maintains network rules |
| **Container Runtime** | Runs containers | containerd, CRI-O |

---

## Kubernetes Objects

### Core Objects

| Object | Purpose | Use Case |
|--------|---------|----------|
| **Pod** | Smallest deployable unit | One or more containers |
| **Deployment** | Declarative pod management | Stateless applications |
| **StatefulSet** | Ordered, stable pod management | Databases, stateful apps |
| **DaemonSet** | Pod on every node | Monitoring agents, log collectors |
| **Job** | Run to completion | Batch processing |
| **CronJob** | Scheduled jobs | Periodic tasks |
| **ReplicaSet** | Ensures pod replicas | Managed by Deployment |

### Service Objects

| Service Type | Description | Use Case |
|--------------|-------------|----------|
| **ClusterIP** | Internal cluster IP | Service-to-service communication |
| **NodePort** | Expose on node port | Development, testing |
| **LoadBalancer** | Cloud load balancer | Production external access |
| **ExternalName** | DNS CNAME record | External service alias |
| **Headless** | No cluster IP | StatefulSets, direct pod access |

### Configuration Objects

| Object | Purpose | Use Case |
|--------|---------|----------|
| **ConfigMap** | Non-sensitive configuration | Application config |
| **Secret** | Sensitive data | Passwords, keys, tokens |
| **PersistentVolume** | Storage resource | Cluster-level storage |
| **PersistentVolumeClaim** | Storage request | Application storage request |
| **Ingress** | HTTP routing | External HTTP access |
| **NetworkPolicy** | Network traffic rules | Pod-level firewall |

### Example: Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: nginx:1.24
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

## Container Networking

### Kubernetes Networking Model

| Principle | Description |
|-----------|-------------|
| Every pod gets unique IP | No NAT between pods |
| Pods can communicate directly | Flat network space |
| Services provide stable endpoints | Abstract pod IPs |
| Network policies control traffic | Firewall rules |

### CNI Plugins

| Plugin | Features | Use Case |
|--------|----------|----------|
| **Calico** | Network policies, BGP | Enterprise, security focus |
| **Cilium** | eBPF-based, observability | Performance, security |
| **AWS VPC CNI** | Native VPC networking | AWS EKS |
| **Azure CNI** | Native Azure networking | Azure AKS |
| **Flannel** | Simple overlay | Development, simple clusters |

### Service Mesh

| Feature | Description | Tools |
|---------|-------------|-------|
| **Traffic Management** | Routing, load balancing | Istio, Linkerd |
| **Security** | mTLS, authorization | Istio, Linkerd |
| **Observability** | Metrics, traces, logs | Istio, Linkerd |
| **Policy** | Rate limiting, circuit breaking | Istio |

---

## Container Storage

### Storage Classes

| Class | Description | Use Case |
|-------|-------------|----------|
| **Block Storage** | Persistent block devices | Databases |
| **File Storage** | Shared file systems | Shared data |
| **Object Storage** | S3-compatible | Unstructured data |

### Persistent Storage Example

```yaml
# StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  fsType: ext4
volumeBindingMode: WaitForFirstConsumer

---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: database-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 100Gi
```

---

## Container Security

### Security Layers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CONTAINER SECURITY LAYERS                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ CLUSTER: RBAC, Network Policies, Pod Security Standards             │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ NODE: OS hardening, CIS benchmarks, Runtime security                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ CONTAINER: Non-root, read-only filesystem, resource limits          │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ IMAGE: Base image security, vulnerability scanning, signing         │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ CODE: Secure coding, dependency scanning, SAST                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Pod Security Standards

| Standard | Description | Use Case |
|----------|-------------|----------|
| **Privileged** | No restrictions | System/infrastructure workloads |
| **Baseline** | Minimal restrictions | Default for most workloads |
| **Restricted** | Heavily restricted | Security-sensitive workloads |

### Security Best Practices

| Practice | Implementation |
|----------|---------------|
| **Image Security** | Scan images, use minimal base, sign images |
| **Runtime Security** | Pod security standards, seccomp, AppArmor |
| **Network Security** | Network policies, service mesh |
| **Secrets Management** | External secrets operator, Vault |
| **RBAC** | Least privilege, namespace isolation |
| **Audit Logging** | Kubernetes audit logs |

---

## Platform Engineering

### Internal Developer Platform

| Component | Purpose |
|-----------|---------|
| **Self-Service Portal** | Developer infrastructure requests |
| **Templates** | Standardized deployment patterns |
| **GitOps** | Git-based deployment |
| **Service Catalog** | Available services and documentation |
| **Observability** | Built-in monitoring and logging |

### Platform Components

| Category | Tools |
|----------|-------|
| **Ingress** | NGINX, Traefik, Istio Gateway |
| **Service Mesh** | Istio, Linkerd, Consul Connect |
| **Secrets** | External Secrets, Vault, Sealed Secrets |
| **GitOps** | ArgoCD, Flux |
| **Monitoring** | Prometheus, Grafana, Datadog |
| **Logging** | Fluentd, Loki, ELK Stack |

---

## Managed Kubernetes Services

### Service Comparison

| Service | Provider | Features |
|---------|----------|----------|
| **EKS** | AWS | Deep AWS integration, Fargate, add-ons marketplace |
| **AKS** | Azure | Azure AD integration, Azure CNI |
| **GKE** | Google | Autopilot, advanced features, GKE Enterprise |

### Managed vs Self-Managed

| Factor | Managed | Self-Managed |
|--------|---------|--------------|
| **Control Plane** | Provider managed | You manage |
| **Upgrades** | Automated | Manual |
| **Cost** | Higher | Lower (but ops cost) |
| **Flexibility** | Some constraints | Full control |
| **Support** | Provider support | Community |

---

## Review Questions

1. **Container vs VM**: When would you choose containers over VMs? What security considerations apply?

2. **Kubernetes Design**: Design a Kubernetes deployment strategy for a stateless web application requiring high availability across availability zones.

3. **Networking**: Compare ClusterIP, NodePort, and LoadBalancer services. When would you use each?

4. **Security**: Design a security strategy including image scanning, runtime security, and network policies.

5. **Managed vs Self-Managed**: What factors influence your decision between managed Kubernetes and self-managed?

---

## Key Takeaways

- **Containers** provide consistency, portability, and efficiency for application deployment
- **Kubernetes** orchestrates containers at scale with declarative configuration
- **Networking** follows a flat network model with services for abstraction
- **Security** requires defense at multiple layers: image, container, pod, node, cluster
- **Platform engineering** enables developer self-service
- **Managed services** reduce operational burden but have trade-offs

---

## Summary

Container platforms and Kubernetes have become essential infrastructure components for modern applications. Understanding container fundamentals, Kubernetes architecture, and operational best practices enables organizations to leverage the full benefits of containerization while managing complexity and maintaining security.

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 8: Infrastructure as Code](/InfrastructureAndPlatformManagementHandbook/chapters/08-infrastructure-as-code/) | [Chapter 10: Deployment Automation](/InfrastructureAndPlatformManagementHandbook/chapters/10-deployment-automation/) |
