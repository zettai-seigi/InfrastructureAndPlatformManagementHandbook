---
layout: default
title: "Chapter 12: Incident Response and Troubleshooting"
parent: "Part IV: Operations and Management"
nav_order: 12
permalink: /chapters/12-incident-response/
---

# Chapter 12: Incident Response and Troubleshooting

## Learning Objectives

After completing this chapter, you will be able to:
- Respond effectively to infrastructure incidents
- Apply structured troubleshooting methodologies
- Conduct root cause analysis
- Implement incident management processes
- Develop and maintain runbooks
- Lead incident response teams effectively

---

## Introduction

Infrastructure incidents impact business operations and user experience. Effective incident response minimizes impact through rapid detection, diagnosis, and resolution. The difference between a minor disruption and a major outage often comes down to how well prepared teams are to respond.

This chapter covers incident management processes, troubleshooting techniques, and the organizational practices that enable teams to resolve issues quickly and learn from them to prevent recurrence.

---

## Incident Management Process

### Incident Lifecycle

```
+-----------------------------------------------------------------------+
|                    INCIDENT LIFECYCLE                                  |
+-----------------------------------------------------------------------+
|                                                                        |
|  +----------+     +----------+     +----------+     +----------+      |
|  |  DETECT  |---->| TRIAGE   |---->| RESPOND  |---->| RECOVER  |      |
|  |          |     |          |     |          |     |          |      |
|  | - Alerts |     | - Assess |     | - Diagnose    | - Restore |      |
|  | - Users  |     | - Classify    | - Mitigate    | - Verify  |      |
|  | - Monitor|     | - Prioritize  | - Communicate | - Document|      |
|  +----------+     +----------+     +----------+     +----------+      |
|       |                                                   |            |
|       |                                                   v            |
|       |                                            +----------+       |
|       +<-------------------------------------------|  LEARN   |       |
|                                                    |          |       |
|                   Continuous Improvement           | - Review |       |
|                                                    | - Improve|       |
|                                                    | - Prevent|       |
|                                                    +----------+       |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Incident Classification

| Priority | Impact | Urgency | Response Time | Resolution Target | Examples |
|----------|--------|---------|---------------|-------------------|----------|
| P1 Critical | Business stopped | Immediate | < 15 min | 1 hour | Production down, data loss |
| P2 High | Major degradation | High | < 30 min | 4 hours | Partial outage, severe slowdown |
| P3 Medium | Limited impact | Medium | < 2 hours | 24 hours | Non-critical service affected |
| P4 Low | Minimal impact | Low | < 8 hours | 72 hours | Cosmetic issues, workarounds exist |

### Incident Severity Matrix

```
+-----------------------------------------------------------------------+
|                    INCIDENT SEVERITY MATRIX                            |
+-----------------------------------------------------------------------+
|                                                                        |
|               |   Wide Impact    |   Narrow Impact   |                 |
|               |   (Many users)   |   (Few users)     |                 |
|  -------------+------------------+-------------------+                 |
|               |                  |                   |                 |
|  High         |    CRITICAL      |      HIGH         |                 |
|  Business     |    (P1)          |      (P2)         |                 |
|  Impact       |                  |                   |                 |
|               |                  |                   |                 |
|  -------------+------------------+-------------------+                 |
|               |                  |                   |                 |
|  Low          |    MEDIUM        |      LOW          |                 |
|  Business     |    (P3)          |      (P4)         |                 |
|  Impact       |                  |                   |                 |
|               |                  |                   |                 |
+-----------------------------------------------------------------------+
```

---

## Incident Response Roles

### Incident Command System

| Role | Responsibilities | Skills Required |
|------|-----------------|-----------------|
| Incident Commander | Overall coordination, decision-making | Leadership, communication |
| Technical Lead | Technical diagnosis and resolution | Deep technical expertise |
| Communications Lead | Stakeholder updates, status page | Clear communication |
| Scribe | Document timeline, actions, decisions | Attention to detail |
| Subject Matter Expert | Domain-specific knowledge | Specialized skills |

### On-Call Responsibilities

```
+-----------------------------------------------------------------------+
|                    ON-CALL ROTATION STRUCTURE                          |
+-----------------------------------------------------------------------+
|                                                                        |
|  PRIMARY ON-CALL                                                       |
|  +-------------------------------------------------------------------+|
|  | - First responder for all alerts                                   ||
|  | - Initial triage and assessment                                    ||
|  | - Escalate if needed within 15 minutes                            ||
|  +-------------------------------------------------------------------+|
|                                |                                       |
|                                v                                       |
|  SECONDARY ON-CALL                                                     |
|  +-------------------------------------------------------------------+|
|  | - Backup if primary unavailable                                    ||
|  | - Complex issues requiring additional support                      ||
|  | - Mentoring/support for primary                                    ||
|  +-------------------------------------------------------------------+|
|                                |                                       |
|                                v                                       |
|  ESCALATION PATH                                                       |
|  +-------------------------------------------------------------------+|
|  | Team Lead → Engineering Manager → Director → VP → CTO             ||
|  | (based on severity and business impact)                            ||
|  +-------------------------------------------------------------------+|
|                                                                        |
+-----------------------------------------------------------------------+
```

---

## Troubleshooting Methodology

### Systematic Approach

| Step | Description | Activities | Tools |
|------|-------------|------------|-------|
| 1. Define | What is the actual problem? | Gather symptoms, user reports | Monitoring, logs |
| 2. Scope | What is affected? What works? | Identify blast radius | Dashboards, traces |
| 3. Hypothesize | What could cause this? | List potential causes | Experience, documentation |
| 4. Test | Verify or eliminate hypotheses | Execute tests | CLI tools, scripts |
| 5. Resolve | Implement fix | Apply solution | IaC, manual intervention |
| 6. Verify | Confirm resolution | Validate fix | Monitoring, user confirmation |
| 7. Document | Record findings | Write postmortem | Wiki, ticketing system |

### Troubleshooting Decision Tree

```
+-----------------------------------------------------------------------+
|                    TROUBLESHOOTING DECISION TREE                       |
+-----------------------------------------------------------------------+
|                                                                        |
|                    [Service Degraded]                                  |
|                           |                                            |
|                           v                                            |
|                  [Recent Changes?]                                     |
|                    /           \                                       |
|                  Yes            No                                     |
|                  /               \                                     |
|                 v                 v                                    |
|        [Rollback?]      [Resource Issue?]                             |
|         /     \           /         \                                  |
|       Yes     No        Yes          No                                |
|       /         \       /              \                               |
|      v           v     v                v                              |
|  [Execute]  [Investigate]  [Scale/     [Dependency                    |
|  [Rollback]  [Further]      Optimize]   Issue?]                       |
|                                           |                            |
|                                     [Yes/No]                           |
|                                      /    \                            |
|                                     v      v                           |
|                              [Check       [Deep                        |
|                               Deps]        Dive]                       |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Common Infrastructure Issues

