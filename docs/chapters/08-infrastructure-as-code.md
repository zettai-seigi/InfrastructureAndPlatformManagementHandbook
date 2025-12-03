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

Infrastructure as Code represents one of the most significant transformations in how organizations manage their technology environments. For decades, infrastructure management meant logging into servers, clicking through management consoles, and maintaining sprawling spreadsheets of configuration details. When something went wrong, teams scrambled to understand what had changed and when. Rebuilding a failed server might take days of painstaking work, and creating a new environment that matched production was often more art than science.

Infrastructure as Code changes all of this by applying software engineering discipline to infrastructure management. Instead of making changes through graphical interfaces and hoping to remember what you did, you write code that describes exactly what your infrastructure should look like. This code lives in version control, where every change is tracked, reviewed, and auditable. When you need to rebuild infrastructure, you run the code again and get exactly what you had before.

The business impact of this transformation is profound. Organizations that adopt IaC effectively can provision new environments in minutes rather than weeks. They can confidently make changes knowing they can roll back if something goes wrong. They can ensure that development, staging, and production environments are truly identical, eliminating the dreaded "works on my machine" problem. Perhaps most importantly, they can scale their infrastructure operations without proportionally scaling their teams—the same code that provisions one server can provision a hundred.

This chapter provides a comprehensive guide to implementing Infrastructure as Code effectively. We will explore the fundamental principles that make IaC work, examine the major tools in the ecosystem, develop strategies for building reusable infrastructure components, and establish practices for testing and deploying infrastructure safely. Whether you are just beginning your IaC journey or looking to mature your existing practices, this chapter provides the foundation you need.

---

## The Evolution from Manual to Coded Infrastructure

### Understanding the Traditional Approach

To appreciate the value of Infrastructure as Code, it helps to understand what came before. Traditional infrastructure management relied heavily on manual processes. When a team needed a new server, they would submit a request, wait for approval, and then watch as an administrator worked through a checklist of configuration steps. Each server might take hours or days to provision, and the configuration often varied subtly between machines depending on who set them up and when.

Documentation was perpetually out of date. The official configuration guide might describe how servers should be configured, but the reality often drifted over time as administrators made quick fixes, applied patches, or experimented with optimizations. When problems arose, teams had to investigate what the current state actually was before they could begin troubleshooting.

This approach created several cascading problems. Environment drift meant that production behaved differently from staging, which behaved differently from development. Changes were risky because nobody was entirely sure what the current configuration was. Scaling was painful because adding capacity meant repeating all those manual steps. Disaster recovery was uncertain because recreating infrastructure from scratch required heroic effort.

### The Infrastructure as Code Revolution

Infrastructure as Code addresses these problems by making infrastructure configuration explicit, repeatable, and version-controlled. Instead of clicking through a console to create a virtual machine, you write code that declares what that virtual machine should look like—its size, its network configuration, its attached storage, its security groups. The IaC tool then reads your code and makes reality match your declaration.

This shift from imperative to declarative thinking is subtle but powerful. Traditional scripts told the system what steps to take: "Create a VM, then attach a disk, then configure networking." Declarative IaC tells the system what the end state should be: "There should be a VM with these properties." The tool figures out what steps are needed to achieve that state, including what to do if some parts already exist or need to be modified.

The benefits cascade throughout the organization. Version control provides complete history of every change, making auditing straightforward and rollback possible. Code review catches problems before they reach production. Automated testing validates that infrastructure works correctly. Continuous integration and deployment pipelines ensure changes flow smoothly from development to production.

---

## Core Principles of Infrastructure as Code

### Declarative Over Imperative

The most fundamental principle of modern IaC is declarative definition. Rather than writing scripts that describe procedural steps, you write configurations that describe the desired end state. This approach offers several advantages that become clear over time.

First, declarative configurations are idempotent by design. You can run the same configuration multiple times and get the same result. If the infrastructure already matches the declaration, nothing changes. If it has drifted, the tool corrects the drift. This property makes it safe to re-run configurations, which is essential for automation.

