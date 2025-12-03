---
layout: default
title: "Chapter 8: Infrastructure as Code"
parent: "Part III: Build and Deployment"
nav_order: 8
permalink: /chapters/08-infrastructure-as-code/
---

# Chapter 8: Infrastructure as Code

## Learning Objectives

After completing this chapter, you will be able to:
- Implement Infrastructure as Code principles and practices
- Select appropriate IaC tools for different use cases
- Develop reusable infrastructure modules
- Manage infrastructure state effectively
- Test and validate infrastructure code
- Implement IaC pipelines for automated deployment
- Apply best practices for IaC development and operations

---

## Introduction

Infrastructure as Code (IaC) transforms infrastructure management from manual processes to software engineering practices. By defining infrastructure in code, organizations achieve consistency, speed, and reliability in infrastructure operations. IaC is foundational to modern infrastructure management—without it, achieving the scale, consistency, and agility required by today's businesses is virtually impossible.

This chapter covers IaC principles, tools, patterns, and best practices for implementing infrastructure as code across cloud and on-premises environments.

---

## IaC Fundamentals

### What is Infrastructure as Code?

Infrastructure as Code is the practice of managing and provisioning infrastructure through machine-readable definition files rather than manual processes or interactive configuration tools.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    TRADITIONAL vs IaC APPROACH                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   TRADITIONAL                          INFRASTRUCTURE AS CODE                │
│   ┌───────────────────┐               ┌───────────────────┐                 │
│   │                   │               │                   │                 │
│   │  Manual Console   │               │   Code Files      │                 │
│   │  Click-through    │               │   (HCL, YAML)     │                 │
│   │                   │               │                   │                 │
│   └─────────┬─────────┘               └─────────┬─────────┘                 │
│             │                                   │                            │
│             ▼                                   ▼                            │
│   ┌───────────────────┐               ┌───────────────────┐                 │
│   │                   │               │                   │                 │
│   │  Documentation    │               │   Version Control │                 │
│   │  (Often outdated) │               │   (Git)           │                 │
│   │                   │               │                   │                 │
│   └─────────┬─────────┘               └─────────┬─────────┘                 │
│             │                                   │                            │
│             ▼                                   ▼                            │
│   ┌───────────────────┐               ┌───────────────────┐                 │
│   │                   │               │                   │                 │
│   │  Environment      │               │   CI/CD Pipeline  │                 │
│   │  Drift            │               │   (Automated)     │                 │
│   │                   │               │                   │                 │
│   └───────────────────┘               └─────────┬─────────┘                 │
│                                                 │                            │
│                                                 ▼                            │
│                                       ┌───────────────────┐                 │
│                                       │   Consistent      │                 │
│                                       │   Infrastructure  │                 │
│                                       └───────────────────┘                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core Principles

| Principle | Description | Benefits |
|-----------|-------------|----------|
| **Declarative Definition** | Define desired state, not procedural steps | System determines how to achieve state |
| **Version Control** | All infrastructure code in Git | History, collaboration, rollback |
| **Idempotency** | Apply repeatedly with same result | Safe re-runs, drift correction |
| **Modularity** | Reusable, composable components | DRY, consistency |
| **Immutability** | Replace rather than modify | Reproducibility, reduced drift |
| **Testing** | Validate before deployment | Confidence, quality |
| **Self-Documentation** | Code describes infrastructure | Always up-to-date documentation |

### IaC Benefits

| Benefit | Description | Impact |
|---------|-------------|--------|
| **Consistency** | Identical infrastructure across environments | Fewer "works on my machine" issues |
| **Speed** | Provision in minutes, not weeks | Faster time to market |
| **Repeatability** | Rebuild infrastructure reliably | Disaster recovery, scaling |
| **Documentation** | Code is the documentation | Always current, never stale |
| **Auditability** | Git history shows all changes | Compliance, troubleshooting |
| **Cost Reduction** | Automation reduces manual effort | Lower operational costs |
| **Quality** | Code review, testing | Fewer errors, better architecture |

### Declarative vs Imperative

| Approach | Description | Example | Use Case |
|----------|-------------|---------|----------|
| **Declarative** | Define desired end state | Terraform, CloudFormation | Infrastructure provisioning |
| **Imperative** | Define steps to achieve state | Scripts, Ansible (procedural) | Complex procedural logic |
| **Hybrid** | Combination of both | Pulumi (code with declarative model) | Flexibility with structure |

---

## IaC Tools Landscape

