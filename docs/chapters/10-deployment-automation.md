---
layout: default
title: "Chapter 10: Deployment and Release Automation"
parent: "Part III: Build and Deployment"
nav_order: 10
permalink: /chapters/10-deployment-automation/
---

# Chapter 10: Deployment and Release Automation

## Learning Objectives

After completing this chapter, you will be able to:
- Design CI/CD pipelines for infrastructure
- Implement deployment strategies (blue-green, canary, rolling)
- Manage multiple environments effectively
- Automate release orchestration
- Implement rollback procedures
- Configure GitOps workflows for infrastructure

---

## Introduction

Deployment automation enables consistent, reliable infrastructure changes. By automating deployments, organizations reduce human error, accelerate delivery, and improve quality. Manual deployments are error-prone, slow, and difficult to reproduce. Modern infrastructure management requires automated pipelines that can deploy changes safely, quickly, and repeatedly across multiple environments.

This chapter covers the principles, patterns, and practices for automating infrastructure deployments, from basic CI/CD pipelines to advanced deployment strategies and GitOps workflows.

---

## CI/CD for Infrastructure

### Infrastructure Pipeline Fundamentals

Infrastructure CI/CD differs from application CI/CD in several key ways:

| Aspect | Application CI/CD | Infrastructure CI/CD |
|--------|-------------------|---------------------|
| Artifact | Compiled binaries, containers | Terraform plans, CloudFormation templates |
| Testing | Unit tests, integration tests | Policy validation, security scans |
| Deployment | Replace containers/processes | Modify cloud resources, networks |
| Rollback | Deploy previous version | Apply previous state (sometimes destructive) |
| State | Stateless (typically) | Stateful (resources persist) |
| Blast Radius | Single service | Potentially entire infrastructure |

### Pipeline Stages

```
+-----------------------------------------------------------------------+
|                 INFRASTRUCTURE CI/CD PIPELINE                          |
+-----------------------------------------------------------------------+
|                                                                        |
|  +--------+  +--------+  +--------+  +--------+  +--------+           |
|  | Commit |->|  Lint  |->|  Test  |->|  Plan  |->| Review |           |
|  +--------+  +--------+  +--------+  +--------+  +--------+           |
|       |                                               |                |
|       |                                  +------------+                |
|       |                                  v                             |
|  +--------+  +--------+  +--------+  +--------+                       |
|  | Verify |<-| Apply  |<-|Approve |<-|  Plan  |                       |
|  +--------+  +--------+  +--------+  +--------+                       |
|       |                                                                |
|       v                                                                |
|  +--------+                                                            |
|  |Monitor |  Continuous monitoring and alerting                       |
|  +--------+                                                            |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Pipeline Components

| Stage | Purpose | Tools | Activities |
|-------|---------|-------|------------|
| Lint | Code quality | tflint, ansible-lint, yamllint | Syntax validation, style enforcement |
| Format | Code consistency | terraform fmt, black | Auto-format code to standards |
| Validate | Configuration validity | terraform validate | Check configuration syntax |
| Security | Vulnerability scan | Checkov, tfsec, Snyk | Policy compliance, security rules |
| Test | Unit and integration | Terratest, Kitchen-Terraform | Verify infrastructure behavior |
| Plan | Preview changes | terraform plan | Show what will change |
| Cost | Estimate impact | Infracost | Calculate cost implications |
| Review | Human approval | Pull requests, code review | Peer validation |
| Apply | Deploy changes | terraform apply | Execute infrastructure changes |
| Verify | Post-deployment | Automated tests, smoke tests | Confirm deployment success |

### Sample GitHub Actions Pipeline

```yaml
name: Infrastructure CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'terraform/**'
  pull_request:
    branches: [main]
    paths:
      - 'terraform/**'

env:
  TF_VERSION: '1.5.0'
  WORKING_DIR: './terraform/environments/production'