Second, declarative configurations are self-documenting. The code itself describes what the infrastructure should look like. Unlike procedural scripts where understanding the current state requires mentally executing the script, declarative configurations can be read directly to understand intent.

Third, declarative tools can plan changes before applying them. Because the tool knows both the current state and the desired state, it can compute and display exactly what changes it will make. This plan can be reviewed, approved, and only then applied.

### Version Control as the Source of Truth

Every piece of infrastructure code should live in version control, typically Git. This principle seems obvious but has deeper implications than simply having a backup of your configurations.

Version control makes infrastructure changes auditable. When something breaks, you can look at the commit history to see what changed, who changed it, when, and why (assuming good commit messages). This audit trail is invaluable for troubleshooting and for compliance requirements that demand change documentation.

Version control enables collaboration. Multiple team members can work on infrastructure changes simultaneously, using branches and pull requests to manage concurrent work. Code review becomes possible, bringing additional eyes to infrastructure changes before they affect production.

Version control enables rollback. If a change causes problems, you can revert to a previous version of the configuration and apply it. The infrastructure returns to its previous known-good state. This capability dramatically reduces the risk of making changes.

### Modularity and Reusability

Well-designed IaC is modular. Rather than writing monolithic configurations that describe an entire environment, you create smaller, focused modules that can be composed together. A VPC module creates networking infrastructure. A compute module creates servers. A database module creates data stores. You combine these modules to build complete environments.

Modularity provides several benefits. Modules can be reused across environments and projects, ensuring consistency and reducing duplication. Changes to a module flow to all consumers, making updates easier. Modules can be tested independently, improving quality. Teams can work on different modules simultaneously without conflicts.

Good modules have clear interfaces. They accept inputs (variables) that customize their behavior and produce outputs that other modules can consume. The internal implementation is hidden, allowing the module to be improved without affecting consumers. This encapsulation enables teams to build on each other's work effectively.

### Immutability as a Strategy

The immutable infrastructure pattern takes declarative IaC to its logical conclusion. Rather than modifying existing infrastructure when changes are needed, you replace it with new infrastructure that has the changes built in. The old infrastructure is then destroyed.

This pattern might seem wasteful, but it offers compelling advantages. Immutable infrastructure eliminates configuration drift by design—there is no drift when every change involves building from scratch. Rollback becomes trivial: you simply deploy the previous version again. The infrastructure is inherently reproducible because it is always built from the same process.

Immutability works particularly well with containerized applications and auto-scaling groups. When you need to update an application, you build a new container image and replace the running containers. When you need to update server configurations, you build a new machine image and replace the instances in the auto-scaling group. The infrastructure is treated as cattle, not pets—interchangeable resources rather than precious individuals.

---

## The Infrastructure as Code Tool Landscape

### Provisioning Tools

Provisioning tools create and manage cloud resources—the building blocks of infrastructure. They interact with cloud APIs to create virtual machines, networks, storage, databases, and the hundreds of other resource types that cloud providers offer.

Terraform has emerged as the dominant multi-cloud provisioning tool. Created by HashiCorp, Terraform uses a declarative language called HCL (HashiCorp Configuration Language) to describe infrastructure. Its provider ecosystem supports every major cloud provider plus hundreds of other services, from DNS providers to monitoring platforms. Terraform maintains a state file that tracks the current infrastructure, enabling it to compute plans and detect drift.

The strength of Terraform lies in its multi-cloud capability and ecosystem. You can use the same workflow and similar concepts whether you're working with AWS, Azure, GCP, or on-premises infrastructure. The module registry provides thousands of pre-built modules that encapsulate best practices. The community is large and active, ensuring good documentation and rapid problem-solving.

Cloud-specific tools offer an alternative for organizations committed to a single provider. AWS CloudFormation is the native provisioning tool for AWS, offering deep integration with AWS services and features. Azure Resource Manager templates serve a similar role for Azure. These tools often support newer services before third-party tools do, and they integrate seamlessly with other native management features.

