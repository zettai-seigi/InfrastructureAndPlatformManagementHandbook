---
layout: default
title: "Chapter 13: Patch Management and Maintenance"
parent: "Part IV: Operations and Management"
nav_order: 13
permalink: /chapters/13-patch-management/
---

# Chapter 13: Patch Management and Maintenance

## Learning Objectives

After completing this chapter, you will be able to:
- Implement effective patch management processes
- Manage vulnerability remediation
- Plan and execute maintenance windows
- Integrate patching with change management
- Automate patching operations
- Balance security urgency with operational stability

---

## Introduction

Patch management is essential for maintaining security and stability. Unpatched systems are vulnerable to exploitation and may lack important fixes. The 2017 Equifax breach, which exposed 147 million records, resulted from an unpatched vulnerability that had a fix available for two months. Effective patch management balances security urgency with operational stability.

This chapter covers the processes, tools, and practices for managing patches across infrastructure, from vulnerability identification through deployment and verification.

---

## Patch Management Process

### Process Lifecycle

```
+-----------------------------------------------------------------------+
|                    PATCH MANAGEMENT LIFECYCLE                          |
+-----------------------------------------------------------------------+
|                                                                        |
|  +----------+     +----------+     +----------+     +----------+      |
|  | IDENTIFY |---->| ASSESS   |---->|   PLAN   |---->|   TEST   |      |
|  |          |     |          |     |          |     |          |      |
|  | - Vendor |     | - Risk   |     | - Schedule    | - Validate|      |
|  | - CVE    |     | - Impact |     | - Resources   | - Staging |      |
|  | - Scans  |     | - Urgency|     | - Rollback    | - Compat  |      |
|  +----------+     +----------+     +----------+     +----------+      |
|       ^                                                   |            |
|       |                                                   v            |
|       |                                            +----------+       |
|       |           Continuous Cycle                 |  DEPLOY  |       |
|       |                                            |          |       |
|       +<-------------------------------------------| - Execute|       |
|       |                                            | - Monitor|       |
|       |                                            | - Verify |       |
|       |                                            +----+-----+       |
|       |                                                 |             |
|       |                                                 v             |
|       |                                            +----------+       |
|       +<-------------------------------------------| DOCUMENT |       |
|                                                    |          |       |
|                                                    | - Record |       |
|                                                    | - Report |       |
|                                                    +----------+       |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Process Stages

| Stage | Activities | Outputs | Responsible |
|-------|------------|---------|-------------|
| Identify | Monitor advisories, scan systems | Patch inventory | Security team |
| Assess | Evaluate risk, applicability | Prioritized list | Security + Ops |
| Plan | Schedule, prepare rollback | Deployment plan | Operations |
| Test | Validate in non-production | Test results | QA + Ops |
| Deploy | Execute deployment | Deployed patches | Operations |
| Verify | Confirm success, check stability | Verification report | Operations |
| Document | Record activities, update CMDB | Compliance records | Operations |

### Patch Classifications

| Classification | SLA | CVSS Score | Examples | Response |
|----------------|-----|------------|----------|----------|
| Critical | 72 hours | 9.0-10.0 | Zero-day exploits, active attacks | Emergency deployment |
| High | 7 days | 7.0-8.9 | Remotely exploitable, network | Priority scheduling |
| Medium | 30 days | 4.0-6.9 | Local vulnerabilities, limited | Standard cycle |
| Low | 90 days | 0.1-3.9 | Minor issues, unlikely exploit | Next maintenance |

---

## Vulnerability Management

### Vulnerability Lifecycle

```
+-----------------------------------------------------------------------+
|                 VULNERABILITY MANAGEMENT LIFECYCLE                     |
+-----------------------------------------------------------------------+
|                                                                        |
|  +----------+     +----------+     +----------+     +----------+      |
|  | DISCOVER |---->|  ASSESS  |---->|PRIORITIZE|---->|REMEDIATE |      |
|  +----------+     +----------+     +----------+     +----------+      |
|       |                |                |                |             |
|       v                v                v                v             |
|  +---------+      +---------+      +---------+      +---------+       |
|  | - Scans |      | - CVSS  |      | - Risk  |      | - Patch |       |
|  | - Feeds |      | - Asset |      | - Effort|      | - Config|       |
|  | - Audit |      | - Context|     | - SLA   |      | - Accept|       |
|  +---------+      +---------+      +---------+      +---------+       |
|                                                           |            |
|                                                           v            |
|                                                      +----------+     |
|                                                      |  VERIFY  |     |
|                                                      |          |     |
|                                                      | - Rescan |     |
|                                                      | - Report |     |
|                                                      +----------+     |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Vulnerability Scanning