jobs:
  validate:
    name: Validate
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Terraform Format
        run: terraform fmt -check -recursive
        working-directory: ./terraform

      - name: Terraform Init
        run: terraform init -backend=false
        working-directory: ${{ env.WORKING_DIR }}

      - name: Terraform Validate
        run: terraform validate
        working-directory: ${{ env.WORKING_DIR }}

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1.0.0
        with:
          working_directory: ./terraform

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: ./terraform
          framework: terraform

  plan:
    name: Plan
    runs-on: ubuntu-latest
    needs: [validate, security]
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Terraform Init
        run: terraform init
        working-directory: ${{ env.WORKING_DIR }}

      - name: Terraform Plan
        run: terraform plan -out=tfplan
        working-directory: ${{ env.WORKING_DIR }}

      - name: Upload Plan
        uses: actions/upload-artifact@v4
        with:
          name: tfplan
          path: ${{ env.WORKING_DIR }}/tfplan

  apply:
    name: Apply
    runs-on: ubuntu-latest
    needs: plan
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Download Plan
        uses: actions/download-artifact@v4
        with:
          name: tfplan
          path: ${{ env.WORKING_DIR }}

      - name: Terraform Init
        run: terraform init
        working-directory: ${{ env.WORKING_DIR }}

      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
        working-directory: ${{ env.WORKING_DIR }}
```

---

## Deployment Strategies

### Blue-Green Deployment

Blue-green deployment maintains two identical production environments. Only one environment serves traffic at any time, enabling instant rollback by switching traffic to the idle environment.

```
+-----------------------------------------------------------------------+
|                    BLUE-GREEN DEPLOYMENT                               |
+-----------------------------------------------------------------------+
|                                                                        |
|  BEFORE                             AFTER                              |
|  +------------------+               +------------------+               |
|  |   Load Balancer  |               |   Load Balancer  |               |
|  +--------+---------+               +--------+---------+               |
|           |                                  |                         |
|           | 100% traffic                     | 100% traffic            |
|           v                                  v                         |
|  +--------+---------+               +--------+---------+               |
|  |  BLUE (Active)   |               |  GREEN (Active)  |               |
|  |  Version 1.0     |               |  Version 2.0     |               |
|  |  [Serving]       |               |  [Serving]       |               |
|  +------------------+               +------------------+               |
|                                                                        |
|  +------------------+               +------------------+               |
|  |  GREEN (Idle)    |               |  BLUE (Standby)  |               |
|  |  Version 2.0     |               |  Version 1.0     |               |
|  |  [Staging]       |               |  [Rollback]      |               |
|  +------------------+               +------------------+               |
|                                                                        |
+-----------------------------------------------------------------------+
```

**Blue-Green Implementation with Terraform:**

```hcl
# Blue-Green using AWS ALB target groups
resource "aws_lb_listener_rule" "blue_green" {
  listener_arn = aws_lb_listener.main.arn
  priority     = 100

  action {
    type             = "forward"
    target_group_arn = var.active_environment == "blue" ?
                       aws_lb_target_group.blue.arn :
                       aws_lb_target_group.green.arn
  }

  condition {
    path_pattern {
      values = ["/*"]
    }
  }
}

resource "aws_lb_target_group" "blue" {
  name        = "${var.app_name}-blue"
  port        = 80
  protocol    = "HTTP"
  vpc_id      = var.vpc_id
  target_type = "instance"

  health_check {
    path                = "/health"
    healthy_threshold   = 2
    unhealthy_threshold = 3
    timeout             = 5
    interval            = 30
  }
}

resource "aws_lb_target_group" "green" {
  name        = "${var.app_name}-green"
  port        = 80
  protocol    = "HTTP"
  vpc_id      = var.vpc_id
  target_type = "instance"

  health_check {
    path                = "/health"
    healthy_threshold   = 2
    unhealthy_threshold = 3
    timeout             = 5
    interval            = 30
  }
}
```

| Advantage | Consideration |
|-----------|---------------|
| Instant rollback | Double infrastructure cost |
| Zero downtime | Database schema changes complex |
| Easy testing in production | Requires load balancer support |
| Full environment validation | Session management needed |

### Canary Deployment

Canary deployment gradually shifts traffic to the new version, allowing validation with a small percentage of users before full rollout.

```
+-----------------------------------------------------------------------+
|                    CANARY DEPLOYMENT PHASES                            |
+-----------------------------------------------------------------------+
|                                                                        |
|  Phase 1: Initial (5%)        Phase 2: Expand (25%)                   |
|  +------------------+         +------------------+                     |
|  |  Load Balancer   |         |  Load Balancer   |                    |
|  +--------+---------+         +--------+---------+                    |
|           |                            |                               |
|     +-----+-----+               +------+------+                        |
|     |           |               |             |                        |
|     v           v               v             v                        |
|  +------+    +------+        +------+     +------+                    |
|  | v1.0 |    | v2.0 |        | v1.0 |     | v2.0 |                    |
|  | 95%  |    |  5%  |        | 75%  |     | 25%  |                    |
|  +------+    +------+        +------+     +------+                    |
|                                                                        |
|  Phase 3: Majority (75%)      Phase 4: Complete (100%)                |
|  +------------------+         +------------------+                     |
|  |  Load Balancer   |         |  Load Balancer   |                    |
|  +--------+---------+         +--------+---------+                    |
|           |                            |                               |
|     +-----+-----+                      |                               |
|     |           |                      v                               |
|     v           v               +------+------+                        |
|  +------+    +------+           |    v2.0    |                        |
|  | v1.0 |    | v2.0 |           |    100%    |                        |
|  | 25%  |    | 75%  |           +-------------+                        |
|  +------+    +------+                                                  |
|                                                                        |
+-----------------------------------------------------------------------+
```

**Canary Metrics to Monitor:**

| Metric Category | Specific Metrics | Threshold |
|-----------------|------------------|-----------|
| Error Rate | HTTP 5xx responses | < 0.1% increase |
| Latency | P50, P95, P99 response time | < 10% increase |
| Saturation | CPU, memory utilization | < 80% |
| Business | Conversion rate, transactions | No degradation |

**Automated Canary Analysis:**

```yaml
# Argo Rollouts Canary Configuration
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: web-application
spec:
  replicas: 10
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: { duration: 5m }
        - analysis:
            templates:
              - templateName: success-rate
            args:
              - name: service-name
                value: web-application
        - setWeight: 25
        - pause: { duration: 10m }
        - analysis:
            templates:
              - templateName: success-rate
        - setWeight: 50
        - pause: { duration: 10m }
        - setWeight: 100
      canaryService: web-application-canary
      stableService: web-application-stable