Pulumi represents a newer approach that allows infrastructure to be defined in general-purpose programming languages like Python, TypeScript, or Go. This appeals to developers who prefer familiar languages over domain-specific ones. Pulumi enables sophisticated logic, loops, and conditionals that can be awkward in declarative languages. It supports all major clouds and maintains Terraform-compatible state.

### Configuration Management Tools

While provisioning tools create infrastructure resources, configuration management tools configure what runs on those resources. They install software, manage configurations, and ensure systems remain in their desired state.

Ansible has become the most popular configuration management tool, largely due to its agentless architecture and gentle learning curve. Ansible connects to managed systems over SSH and executes tasks without requiring any software to be pre-installed on the target. Its playbooks use YAML syntax that non-developers can read and understand. The large collection of built-in modules handles common tasks like package installation, file management, and service control.

Ansible shines for ad-hoc tasks and procedural workflows where the order of operations matters. It integrates well with provisioning tools—many teams use Terraform to create infrastructure and Ansible to configure it. The dynamic inventory feature allows Ansible to discover infrastructure created by other tools and manage it accordingly.

Puppet and Chef represent an earlier generation of configuration management tools that use agents installed on managed systems. These tools offer sophisticated features for managing configuration at enterprise scale, including robust reporting, compliance checking, and self-healing capabilities. They require more infrastructure to operate (Puppet servers, Chef servers) but provide capabilities that matter in large environments.

### Container and Image Tools

Modern infrastructure increasingly relies on containers and machine images rather than configuring long-lived systems. This shift has spawned a category of tools focused on building and managing these artifacts.

Docker transformed application deployment by packaging applications with their dependencies into portable containers. Dockerfiles describe how to build container images, layer by layer. These images can run consistently across development laptops, CI systems, and production servers. The container runtime provides isolation without the overhead of full virtual machines.

Packer complements container tools by building machine images for virtual machines and cloud instances. Rather than configuring a base image manually and creating a snapshot, Packer automates the entire process. It can build images for multiple platforms from a single configuration, ensuring that your AWS AMIs, Azure images, and VMware templates are consistent.

The container ecosystem continues to evolve rapidly. Tools like Buildah offer alternatives to Docker for building images. Containerd and CRI-O provide container runtimes for Kubernetes. Image registries like Docker Hub, ECR, and Harbor store and distribute images. Security scanners like Trivy and Clair check images for vulnerabilities.

---

## Building with Terraform

### Understanding Terraform's Architecture

Terraform's architecture centers on the concept of providers and resources. A provider is a plugin that knows how to interact with a particular API—the AWS provider knows how to create AWS resources, the Kubernetes provider knows how to create Kubernetes objects. Resources are the individual infrastructure components that Terraform manages.

When you run Terraform, it follows a consistent workflow. First, `terraform init` downloads the required providers and initializes the backend where state will be stored. Then, `terraform plan` compares your configuration to the current state and computes what changes are needed. Finally, `terraform apply` executes those changes, updating the state file to reflect the new reality.

The state file is a critical concept that newcomers often misunderstand. Terraform needs to know what infrastructure it currently manages so it can determine what changes to make. The state file maps the resources in your configuration to real infrastructure in the cloud. Losing the state file means Terraform no longer knows what it manages, making recovery difficult.

### Writing Effective Terraform Code

Good Terraform code is organized, readable, and maintainable. The conventions have evolved over years of community experience, and following them makes your code easier for others to understand and modify.

Configuration files typically follow a standard layout. The main.tf file contains the primary resource definitions. Variables.tf declares input variables that customize the configuration. Outputs.tf declares values to expose to other configurations. Versions.tf specifies the required Terraform and provider versions. Locals.tf defines local values for intermediate computations.

Variables should have descriptions and, where appropriate, validation rules. Descriptions help users understand what each variable does. Validation rules catch errors early, before Terraform tries to create resources with invalid values. Default values make variables optional when sensible defaults exist.