| Category | Symptoms | Common Causes | Diagnostic Commands |
|----------|----------|---------------|---------------------|
| CPU | High load, slow response | Resource contention, runaway process | `top`, `htop`, `ps aux` |
| Memory | OOM kills, swapping | Memory leak, insufficient allocation | `free -m`, `vmstat`, `dmesg` |
| Disk | Slow I/O, full filesystem | Capacity, IOPS saturation | `df -h`, `iostat`, `iotop` |
| Network | Timeouts, packet loss | Congestion, misconfig, DNS | `ping`, `traceroute`, `netstat` |
| Application | Errors, crashes | Code bugs, config issues | Application logs, stack traces |

### Diagnostic Commands Reference

```bash
# System Overview
uptime                          # System load averages
top -bn1 | head -20            # Process summary
vmstat 1 5                      # Virtual memory stats
iostat -xz 1 5                  # Disk I/O stats

# CPU Analysis
mpstat -P ALL 1 5              # Per-CPU utilization
pidstat 1 5                     # Per-process CPU usage
perf top                        # Real-time CPU profiling

# Memory Analysis
free -m                         # Memory usage
cat /proc/meminfo              # Detailed memory info
ps aux --sort=-%mem | head     # Top memory consumers
slabtop                        # Kernel slab cache

# Disk Analysis
df -h                          # Filesystem usage
du -sh /* 2>/dev/null          # Directory sizes
lsof +D /path                  # Open files in directory
iotop -oa                      # I/O by process

# Network Analysis
ss -tuln                       # Listening sockets
ss -s                          # Socket statistics
netstat -i                     # Interface statistics
tcpdump -i eth0 port 80        # Packet capture

# Application Analysis
journalctl -u service-name -f  # Service logs
strace -p PID                  # System call trace
lsof -p PID                    # Open files by process
```