```

### Rolling Deployment

Rolling deployment updates instances gradually while maintaining availability. Unlike blue-green, it doesn't require double infrastructure.

```
+-----------------------------------------------------------------------+
|                    ROLLING DEPLOYMENT                                  |
+-----------------------------------------------------------------------+
|                                                                        |
|  Time T0: Initial State                                                |
|  +--------+ +--------+ +--------+ +--------+ +--------+               |
|  |  v1.0  | |  v1.0  | |  v1.0  | |  v1.0  | |  v1.0  |               |
|  +--------+ +--------+ +--------+ +--------+ +--------+               |
|                                                                        |
|  Time T1: 20% Rollout                                                  |
|  +--------+ +--------+ +--------+ +--------+ +--------+               |
|  |  v2.0  | |  v1.0  | |  v1.0  | |  v1.0  | |  v1.0  |               |
|  +--------+ +--------+ +--------+ +--------+ +--------+               |
|                                                                        |
|  Time T2: 40% Rollout                                                  |
|  +--------+ +--------+ +--------+ +--------+ +--------+               |
|  |  v2.0  | |  v2.0  | |  v1.0  | |  v1.0  | |  v1.0  |               |
|  +--------+ +--------+ +--------+ +--------+ +--------+               |
|                                                                        |
|  Time T3: Complete                                                     |
|  +--------+ +--------+ +--------+ +--------+ +--------+               |
|  |  v2.0  | |  v2.0  | |  v2.0  | |  v2.0  | |  v2.0  |               |
|  +--------+ +--------+ +--------+ +--------+ +--------+               |
|                                                                        |
+-----------------------------------------------------------------------+
```

| Advantage | Consideration |
|-----------|---------------|
| No additional infrastructure | Temporary mixed versions |
| Gradual rollout | Longer deployment time |
| Easy to implement | Requires backward compatible versions |
| Built into most orchestrators | Rollback slower than blue-green |

### Deployment Strategy Selection

| Factor | Blue-Green | Canary | Rolling |
|--------|------------|--------|---------|
| Infrastructure Cost | High (2x) | Medium | Low |
| Rollback Speed | Instant | Fast | Slow |
| Risk Exposure | Full (at switch) | Gradual | Gradual |
| Validation Depth | Full environment | Production subset | Progressive |
| Complexity | Medium | High | Low |
| Best For | Critical apps, quick rollback | High-traffic, risk-sensitive | Standard deployments |

---

## Environment Management

### Environment Hierarchy

```
+-----------------------------------------------------------------------+
|                    ENVIRONMENT HIERARCHY                               |
+-----------------------------------------------------------------------+
|                                                                        |
|  +------------------+                                                  |
|  |   PRODUCTION     |  Live traffic, real data                        |
|  |   - Blue/Green   |  Most restrictive access                        |
|  +--------+---------+                                                  |
|           ^                                                            |
|           | Promotion                                                  |
|  +--------+---------+                                                  |
|  |    STAGING       |  Production-like, synthetic data                |
|  |    (Pre-Prod)    |  Final validation before production             |
|  +--------+---------+                                                  |
|           ^                                                            |
|           | Promotion                                                  |
|  +--------+---------+                                                  |
|  |    TESTING       |  Integration testing, QA                        |
|  |    (QA/UAT)      |  Automated test execution                       |
|  +--------+---------+                                                  |
|           ^                                                            |
|           | Promotion                                                  |
|  +--------+---------+                                                  |
|  |   DEVELOPMENT    |  Developer testing, feature validation          |
|  |                  |  Most frequent changes                          |
|  +------------------+                                                  |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Environment Strategy