```hcl
# Example of well-structured Terraform configuration

terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "production/infrastructure.tfstate"
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

# Variables with validation
variable "environment" {
  description = "Deployment environment (dev, staging, prod)"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_type" {
  description = "EC2 instance type for web servers"
  type        = string
  default     = "t3.medium"
}

# Resources with clear naming
resource "aws_instance" "web" {
  count = var.instance_count

  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  subnet_id     = module.vpc.private_subnets[count.index % length(module.vpc.private_subnets)]

  vpc_security_group_ids = [aws_security_group.web.id]

  root_block_device {
    volume_size = 50
    encrypted   = true
  }

  tags = {
    Name = "${var.project_name}-web-${count.index + 1}"
  }
}

# Outputs for consumption by other configurations
output "instance_ids" {
  description = "IDs of the web server instances"
  value       = aws_instance.web[*].id
}
```

### Creating Reusable Modules

Modules are the key to scaling Terraform usage across teams and projects. A well-designed module encapsulates a complete piece of infrastructure—not just a single resource, but a coherent set of resources that work together.

A VPC module, for example, might create the VPC itself, public and private subnets across multiple availability zones, internet and NAT gateways, route tables, and network ACLs. Users of the module specify what they need (CIDR ranges, availability zones, whether to use NAT gateways) and receive a functioning network.

Module design requires thinking carefully about interfaces. What inputs does the module need? What outputs should it expose? What are sensible defaults? The goal is to make the module easy to use while remaining flexible enough for different use cases.

Good modules are published with documentation and examples. The README explains what the module does, how to use it, and what each variable means. Example configurations demonstrate common use cases. This documentation is essential because users of the module shouldn't need to read its internals to use it effectively.

```hcl
# Module structure example

# modules/vpc/variables.tf
variable "name" {
  description = "Name prefix for VPC resources"
  type        = string
}

variable "cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "enable_nat_gateway" {
  description = "Whether to provision NAT gateways"
  type        = bool
  default     = true
}

# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "${var.name}-vpc"
  }
}

# Additional resources for subnets, gateways, etc.

# modules/vpc/outputs.tf
output "vpc_id" {
  description = "ID of the created VPC"
  value       = aws_vpc.main.id
}

output "private_subnet_ids" {
  description = "IDs of private subnets"
  value       = aws_subnet.private[*].id
}
```

---

## Managing Configuration with Ansible

### The Ansible Approach

Ansible takes a fundamentally different approach from Terraform. Where Terraform manages infrastructure resources through cloud APIs, Ansible manages system configuration through SSH connections. Where Terraform maintains state, Ansible is stateless—each run evaluates the current system state and makes necessary changes.

This stateless approach has advantages and disadvantages. On the positive side, there is no state file to manage, back up, or potentially corrupt. Ansible can work with systems that weren't originally provisioned through IaC. It excels at one-off tasks and configurations that don't map well to declarative models. On the negative side, Ansible can't show you a plan before making changes, and it has no inherent way to detect or correct drift.

Ansible's YAML-based playbooks describe sequences of tasks to perform on groups of hosts. The inventory defines which hosts belong to which groups. Tasks use modules—over three thousand are built in—to perform common operations. Variables customize behavior across different environments or hosts.

### Writing Effective Playbooks

A well-written playbook is readable, maintainable, and idempotent. Readability comes from clear naming, logical organization, and appropriate comments. Maintainability comes from using roles to organize related tasks and from avoiding repetition. Idempotency comes from using modules correctly and testing that repeated runs don't cause unintended changes.

Playbooks should be organized into roles for anything beyond simple tasks. A role encapsulates everything needed for a specific function: the tasks to perform, variables with defaults, templates for configuration files, handlers for service restarts, and files to copy. Roles can be reused across playbooks and shared through Ansible Galaxy.