### Tool Categories

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    IaC TOOL CATEGORIES                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                    PROVISIONING TOOLS                                  │  │
│  │  Create/manage cloud resources, networking, storage                    │  │
│  │  • Terraform    • CloudFormation    • Pulumi    • CDK                 │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│                                   ▼                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                    CONFIGURATION MANAGEMENT                            │  │
│  │  Configure systems, install software, manage settings                  │  │
│  │  • Ansible    • Puppet    • Chef    • Salt                            │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│                                   ▼                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                    CONTAINER/IMAGE TOOLS                               │  │
│  │  Build and manage container images and specifications                  │  │
│  │  • Docker/Dockerfile    • Packer    • Buildah                         │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│                                   ▼                                          │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                    ORCHESTRATION TOOLS                                 │  │
│  │  Manage containerized workloads at scale                               │  │
│  │  • Kubernetes    • Nomad    • Docker Swarm    • ECS                   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Tool Comparison

| Tool | Type | Language | Multi-Cloud | State | Best For |
|------|------|----------|-------------|-------|----------|
| **Terraform** | Provisioning | HCL | Yes | Required | Multi-cloud, modules |
| **CloudFormation** | Provisioning | JSON/YAML | No (AWS) | Managed | AWS-only shops |
| **Pulumi** | Provisioning | Python/TS/Go | Yes | Required | Developers, complex logic |
| **AWS CDK** | Provisioning | Python/TS/Java | No (AWS) | Managed | AWS, type safety |
| **Ansible** | Config Mgmt | YAML | Yes | Stateless | Configuration, ad-hoc |
| **Puppet** | Config Mgmt | DSL | Yes | Agent-based | Large enterprises |
| **Chef** | Config Mgmt | Ruby | Yes | Agent-based | Complex automation |

### Terraform Deep Dive

Terraform is the most widely adopted multi-cloud IaC tool:

```hcl
# main.tf - Example Terraform configuration

terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "terraform-state-prod"
    key            = "infrastructure/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Project     = var.project_name
    }
  }
}

# VPC Module
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "${var.project_name}-${var.environment}-vpc"
  cidr = var.vpc_cidr

  azs             = var.availability_zones
  private_subnets = var.private_subnet_cidrs
  public_subnets  = var.public_subnet_cidrs

  enable_nat_gateway = true
  single_nat_gateway = var.environment != "prod"

  tags = {
    Name = "${var.project_name}-${var.environment}-vpc"
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  count = var.web_instance_count

  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.web_instance_type
  subnet_id     = module.vpc.private_subnets[count.index % length(module.vpc.private_subnets)]

  vpc_security_group_ids = [aws_security_group.web.id]

  root_block_device {
    volume_size = 30
    encrypted   = true
  }

  tags = {
    Name = "${var.project_name}-web-${count.index + 1}"
  }
}
```

### Ansible Deep Dive

Ansible is the leading agentless configuration management tool:

```yaml
# playbook.yml - Example Ansible playbook
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    nginx_port: 80
    app_user: appuser

  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

  tasks:
    - name: Install required packages
      package:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - python3
        - python3-pip

    - name: Create application user
      user:
        name: "{{ app_user }}"
        shell: /bin/bash
        create_home: yes

    - name: Deploy nginx configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
      notify: Restart nginx

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

---

## Module Development

### Module Structure

Well-organized modules are essential for reusable IaC:

```
modules/
├── vpc/
│   ├── main.tf           # Main resource definitions
│   ├── variables.tf      # Input variables
│   ├── outputs.tf        # Output values
│   ├── versions.tf       # Provider requirements
│   ├── locals.tf         # Local values
│   ├── README.md         # Documentation
│   └── examples/         # Usage examples
│       ├── simple/
│       │   └── main.tf
│       └── complete/
│           └── main.tf
├── ec2/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
└── rds/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

### Module Best Practices

| Practice | Description | Example |
|----------|-------------|---------|
| **Single Purpose** | One module, one function | VPC module only creates VPCs |
| **Clear Interface** | Well-defined inputs and outputs | Required vs optional variables |
| **Sensible Defaults** | Reasonable default values | `instance_type = "t3.micro"` |
| **Documentation** | README with usage examples | Include all variables/outputs |
| **Versioning** | Semantic versioning | v1.0.0, v1.1.0, v2.0.0 |
| **Testing** | Automated tests | Terratest, kitchen-terraform |
| **Validation** | Input validation | Variable validation blocks |

### Module Variables Example