| Environment | Purpose | Data | Access | Refresh |
|-------------|---------|------|--------|---------|
| Development | Developer testing | Minimal synthetic | Developers | Daily |
| Testing | QA and integration | Controlled test data | QA team | Per release |
| Staging | Pre-production validation | Anonymized production | Limited | Weekly |
| Production | Live workloads | Real production data | Restricted | N/A |

### Environment Parity

Maintaining consistency across environments is crucial for reliable deployments:

| Aspect | Implementation | Rationale |
|--------|----------------|-----------|
| Infrastructure | Same IaC, different parameters | Consistent architecture |
| Configuration | Environment-specific variables | Tailored settings |
| Secrets | Separate secrets per environment | Security isolation |
| Data | Anonymized production data | Realistic testing |
| Monitoring | Consistent across environments | Early issue detection |
| Networking | Similar topology | Network-related bug detection |

**Environment Configuration with Terraform:**

```hcl
# environments/production/main.tf
module "infrastructure" {
  source = "../../modules/infrastructure"

  environment = "production"

  # Compute configuration
  instance_type     = "m5.xlarge"
  min_capacity      = 4
  max_capacity      = 20
  desired_capacity  = 8

  # Database configuration
  db_instance_class = "db.r5.2xlarge"
  db_multi_az       = true
  db_backup_retention = 30

  # Network configuration
  vpc_cidr          = "10.0.0.0/16"
  private_subnets   = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets    = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  # Feature flags
  enable_waf        = true
  enable_shield     = true

  tags = {
    Environment = "production"
    ManagedBy   = "terraform"
    CostCenter  = "infrastructure"
  }
}

# environments/staging/main.tf
module "infrastructure" {
  source = "../../modules/infrastructure"

  environment = "staging"

  # Scaled-down compute
  instance_type     = "m5.large"
  min_capacity      = 2
  max_capacity      = 6
  desired_capacity  = 2

  # Scaled-down database
  db_instance_class = "db.r5.large"
  db_multi_az       = false
  db_backup_retention = 7

  # Same network topology, different CIDR
  vpc_cidr          = "10.1.0.0/16"
  private_subnets   = ["10.1.1.0/24", "10.1.2.0/24", "10.1.3.0/24"]
  public_subnets    = ["10.1.101.0/24", "10.1.102.0/24", "10.1.103.0/24"]

  # Feature flags (testing production features)
  enable_waf        = true
  enable_shield     = false

  tags = {
    Environment = "staging"
    ManagedBy   = "terraform"
    CostCenter  = "infrastructure"
  }
}
```

---

## GitOps for Infrastructure

### GitOps Principles

GitOps applies DevOps best practices to infrastructure management, using Git as the single source of truth.

```
+-----------------------------------------------------------------------+
|                    GITOPS WORKFLOW                                     |
+-----------------------------------------------------------------------+
|                                                                        |
|  +----------+     +----------+     +----------+     +----------+      |
|  |Developer |---->|   Git    |---->|  GitOps  |---->|  Target  |      |
|  |  Commit  |     |   Repo   |     |  Agent   |     |  System  |      |
|  +----------+     +----+-----+     +----+-----+     +----------+      |
|                        |                |                              |
|                        |                |                              |
|                        v                v                              |
|                   +----------+    +----------+                         |
|                   |  Pull    |    | Reconcile|                         |
|                   | Request  |    |   Loop   |                         |
|                   +----------+    +----------+                         |
|                        |                |                              |
|                        v                v                              |
|                   +----------+    +----------+                         |
|                   |  Review  |    |  Detect  |                         |
|                   |  Approve |    |  Drift   |                         |
|                   +----------+    +----------+                         |
|                                                                        |
+-----------------------------------------------------------------------+
```

| GitOps Principle | Description |
|------------------|-------------|
| Declarative | Desired state defined in Git |
| Versioned | All changes tracked in Git history |
| Automated | Agents automatically apply changes |
| Continuously Reconciled | Drift detected and corrected |