```yaml
# Example of a well-structured Ansible playbook

---
- name: Configure web application servers
  hosts: webservers
  become: yes

  vars:
    app_name: myapp
    app_user: appuser
    nginx_worker_processes: auto
    nginx_worker_connections: 1024

  pre_tasks:
    - name: Update apt cache if older than 1 hour
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

  roles:
    - role: common
      tags: common
    - role: nginx
      tags: nginx
    - role: application
      tags: app

  post_tasks:
    - name: Verify application is responding
      uri:
        url: "http://localhost/health"
        status_code: 200
      retries: 5
      delay: 10

# roles/nginx/tasks/main.yml
---
- name: Install nginx
  package:
    name: nginx
    state: present

- name: Create nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
    validate: nginx -t -c %s
  notify: Reload nginx

- name: Ensure nginx is running and enabled
  service:
    name: nginx
    state: started
    enabled: yes
```

### Combining Terraform and Ansible

Many organizations use Terraform and Ansible together, leveraging the strengths of each. Terraform handles provisioning—creating the cloud resources like virtual machines, networks, and databases. Ansible handles configuration—installing software, managing configuration files, and ensuring services are running correctly.

This combination requires passing information from Terraform to Ansible. Terraform knows the IP addresses and other details of the resources it created. Ansible needs this information to know what systems to manage. Dynamic inventory scripts or plugins can query Terraform state or cloud APIs to build the inventory automatically.

The workflow typically involves running Terraform first to create infrastructure, then running Ansible to configure it. For changes that only affect configuration (not infrastructure), you can run Ansible alone. For changes that require new infrastructure, you run both. Orchestration tools like Ansible itself (using the terraform module) or CI/CD pipelines coordinate these tools.

---

## State Management Strategies

### The Importance of State

State is the mechanism by which Terraform tracks what infrastructure it manages. Without state, Terraform would have no way to know whether a resource in your configuration already exists or needs to be created. It couldn't compute plans showing what will change. It couldn't detect drift between your configuration and reality.

The state file contains sensitive information. Resource IDs, IP addresses, and sometimes secrets appear in state. This makes state management a security concern, not just an operational one. State files should be encrypted at rest and in transit. Access should be controlled and audited.

State should also be protected from concurrent modification. If two people run Terraform at the same time, they might both read the current state, compute plans, and try to apply changes. The resulting state could be inconsistent or corrupted. State locking prevents this by ensuring only one operation proceeds at a time.

### Implementing Remote State

Local state files—stored on your computer's filesystem—work for learning and simple experiments but fail for team use. Remote state stores the state file in a shared location that multiple team members can access, with locking to prevent concurrent modifications.

For AWS environments, S3 with DynamoDB locking is the most common remote state backend. The state file lives in an S3 bucket with versioning enabled for recovery. A DynamoDB table provides locking so that only one operation can modify the state at a time. Both should be encrypted and access-controlled.

```hcl
# Remote state configuration for AWS

terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "production/networking.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-state-locks"

    # Optional: assume a role for cross-account access
    role_arn       = "arn:aws:iam::123456789012:role/TerraformStateAccess"
  }
}
```

Terraform Cloud and Terraform Enterprise provide managed remote state with additional features like policy enforcement, cost estimation, and a web UI for reviewing runs. For organizations that want to avoid managing state infrastructure themselves, these offerings simplify operations while adding governance capabilities.

### State Isolation Patterns

How you organize state across your infrastructure significantly impacts risk and manageability. A single state file for all infrastructure is simple but dangerous—a problem in one area affects everything. Completely separate state for every resource is safe but makes it hard for resources to reference each other.

Most organizations find a middle ground based on blast radius and team ownership. Environment separation means production state is completely isolated from development state. A mistake in development cannot affect production. Within an environment, state might be organized by team, application, or infrastructure layer.

State can be shared between configurations using data sources. If your networking configuration needs the VPC ID from another state file, it can read that state file as a data source. This creates a dependency without combing the configurations. Changes to the VPC configuration can proceed independently of changes to the networking configuration.

---

## Testing Infrastructure Code

### Why Test Infrastructure?