| Scan Type | Frequency | Coverage | Tools |
|-----------|-----------|----------|-------|
| Network Scan | Weekly | External-facing | Nessus, Qualys |
| Authenticated Scan | Weekly | Internal systems | Nessus, Rapid7 |
| Container Scan | Per-build | Container images | Trivy, Clair |
| Dependency Scan | Per-commit | Application code | Snyk, Dependabot |
| Cloud Config Scan | Daily | Cloud resources | Prowler, ScoutSuite |

### Prioritization Factors

| Factor | Weight | Description | Example |
|--------|--------|-------------|---------|
| CVSS Score | High | Severity rating | CVSS 9.8 = Critical |
| Exploitability | High | Known exploits exist | EPSS score > 0.5 |
| Exposure | High | Internet-facing | Public web server |
| Asset Value | Medium | Business criticality | Production database |
| Compensating Controls | Medium | Mitigations in place | WAF protecting |
| Patch Availability | Medium | Fix available | Vendor patch released |

### Vulnerability Response Matrix

```
+-----------------------------------------------------------------------+
|                    VULNERABILITY RESPONSE MATRIX                       |
+-----------------------------------------------------------------------+
|                                                                        |
|               |  High Exploitability  |  Low Exploitability  |         |
|               |  (Active Exploits)    |  (Theoretical)       |         |
|  -------------+-----------------------+----------------------+         |
|               |                       |                      |         |
|  High Asset   |    IMMEDIATE          |     PRIORITY         |         |
|  Value        |    (< 72 hours)       |     (< 7 days)       |         |
|               |                       |                      |         |
|  -------------+-----------------------+----------------------+         |
|               |                       |                      |         |
|  Low Asset    |    PRIORITY           |     STANDARD         |         |
|  Value        |    (< 7 days)         |     (< 30 days)      |         |
|               |                       |                      |         |
+-----------------------------------------------------------------------+
```

---

## Maintenance Windows

### Window Types

| Type | Purpose | Duration | Frequency | Notification |
|------|---------|----------|-----------|--------------|
| Standard | Routine maintenance | 2-4 hours | Weekly/Monthly | 7 days |
| Emergency | Critical fixes | 1-2 hours | As needed | ASAP |
| Extended | Major upgrades | 4-8 hours | Quarterly | 14 days |
| Rolling | Zero-downtime updates | Continuous | Ongoing | Informational |

### Maintenance Planning

| Element | Description | Checklist |
|---------|-------------|-----------|
| Scope | What systems affected | List all affected systems |
| Impact | Expected user impact | Downtime estimate, workarounds |
| Duration | Start and end time | Include buffer for issues |
| Dependencies | Related systems | Notify dependent teams |
| Rollback | Recovery procedure | Test rollback before starting |
| Communication | Notification plan | Stakeholders, customers |

### Maintenance Window Checklist

```markdown
## Pre-Maintenance Checklist

### 24 Hours Before
- [ ] Confirm maintenance window approved
- [ ] Verify rollback procedures tested
- [ ] Notify stakeholders
- [ ] Update status page
- [ ] Confirm team availability

### 2 Hours Before
- [ ] Final backup completed
- [ ] Verify monitoring in place
- [ ] Confirm communication channels ready
- [ ] Pre-stage patches/updates
- [ ] Review change ticket

### Start of Window
- [ ] Begin maintenance communication
- [ ] Update status page to "In Progress"
- [ ] Verify backup accessibility
- [ ] Start timer for duration tracking

## During Maintenance
- [ ] Follow documented procedure
- [ ] Update status page every 30 minutes
- [ ] Log all changes made
- [ ] Monitor system health
- [ ] Test functionality after each change

## Post-Maintenance Checklist
- [ ] Verify all services operational
- [ ] Run health checks
- [ ] Update status page to "Resolved"
- [ ] Send completion notification
- [ ] Update change ticket
- [ ] Document lessons learned
```

---

## Automation

### Automated Patching Strategies