### GitOps Tools Comparison

| Tool | Type | Strengths | Use Case |
|------|------|-----------|----------|
| ArgoCD | Pull-based | Kubernetes-native, UI | Kubernetes deployments |
| Flux | Pull-based | Lightweight, flexible | Kubernetes, Helm |
| Atlantis | Push-based | Terraform-focused | Infrastructure |
| Env0 | Push-based | Multi-cloud, cost | Enterprise Terraform |

### Atlantis Configuration

```yaml
# atlantis.yaml
version: 3
automerge: true
delete_source_branch_on_merge: true

projects:
  - name: production
    dir: environments/production
    workspace: default
    terraform_version: v1.5.0
    autoplan:
      when_modified:
        - "*.tf"
        - "*.tfvars"
        - "../modules/**/*.tf"
      enabled: true
    apply_requirements:
      - approved
      - mergeable
    workflow: production

  - name: staging
    dir: environments/staging
    workspace: default
    terraform_version: v1.5.0
    autoplan:
      when_modified:
        - "*.tf"
        - "*.tfvars"
        - "../modules/**/*.tf"
      enabled: true
    workflow: default

workflows:
  production:
    plan:
      steps:
        - init
        - run: tfsec .
        - run: checkov -d .
        - plan:
            extra_args: ["-var-file", "production.tfvars"]
    apply:
      steps:
        - apply

  default:
    plan:
      steps:
        - init
        - plan
    apply:
      steps:
        - apply
```

---

## Rollback Procedures

### Rollback Strategies

| Strategy | Description | When to Use | Recovery Time |
|----------|-------------|-------------|---------------|
| Version Revert | Apply previous IaC version | IaC-managed infrastructure | Minutes |
| Blue-Green Switch | Switch traffic to old environment | Blue-green deployments | Seconds |
| Snapshot Restore | Restore from backup | Data corruption, state issues | Minutes to hours |
| Manual Intervention | Hand-crafted fixes | Complex failures | Variable |

### Automated Rollback Triggers

```yaml
# Automated rollback based on health checks
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: error-rate-analysis
spec:
  metrics:
    - name: error-rate
      interval: 1m
      successCondition: result[0] < 0.05  # < 5% error rate
      failureLimit: 3
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            sum(rate(http_requests_total{status=~"5.*"}[5m]))
            /
            sum(rate(http_requests_total[5m]))

    - name: latency-p99
      interval: 1m
      successCondition: result[0] < 0.5  # < 500ms p99
      failureLimit: 3
      provider:
        prometheus:
          address: http://prometheus:9090
          query: |
            histogram_quantile(0.99,
              sum(rate(http_request_duration_seconds_bucket[5m]))
              by (le))
```

### Rollback Runbook

| Step | Action | Verification |
|------|--------|--------------|
| 1 | Identify rollback need | Review metrics, alerts, user reports |
| 2 | Notify stakeholders | Incident channel, status page |
| 3 | Execute rollback | Run rollback procedure |
| 4 | Verify rollback | Check health endpoints, metrics |
| 5 | Investigate root cause | Post-incident analysis |
| 6 | Document findings | Update runbooks, improve process |

---

## Review Questions

1. What are the key differences between infrastructure CI/CD and application CI/CD?
2. When would you choose canary deployment over blue-green deployment?
3. How does GitOps improve infrastructure management?
4. What metrics should trigger an automatic rollback?
5. How do you maintain environment parity while managing costs?

---

## Summary

Deployment automation transforms infrastructure management from error-prone manual processes to reliable, repeatable operations. Key elements include:

- **CI/CD pipelines** validate, test, and deploy infrastructure changes
- **Deployment strategies** (blue-green, canary, rolling) manage risk
- **Environment management** ensures consistency across stages
- **GitOps** uses Git as the single source of truth
- **Rollback procedures** enable rapid recovery from failures

---

## Key Takeaways

- **CI/CD pipelines** automate infrastructure deployment with validation gates
- **Deployment strategies** manage risk during changes through gradual rollout
- **Environment management** maintains consistency while controlling costs
- **GitOps** provides declarative, version-controlled infrastructure
- **Rollback procedures** enable quick recovery from failed deployments

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 9: Container Platforms](/InfrastructureAndPlatformManagementHandbook/chapters/09-container-platforms/) | [Chapter 11: Monitoring and Observability](/InfrastructureAndPlatformManagementHandbook/chapters/11-monitoring-observability/) |