Testing infrastructure code might seem unnecessary—doesn't the code either work or not? But the same arguments that justify testing application code apply to infrastructure. Tests catch errors before they affect production. Tests document expected behavior. Tests enable confident refactoring. Tests verify that changes do what you intend.

Infrastructure testing is particularly valuable because infrastructure changes can be costly to reverse. An application bug might cause incorrect data, but infrastructure bugs can cause outages, security breaches, or significant cloud bills. The asymmetry between the cost of prevention and the cost of remediation makes testing highly valuable.

Infrastructure tests also serve a documentation purpose. When you read a test, you learn how the infrastructure is supposed to behave. This complements the code itself, which shows what the infrastructure is, not what it should do. Tests capture intent in a way that code alone cannot.

### Static Analysis and Linting

The fastest tests are those that don't require running anything. Static analysis tools examine your code and identify problems without executing it. They catch issues like deprecated syntax, missing required fields, and style violations.

Terraform includes built-in validation through `terraform validate`, which checks syntax and basic consistency. TFLint adds additional rules covering best practices and provider-specific recommendations. It catches issues like using deprecated attributes, missing required tags, or instance types that don't exist.

Security scanning tools like Checkov, tfsec, and Snyk examine your code for security issues. They check for overly permissive security groups, unencrypted storage, missing logging configurations, and hundreds of other security best practices. Running these scans in CI ensures that security issues are caught before they reach production.

```bash
# Example of static analysis in a CI pipeline

# Format check
terraform fmt -check -recursive

# Validate syntax
terraform init -backend=false
terraform validate

# Lint for best practices
tflint --recursive

# Security scan
checkov -d . --framework terraform
```

### Integration and End-to-End Testing

Static analysis catches many issues but cannot verify that infrastructure actually works correctly when deployed. Integration tests deploy infrastructure to a real (usually temporary) environment and verify it behaves as expected.

Terratest is a Go library designed for this purpose. You write Go tests that call Terraform to deploy infrastructure, then verify the results by making API calls, SSH connections, or HTTP requests. After verification, the test destroys the infrastructure to avoid ongoing costs.

```go
// Example Terratest integration test

func TestVpcModule(t *testing.T) {
    t.Parallel()

    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
        TerraformDir: "../modules/vpc",
        Vars: map[string]interface{}{
            "name":               "test",
            "cidr":               "10.0.0.0/16",
            "availability_zones": []string{"us-west-2a", "us-west-2b"},
        },
    })

    // Clean up at the end
    defer terraform.Destroy(t, terraformOptions)

    // Deploy infrastructure
    terraform.InitAndApply(t, terraformOptions)

    // Verify outputs
    vpcId := terraform.Output(t, terraformOptions, "vpc_id")
    assert.NotEmpty(t, vpcId)

    privateSubnets := terraform.OutputList(t, terraformOptions, "private_subnet_ids")
    assert.Equal(t, 2, len(privateSubnets))

    // Verify VPC exists and is configured correctly
    vpc := aws.GetVpcById(t, vpcId, "us-west-2")
    assert.Equal(t, "10.0.0.0/16", aws.GetCidrBlockForVpc(t, vpc))
}
```

Integration tests are slower and more expensive than static analysis but catch issues that static analysis cannot. Use them for modules that will be widely reused or that provision critical infrastructure. Run them on every change to shared modules but perhaps only periodically for application-specific configurations.

---

## Building IaC Pipelines

### Pipeline Architecture

Infrastructure changes should flow through a pipeline just like application code changes. The pipeline validates, plans, and applies changes in a controlled, auditable way. It enforces policies, requires approvals, and produces artifacts that document exactly what was changed.

A typical IaC pipeline begins with validation stages that run quickly and catch obvious problems. Formatting checks ensure code meets style standards. Validation confirms syntax is correct. Linting catches best practice violations. Security scans identify vulnerabilities. These stages provide fast feedback, usually completing in seconds to minutes.