---

## Root Cause Analysis

### RCA Techniques

| Technique | Description | When to Use | Output |
|-----------|-------------|-------------|--------|
| 5 Whys | Ask "why" repeatedly | Simple, linear causation | Root cause statement |
| Fishbone Diagram | Categorize potential causes | Multiple contributing factors | Visual cause map |
| Timeline Analysis | Map events chronologically | Complex, multi-system issues | Event timeline |
| Fault Tree | Logic diagram of failure paths | Risk analysis, prevention | Failure tree |

### 5 Whys Example

```
+-----------------------------------------------------------------------+
|                    5 WHYS ANALYSIS EXAMPLE                             |
+-----------------------------------------------------------------------+
|                                                                        |
|  PROBLEM: Production API returned 500 errors for 30 minutes           |
|                                                                        |
|  WHY 1: Why did the API return 500 errors?                            |
|         → The database connection pool was exhausted                   |
|                                                                        |
|  WHY 2: Why was the connection pool exhausted?                        |
|         → Connections were not being released                          |
|                                                                        |
|  WHY 3: Why were connections not being released?                      |
|         → A code change introduced a connection leak                   |
|                                                                        |
|  WHY 4: Why wasn't the leak caught before production?                 |
|         → No load testing was performed on the change                  |
|                                                                        |
|  WHY 5: Why wasn't load testing performed?                            |
|         → No requirement for load testing in the deployment process    |
|                                                                        |
|  ROOT CAUSE: Missing load testing requirement in deployment process   |
|                                                                        |
|  CORRECTIVE ACTION: Add load testing to CI/CD pipeline                |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Fishbone Diagram Categories

```
+-----------------------------------------------------------------------+
|                    FISHBONE DIAGRAM                                    |
+-----------------------------------------------------------------------+
|                                                                        |
|        PEOPLE            PROCESS           TECHNOLOGY                  |
|           \                  |                  /                      |
|            \                 |                 /                       |
|             \                |                /                        |
|              \               |               /                         |
|               v              v              v                          |
|               +----------------------------+                           |
|               |       SERVICE OUTAGE       |                           |
|               +----------------------------+                           |
|               ^              ^              ^                          |
|              /               |               \                         |
|             /                |                \                        |
|            /                 |                 \                       |
|           /                  |                  \                      |
|     ENVIRONMENT         MEASUREMENT          MATERIALS                 |
|                                                                        |
|  Categories:                                                           |
|  - People: Training, staffing, communication                          |
|  - Process: Procedures, workflows, approvals                          |
|  - Technology: Hardware, software, infrastructure                     |
|  - Environment: Data center, network, external dependencies           |
|  - Measurement: Monitoring, alerting, metrics                         |
|  - Materials: Configuration, data, third-party services               |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Blameless Postmortems