| Approach | Description | Use Case | Pros | Cons |
|----------|-------------|----------|------|------|
| Scheduled | Auto-deploy at set times | Non-critical systems | Consistent | Less control |
| Rolling | Gradual deployment | Large fleets | Low risk | Slower |
| Immutable | Replace with patched images | Cloud-native | Clean state | Requires image pipeline |
| Blue-Green | Parallel environments | Production systems | Zero downtime | Double infrastructure |

### Ansible Patching Playbook

```yaml
---
# patch_servers.yml - Automated patching playbook
- name: Patch Linux servers
  hosts: all
  become: yes
  serial: "{{ batch_size | default('25%') }}"
  max_fail_percentage: 10

  vars:
    reboot_required: false
    pre_patch_services:
      - nginx
      - application

  tasks:
    - name: Check for available updates
      package_facts:
        manager: auto

    - name: Create pre-patch backup
      command: /opt/scripts/backup-config.sh
      changed_when: false

    - name: Update package cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Upgrade all packages (Debian/Ubuntu)
      apt:
        upgrade: safe
        autoremove: yes
        autoclean: yes
      register: apt_result
      when: ansible_os_family == "Debian"

    - name: Upgrade all packages (RHEL/CentOS)
      yum:
        name: '*'
        state: latest
        security: yes
      register: yum_result
      when: ansible_os_family == "RedHat"

    - name: Check if reboot required (Debian)
      stat:
        path: /var/run/reboot-required
      register: reboot_required_file
      when: ansible_os_family == "Debian"

    - name: Check if reboot required (RHEL)
      command: needs-restarting -r
      register: reboot_required_rhel
      failed_when: false
      changed_when: false
      when: ansible_os_family == "RedHat"

    - name: Set reboot flag
      set_fact:
        reboot_required: true
      when: >
        (ansible_os_family == "Debian" and reboot_required_file.stat.exists) or
        (ansible_os_family == "RedHat" and reboot_required_rhel.rc == 1)

    - name: Drain node from load balancer
      uri:
        url: "http://{{ lb_api }}/drain/{{ inventory_hostname }}"
        method: POST
      when: reboot_required and lb_api is defined
      delegate_to: localhost

    - name: Wait for connections to drain
      wait_for:
        timeout: 60
      when: reboot_required

    - name: Reboot server
      reboot:
        msg: "Reboot for patching"
        reboot_timeout: 600
        pre_reboot_delay: 5
        post_reboot_delay: 30
      when: reboot_required

    - name: Wait for services to start
      wait_for:
        port: "{{ item }}"
        state: started
        timeout: 300
      loop:
        - 22
        - 80
        - 443

    - name: Verify services running
      service:
        name: "{{ item }}"
        state: started
      loop: "{{ pre_patch_services }}"

    - name: Run health check
      uri:
        url: "http://localhost/health"
        status_code: 200
        timeout: 30
      register: health_check
      retries: 5
      delay: 10

    - name: Re-enable in load balancer
      uri:
        url: "http://{{ lb_api }}/enable/{{ inventory_hostname }}"
        method: POST
      when: lb_api is defined
      delegate_to: localhost

    - name: Record patch completion
      lineinfile:
        path: /var/log/patch-history.log
        line: "{{ ansible_date_time.iso8601 }} - Patch completed successfully"
        create: yes

  handlers:
    - name: Rollback on failure
      command: /opt/scripts/rollback-config.sh
      listen: "rollback"
```

### AWS Systems Manager Patching

```yaml
# AWS SSM Patch Baseline
Resources:
  LinuxPatchBaseline:
    Type: AWS::SSM::PatchBaseline
    Properties:
      Name: Linux-Security-Patches
      Description: Security patches for Linux servers
      OperatingSystem: AMAZON_LINUX_2
      PatchGroups:
        - Production
        - Staging
      ApprovalRules:
        PatchRules:
          - ApproveAfterDays: 7
            ComplianceLevel: CRITICAL
            EnableNonSecurity: false
            PatchFilterGroup:
              PatchFilters:
                - Key: SEVERITY
                  Values:
                    - Critical
                    - Important
                - Key: CLASSIFICATION
                  Values:
                    - Security
          - ApproveAfterDays: 14
            ComplianceLevel: HIGH
            PatchFilterGroup:
              PatchFilters:
                - Key: SEVERITY
                  Values:
                    - Medium
                    - Low

  PatchMaintenanceWindow:
    Type: AWS::SSM::MaintenanceWindow
    Properties:
      Name: Weekly-Patch-Window
      Schedule: cron(0 4 ? * SUN *)
      Duration: 4
      Cutoff: 1
      AllowUnassociatedTargets: false

  PatchMaintenanceTarget:
    Type: AWS::SSM::MaintenanceWindowTarget
    Properties:
      WindowId: !Ref PatchMaintenanceWindow
      ResourceType: INSTANCE
      Targets:
        - Key: tag:Environment
          Values:
            - Production

  PatchTask:
    Type: AWS::SSM::MaintenanceWindowTask
    Properties:
      WindowId: !Ref PatchMaintenanceWindow
      TaskArn: AWS-RunPatchBaseline
      TaskType: RUN_COMMAND
      Priority: 1
      MaxConcurrency: 25%
      MaxErrors: 10%
      Targets:
        - Key: WindowTargetIds
          Values:
            - !Ref PatchMaintenanceTarget
      TaskInvocationParameters:
        MaintenanceWindowRunCommandParameters:
          Parameters:
            Operation:
              - Install
            RebootOption:
              - RebootIfNeeded
```