After validation comes planning. The pipeline runs `terraform plan` to compute what changes will be made. This plan is stored as an artifact and, for pull requests, posted as a comment so reviewers can see exactly what will change. The plan stage requires access to current state but doesn't modify anything.

Application comes last and only for approved changes. For production environments, application typically requires manual approval after plan review. The pipeline applies the exact plan that was reviewed, ensuring that what was approved is what gets deployed. After application, the pipeline might run verification tests to confirm the infrastructure works correctly.

### Implementing with GitHub Actions

```yaml
# .github/workflows/terraform.yml

name: Terraform Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  pull-requests: write
  id-token: write  # For OIDC authentication

env:
  TF_VERSION: "1.6.0"
  AWS_REGION: "us-west-2"

jobs:
  validate:
    name: Validate
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Check formatting
        run: terraform fmt -check -recursive

      - name: Initialize (no backend)
        run: terraform init -backend=false

      - name: Validate syntax
        run: terraform validate

      - name: Setup TFLint
        uses: terraform-linters/setup-tflint@v4

      - name: Run TFLint
        run: tflint --recursive

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: .
          framework: terraform
          soft_fail: false

  plan:
    name: Plan
    needs: [validate, security]
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/TerraformRole
          aws-region: ${{ env.AWS_REGION }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Initialize
        run: terraform init

      - name: Plan
        id: plan
        run: terraform plan -no-color -out=plan.tfplan
        continue-on-error: true

      - name: Post plan to PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const output = `#### Terraform Plan 📖
            \`\`\`
            ${{ steps.plan.outputs.stdout }}
            \`\`\`
            `;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            })

      - name: Upload plan
        uses: actions/upload-artifact@v4
        with:
          name: terraform-plan
          path: plan.tfplan

  apply:
    name: Apply
    needs: [plan]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: production  # Requires approval
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/TerraformRole
          aws-region: ${{ env.AWS_REGION }}

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Initialize
        run: terraform init

      - name: Download plan
        uses: actions/download-artifact@v4
        with:
          name: terraform-plan

      - name: Apply
        run: terraform apply -auto-approve plan.tfplan
```

### Handling Multiple Environments

Real-world infrastructure spans multiple environments—development, staging, and production at minimum. The pipeline must handle these environments appropriately, with different policies for each.

Development environments might allow automatic deployment without approval. Changes merge and deploy immediately, enabling rapid iteration. Staging environments might require the same approval as production but deploy automatically after approval. Production requires explicit approval from designated reviewers.

Environment-specific variables customize the infrastructure for each environment. The same Terraform code might provision smaller instances in development, larger ones in staging, and full-scale resources in production. Variable files (terraform.tfvars) contain the environment-specific values.

---

## Best Practices and Common Patterns

### Code Organization

How you organize IaC code significantly impacts maintainability as your infrastructure grows. The patterns that work for a single project may not scale to an enterprise with hundreds of applications.

A common pattern separates modules from environments. Modules live in a modules directory (or a separate repository) and contain reusable infrastructure components. Environments directory contains the configurations for each deployment environment, calling modules with environment-specific parameters.

```
infrastructure/
├── modules/                    # Reusable components
│   ├── networking/
│   │   └── vpc/
│   ├── compute/
│   │   ├── ec2/
│   │   └── eks/
│   ├── database/
│   │   └── rds/
│   └── security/
│       └── security-groups/
├── environments/               # Deployable configurations
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
└── global/                     # Cross-environment resources
    ├── iam/
    └── dns/