| Element | Content | Example |
|---------|---------|---------|
| Summary | Brief description of incident | API outage affecting 10,000 users |
| Timeline | Chronological events | 14:23 Alert fired → 14:45 Root cause identified |
| Impact | Business and technical impact | 30 min downtime, $50K revenue loss |
| Root Cause | Underlying cause(s) | Database connection leak in v2.3.1 |
| Contributing Factors | What made it worse | Missing monitoring for connection pool |
| Actions | Remediation and prevention | Add connection pool monitoring, load testing |
| Lessons | What we learned | Importance of connection management |

**Postmortem Template:**

```markdown
# Incident Postmortem: [INCIDENT-ID]

## Summary
Brief description of what happened, when, and the impact.

## Timeline (All times UTC)
| Time | Event |
|------|-------|
| 14:23 | Alert: High error rate on API |
| 14:25 | On-call engineer acknowledged |
| 14:30 | Incident declared, commander assigned |
| 14:35 | Root cause identified: DB connection leak |
| 14:40 | Rollback initiated |
| 14:45 | Service restored |
| 15:00 | Incident resolved |

## Impact
- Duration: 22 minutes
- Users affected: ~10,000
- Revenue impact: ~$50,000
- SLA impact: 0.05% of monthly error budget consumed

## Root Cause
The deployment of v2.3.1 introduced a database connection leak
in the order processing module. Under load, connections were
exhausted within 20 minutes of deployment.

## Contributing Factors
1. No load testing for the specific change
2. Connection pool monitoring not alerting
3. Gradual rollout not used for this deployment

## Detection
Alert triggered on HTTP 500 error rate exceeding 1% threshold.

## Resolution
Rolled back to v2.3.0. Connection pool returned to normal within
2 minutes of rollback completion.

## Action Items
| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| Add connection pool metrics to dashboard | @engineer | 2024-01-20 | Done |
| Implement load testing in CI/CD | @platform | 2024-01-25 | In Progress |
| Enable gradual rollout for all deployments | @platform | 2024-01-30 | Todo |
| Review connection handling patterns | @architect | 2024-02-01 | Todo |

## Lessons Learned
- Connection management requires explicit testing under load
- Monitoring gaps in core resources create blind spots
- Gradual rollout would have limited blast radius
```

---

## Runbooks

### Runbook Structure

| Section | Content | Purpose |
|---------|---------|---------|
| Overview | What this runbook addresses | Quick identification |
| Symptoms | How to recognize this issue | Confirm applicability |
| Prerequisites | Required access, tools | Preparation |
| Diagnosis | How to confirm the issue | Validation steps |
| Resolution | Step-by-step fix | Clear instructions |
| Verification | How to confirm resolution | Success criteria |
| Escalation | When and how to escalate | Handoff guidance |
| References | Related documentation | Additional context |

### Sample Runbook: High CPU Utilization

```markdown
# Runbook: High CPU Utilization

## Overview
This runbook addresses alerts for high CPU utilization on production servers.

## Alert Trigger
- HighCPUUtilization: CPU > 85% for 10+ minutes
- CriticalCPUUtilization: CPU > 95% for 5+ minutes

## Symptoms
- Slow application response times
- Increased request latency
- Alert: "High CPU utilization on {{ instance }}"

## Prerequisites
- SSH access to affected server
- Sudo privileges
- Access to monitoring dashboards

## Diagnosis Steps

### Step 1: Verify the Alert
```bash
# Check current CPU utilization
top -bn1 | head -5

# Expected: See overall CPU utilization
# If < 70%, alert may be stale - verify in monitoring
```

### Step 2: Identify Top Consumers
```bash
# List processes by CPU usage
ps aux --sort=-%cpu | head -20

# Check for any runaway processes
# Look for processes using > 50% CPU individually
```

### Step 3: Check for Recent Changes
- Review recent deployments in deployment dashboard
- Check for configuration changes in audit log
- Verify no scheduled jobs running

## Resolution Steps

### Option A: Process-Specific Issue
```bash
# If a specific process is consuming excessive CPU:

# 1. Get process details
ps -p PID -o pid,ppid,cmd,%cpu,%mem,etime

# 2. Check if process can be safely restarted
systemctl status service-name

# 3. Restart the service
sudo systemctl restart service-name

# 4. Monitor recovery
watch -n 5 'ps aux --sort=-%cpu | head -10'
```

### Option B: Load-Based Issue
```bash
# If overall system load is high:

# 1. Check request rate
curl -s localhost:9090/metrics | grep http_requests_total

# 2. Consider scaling if load is legitimate
# Trigger auto-scaling or manual scale-up

# 3. For immediate relief, enable traffic shedding
# (if available in your load balancer)
```

### Option C: Runaway Process
```bash
# For a clearly runaway process:

# 1. Document the process state
ps -p PID -o pid,ppid,cmd,%cpu,%mem,etime > /tmp/runaway_process.txt

# 2. Capture thread dump if applicable
kill -3 PID  # For Java processes

# 3. Kill if necessary (last resort)
sudo kill -15 PID  # Graceful
sudo kill -9 PID   # Force (if -15 fails)
```

## Verification
1. CPU utilization returns below 80%
2. Application response times normalize
3. No new alerts triggered
4. Health checks passing

## Escalation
Escalate if:
- Issue persists after 30 minutes
- Root cause is unclear
- Application-level debugging required
- Multiple servers affected simultaneously

Contact: @platform-oncall or #incident-response channel
```

---

## Communication During Incidents

### Communication Timeline

| Phase | Audience | Channel | Content |
|-------|----------|---------|---------|
| Detection | On-call team | PagerDuty, Slack | Alert details |
| Triage | Engineering | Incident channel | Initial assessment |
| Response | Stakeholders | Status page, email | Impact, ETA |
| Updates | All | Status page | Progress every 15-30 min |
| Resolution | All | Status page, email | Resolution, summary |
| Postmortem | Engineering | Wiki, meeting | Full analysis |

### Status Page Updates

**Initial Update:**
```
[INVESTIGATING] Elevated error rates on API
We are investigating reports of elevated error rates
affecting the API. Some users may experience slow
responses or errors. We are working to identify
the cause.

Posted: 14:25 UTC
```

**Progress Update:**
```
[IDENTIFIED] Elevated error rates on API
We have identified the cause as a database
connection issue. Engineering is working on
a fix. We expect resolution within 30 minutes.

Posted: 14:40 UTC
```

**Resolution Update:**
```
[RESOLVED] Elevated error rates on API
The issue has been resolved. A database connection
leak was identified and fixed by rolling back a
recent deployment. All systems are now operating
normally. We apologize for any inconvenience.

Duration: 22 minutes
Posted: 15:00 UTC
```

---

## Review Questions

1. What are the key phases of the incident lifecycle?
2. How should incident priority map to response and resolution times?
3. What is the purpose of a blameless postmortem?
4. What sections should a runbook contain?
5. How often should status updates be provided during an incident?

---

## Summary

Effective incident response minimizes business impact and enables continuous improvement:

- **Incident process** provides structured response to issues
- **Clear roles** enable coordinated team response
- **Troubleshooting methodology** enables systematic problem solving
- **Root cause analysis** prevents recurrence
- **Runbooks** enable consistent, rapid response
- **Communication** keeps stakeholders informed

---

## Key Takeaways

- **Incident management** provides structured response to infrastructure issues
- **Clear roles and responsibilities** enable coordinated team response
- **Troubleshooting methodology** enables systematic problem solving
- **Root cause analysis** prevents recurrence through learning
- **Runbooks** enable consistent, rapid response to known issues

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 11: Monitoring](/InfrastructureAndPlatformManagementHandbook/chapters/11-monitoring-observability/) | [Chapter 13: Patch Management](/InfrastructureAndPlatformManagementHandbook/chapters/13-patch-management/) |