```hcl
# variables.tf

variable "name" {
  description = "Name prefix for all resources"
  type        = string

  validation {
    condition     = length(var.name) <= 20
    error_message = "Name must be 20 characters or less."
  }
}

variable "environment" {
  description = "Environment (dev, staging, prod)"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "enable_monitoring" {
  description = "Enable detailed monitoring"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Additional tags to apply"
  type        = map(string)
  default     = {}
}
```

---

## State Management

### State Concepts

| Concept | Description | Importance |
|---------|-------------|------------|
| **State File** | Records current infrastructure state | Tracks what Terraform manages |
| **Remote State** | Shared state storage | Team collaboration |
| **State Locking** | Prevent concurrent modifications | Avoid corruption |
| **State Encryption** | Protect sensitive data | Security requirement |

### Remote State Configuration

```hcl
# AWS S3 Backend
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/infrastructure/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-locks"

    # Optional: Use IAM role
    role_arn       = "arn:aws:iam::123456789012:role/TerraformRole"
  }
}

# Azure Blob Backend
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# Terraform Cloud/Enterprise
terraform {
  cloud {
    organization = "my-company"

    workspaces {
      name = "production-infrastructure"
    }
  }
}
```

### State Best Practices

| Practice | Description | Why |
|----------|-------------|-----|
| **Remote Backend** | Never store state locally in production | Team access, durability |
| **Enable Locking** | Prevent concurrent changes | Avoid state corruption |
| **Encrypt State** | State contains sensitive data | Security, compliance |
| **Separate State** | Different state per environment | Isolation, safety |
| **Backup State** | Regular state backups | Recovery capability |
| **Minimize State** | Use data sources over state | Reduce blast radius |

---

## IaC Testing

### Testing Pyramid

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    IaC TESTING PYRAMID                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                              /\                                              │
│                             /  \                                             │
│                            /    \         E2E Tests                          │
│                           / E2E  \        (Full deployment)                  │
│                          /________\                                          │
│                         /          \                                         │
│                        / Integration\      Integration Tests                 │
│                       /______________\     (Deploy to test account)          │
│                      /                \                                      │
│                     /   Unit Tests     \    Unit Tests                       │
│                    /____________________\   (Module, syntax, logic)          │
│                   /                      \                                   │
│                  /    Static Analysis     \  Static Analysis                 │
│                 /__________________________\ (Lint, validate, security)      │
│                                                                              │
│   More Tests ◄─────────────────────────────────────────────► Slower/Costly  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Test Types

| Type | Purpose | Tools | Speed |
|------|---------|-------|-------|
| **Linting** | Code style, formatting | tflint, ansible-lint | Fast |
| **Validation** | Syntax correctness | terraform validate | Fast |
| **Security Scanning** | Find security issues | Checkov, tfsec, Snyk | Fast |
| **Unit Tests** | Test module logic | Terratest, pytest | Medium |
| **Integration Tests** | Test deployment | Kitchen, Molecule | Slow |
| **E2E Tests** | Full environment | Custom scripts | Slowest |

### Example: Terratest

```go
// vpc_test.go
package test

import (
    "testing"
    "github.com/gruntwork-io/terratest/modules/terraform"
    "github.com/stretchr/testify/assert"
)

func TestVpcModule(t *testing.T) {
    t.Parallel()

    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{
            "name":        "test-vpc",
            "environment": "test",
            "cidr":        "10.0.0.0/16",
        },
    })

    // Clean up resources at the end
    defer terraform.Destroy(t, terraformOptions)

    // Deploy infrastructure
    terraform.InitAndApply(t, terraformOptions)

    // Validate outputs
    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)

    privateSubnets := terraform.OutputList(t, terraformOptions, "private_subnets")
    assert.Equal(t, 3, len(privateSubnets))
}
```

### Example: Checkov Security Scan

```bash
# Run Checkov on Terraform code
checkov -d . --framework terraform

# Common checks:
# CKV_AWS_79: Ensure EC2 instance has encrypted EBS volumes
# CKV_AWS_23: Ensure Security Groups don't allow 0.0.0.0/0 to port 22
# CKV_AWS_144: Ensure S3 bucket has cross-region replication enabled
```

---

## IaC Pipelines

### Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    IaC PIPELINE ARCHITECTURE                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐  │
│  │  Commit  │──►│   Lint   │──►│ Validate │──►│ Security │──►│  Plan    │  │
│  │          │   │          │   │          │   │   Scan   │   │          │  │
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘   └────┬─────┘  │
│                                                                    │        │
│                                                                    ▼        │
│                                                          ┌──────────────┐   │
│                                                          │   PR Review  │   │
│                                                          │   (Plan in   │   │
│                                                          │   comments)  │   │
│                                                          └──────┬───────┘   │
│                                                                 │           │
│                                              On Merge to main   │           │
│                                                                 ▼           │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐           ┌──────────────┐     │
│  │ Complete │◄──│  Verify  │◄──│  Apply   │◄──────────│   Approval   │     │
│  │          │   │          │   │          │           │   (Manual)   │     │
│  └──────────┘   └──────────┘   └──────────┘           └──────────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### GitHub Actions Example

```yaml
# .github/workflows/terraform.yml
name: Terraform CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  TF_VERSION: 1.6.0
  AWS_REGION: us-west-2

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup TFLint
        uses: terraform-linters/setup-tflint@v4

      - name: Run TFLint
        run: tflint --recursive

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: .
          framework: terraform

  plan:
    runs-on: ubuntu-latest
    needs: [lint, security]
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        run: terraform plan -out=plan.tfplan

      - name: Upload Plan
        uses: actions/upload-artifact@v3
        with:
          name: terraform-plan
          path: plan.tfplan

  apply:
    runs-on: ubuntu-latest
    needs: [plan]
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Download Plan
        uses: actions/download-artifact@v3
        with:
          name: terraform-plan

      - name: Terraform Apply
        run: terraform apply -auto-approve plan.tfplan
```

---

## Best Practices

### Code Organization

| Practice | Description | Example |
|----------|-------------|---------|
| **Environment Separation** | Separate directories/workspaces | `environments/prod/`, `environments/dev/` |
| **Module Reuse** | Shared modules across environments | `modules/vpc/` used by all environments |
| **Variable Hierarchy** | Environment-specific variable files | `prod.tfvars`, `dev.tfvars` |
| **Consistent Naming** | Standard naming convention | `{project}-{env}-{resource}` |

### Project Structure

```
infrastructure/
├── modules/                      # Reusable modules
│   ├── vpc/
│   ├── eks/
│   ├── rds/
│   └── security/
├── environments/                 # Environment-specific
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
├── global/                       # Cross-environment resources
│   ├── iam/
│   └── dns/
└── .github/
    └── workflows/
        └── terraform.yml
```

### Security Best Practices

| Practice | Description | Implementation |
|----------|-------------|----------------|
| **No Secrets in Code** | Never hardcode credentials | Use AWS Secrets Manager, Vault |
| **Least Privilege** | Minimal IAM permissions | Role-based access for Terraform |
| **Encryption** | Encrypt state and secrets | S3 encryption, KMS |
| **Audit Logging** | Log all changes | CloudTrail, audit logs |
| **Code Review** | Review all changes | Pull request approval |
| **Security Scanning** | Automated security checks | Checkov, tfsec in pipeline |

---

## Review Questions

1. **IaC Principles**: What are the key differences between declarative and imperative IaC approaches? When would you choose each?

2. **Tool Selection**: Your organization uses AWS, Azure, and GCP. Which IaC tools would you recommend and why?

3. **Module Design**: Design a Terraform module structure for a three-tier web application with VPC, load balancer, compute, and database components.

4. **State Management**: A team accidentally deleted their Terraform state file. What recovery options exist, and how can this be prevented?

5. **Testing Strategy**: Design a testing strategy for a critical production infrastructure module. What tests would you implement at each level?

6. **Pipeline Design**: Design a CI/CD pipeline for Terraform that includes proper controls for production deployments.

---

## Key Takeaways

- **IaC principles** enable consistent, repeatable infrastructure management
- **Tool selection** depends on use case—Terraform for provisioning, Ansible for configuration
- **Modules** create reusable infrastructure components—follow single-purpose design
- **State management** is critical—always use remote state with locking
- **Testing** validates infrastructure before deployment—implement at multiple levels
- **Pipelines** automate and control infrastructure changes—require approval for production

---

## Summary

Infrastructure as Code is foundational to modern infrastructure management. By treating infrastructure like software—with version control, testing, code review, and automated deployment—organizations achieve consistency, speed, and reliability.

Success with IaC requires adopting software engineering practices: modular design, comprehensive testing, code review, and automated pipelines. The investment in building these practices pays dividends in reduced errors, faster delivery, and easier maintenance.

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 7: High Availability](/InfrastructureAndPlatformManagementHandbook/chapters/07-high-availability/) | [Chapter 9: Container Platforms](/InfrastructureAndPlatformManagementHandbook/chapters/09-container-platforms/) |