```

### Security Practices

Security in IaC requires attention at multiple levels. The code itself should not contain secrets—no hardcoded passwords, API keys, or certificates. Secrets should come from secret management systems like AWS Secrets Manager, HashiCorp Vault, or environment variables injected by the CI system.

Access to Terraform state should be tightly controlled. The state file contains sensitive information including resource IDs, network configurations, and sometimes secret values. Use IAM policies (for S3 backend) or workspace permissions (for Terraform Cloud) to limit who can read and write state.

The CI/CD system needs credentials to deploy infrastructure, but these credentials should be limited in scope and duration. OIDC authentication allows GitHub Actions to assume AWS roles without storing long-lived credentials. The roles should have only the permissions needed for the specific infrastructure they manage.

### Drift Detection and Remediation

Configuration drift occurs when the real infrastructure diverges from the IaC configuration. Someone might make a manual change in the console, or an automated process might modify resources. Drift is problematic because it means your code no longer accurately describes your infrastructure.

Regular plan runs detect drift. Running `terraform plan` on a schedule (not just on code changes) reveals when infrastructure has drifted from the expected state. The plan shows what Terraform would change to bring reality back in line with the code.

Automatic drift remediation is more controversial. Some organizations automatically apply drift corrections, ensuring infrastructure always matches code. Others prefer manual review, as "drift" might actually be an intentional change that should be incorporated into the code rather than overwritten.

---

## Review Questions

1. **Comparing Approaches**: What are the key differences between declarative and imperative infrastructure management? Provide examples of when each approach is most appropriate.

2. **Tool Selection**: Your organization operates workloads on AWS, Azure, and on-premises VMware. How would you structure your IaC tooling strategy? What tools would you use for which purposes?

3. **Module Design**: You need to create a Terraform module for a web application stack including load balancer, auto-scaling group, and database. What variables would you expose? What outputs would you provide? How would you handle environment-specific customization?

4. **State Recovery**: A team member accidentally deleted the Terraform state file for a production environment. The infrastructure still exists and is running. What steps would you take to recover? How can this situation be prevented?

5. **Testing Strategy**: Design a comprehensive testing strategy for a module that creates a Kubernetes cluster. What tests would you run at each stage of the pipeline?

6. **Security Architecture**: How would you secure your IaC pipeline? Consider secrets management, authentication, authorization, and audit logging.

---

## Key Takeaways

Infrastructure as Code transforms infrastructure management from a manual craft into an engineering discipline. The principles of version control, testing, and automation that made software development reliable now apply to infrastructure.

Choosing the right tools matters, but principles matter more. Whether you use Terraform, CloudFormation, or Pulumi, the benefits come from applying declarative configuration, version control, and automated deployment consistently.

Modules are the key to scaling IaC. Well-designed modules encapsulate complexity, enable reuse, and ensure consistency. Invest in module design and documentation to maximize their value.

State management deserves serious attention. Lost or corrupted state creates difficult recovery situations. Use remote state with locking, encryption, and appropriate access controls from the start.

Testing catches problems before they become expensive. Static analysis is fast and cheap—run it on every change. Integration testing is slower but catches issues that static analysis cannot.

Pipelines bring governance to infrastructure changes. They ensure that all changes follow the same process: validated, planned, reviewed, and applied with appropriate approvals.

---

## Summary

Infrastructure as Code is foundational to modern infrastructure management. It brings software engineering rigor—version control, testing, code review, automated deployment—to infrastructure operations that were traditionally manual and error-prone. Organizations that adopt IaC effectively achieve dramatic improvements in speed, consistency, and reliability.

Success with IaC requires more than just learning tools. It requires adopting new ways of thinking about infrastructure: as code to be versioned and reviewed, as systems to be tested and validated, as artifacts to be deployed through pipelines. It requires building new skills on teams and new processes in organizations.

The investment pays substantial dividends. Teams that once spent weeks provisioning environments now do it in minutes. Changes that once required heroic effort and careful timing now happen routinely through automated pipelines. Infrastructure that once drifted into mysterious states now matches documented configurations.

The journey from manual infrastructure to mature IaC practices takes time, but every step delivers value. Start with version control for existing configurations. Add validation and security scanning. Build out testing gradually. Implement pipelines for controlled deployment. Each improvement builds on the last, creating compounding benefits over time.

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 7: High Availability](/InfrastructureAndPlatformManagementHandbook/chapters/07-high-availability/) | [Chapter 9: Container Platforms](/InfrastructureAndPlatformManagementHandbook/chapters/09-container-platforms/) |