---

## Compliance and Reporting

### Patch Compliance Dashboard

```
+-----------------------------------------------------------------------+
|                    PATCH COMPLIANCE DASHBOARD                          |
+-----------------------------------------------------------------------+
|                                                                        |
|  OVERALL COMPLIANCE: 94.2%                                            |
|  ========================================================             |
|                                                                        |
|  By Severity:                    By Environment:                       |
|  +------------------------+      +------------------------+            |
|  | Critical: 100%  ███████|      | Production: 96%  █████ |            |
|  | High:      98%  █████  |      | Staging:    92%  ████  |            |
|  | Medium:    91%  ████   |      | Dev:        88%  ████  |            |
|  | Low:       85%  ███    |      | Test:       95%  █████ |            |
|  +------------------------+      +------------------------+            |
|                                                                        |
|  Overdue Patches:                Upcoming Windows:                     |
|  +------------------------+      +------------------------+            |
|  | Critical: 0            |      | Sun 04:00 - Production |            |
|  | High: 3 (2 days over)  |      | Wed 02:00 - Staging    |            |
|  | Medium: 12             |      | Fri 22:00 - Dev        |            |
|  +------------------------+      +------------------------+            |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Compliance Metrics

| Metric | Target | Measurement | Reporting |
|--------|--------|-------------|-----------|
| Critical patch compliance | 100% within SLA | Systems patched / Systems needing | Daily |
| High patch compliance | 95% within SLA | Systems patched / Systems needing | Weekly |
| Mean time to patch | < SLA | Average deployment time | Monthly |
| Patch failure rate | < 5% | Failed / Total attempts | Monthly |
| Reboot success rate | > 99% | Successful / Total reboots | Monthly |

---

## Tools and Technologies

### Patch Management Tools

| Category | Tools | Use Case |
|----------|-------|----------|
| Vulnerability Scanning | Qualys, Nessus, Rapid7 | Discovery and assessment |
| Patch Management | WSUS, SCCM, Jamf | Windows/Mac management |
| Configuration Management | Ansible, Puppet, Chef | Linux/cross-platform |
| Cloud-Native | AWS SSM, Azure Update | Cloud infrastructure |
| Container | Trivy, Clair, Snyk | Container images |

---

## Review Questions

1. What factors determine patch priority beyond CVSS score?
2. How should emergency patches be handled differently from routine patches?
3. What are the benefits and risks of automated patching?
4. How do you maintain compliance while minimizing operational disruption?
5. What should be included in a maintenance window communication?

---

## Summary

Effective patch management maintains security while ensuring operational stability:

- **Process** provides structured approach to patching
- **Prioritization** ensures critical vulnerabilities addressed first
- **Maintenance windows** enable controlled deployments
- **Automation** ensures consistency and reduces manual effort
- **Compliance** tracking demonstrates security posture

---

## Key Takeaways

- **Patch management** maintains security and stability through systematic updates
- **Vulnerability management** prioritizes remediation based on risk
- **Maintenance windows** enable controlled changes with stakeholder awareness
- **Automation** ensures consistent, timely patching across infrastructure
- **Compliance tracking** demonstrates security posture and identifies gaps

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 12: Incident Response](/InfrastructureAndPlatformManagementHandbook/chapters/12-incident-response/) | [Chapter 14: Capacity and Performance](/InfrastructureAndPlatformManagementHandbook/chapters/14-capacity-performance/) |
