---
layout: default
title: "Chapter 14: Capacity and Performance Management"
parent: "Part IV: Operations and Management"
nav_order: 14
permalink: /chapters/14-capacity-performance/
---

# Chapter 14: Capacity and Performance Management

## Learning Objectives

After completing this chapter, you will be able to:
- Plan and manage infrastructure capacity
- Monitor and optimize performance
- Implement auto-scaling strategies
- Right-size infrastructure resources
- Conduct capacity forecasting
- Balance cost efficiency with performance requirements

---

## Introduction

Capacity management ensures infrastructure can meet current and future demand. Performance management optimizes infrastructure to deliver required service levels. Together, they balance resource utilization with service quality, preventing both over-provisioning (waste) and under-provisioning (poor performance).

Organizations that excel at capacity management avoid costly emergency scaling, prevent performance degradation, and optimize infrastructure costs. This chapter covers the processes, metrics, and practices for effective capacity and performance management.

---

## Capacity Planning

### Capacity Planning Process

```
+-----------------------------------------------------------------------+
|                    CAPACITY PLANNING CYCLE                             |
+-----------------------------------------------------------------------+
|                                                                        |
|  +----------+     +----------+     +----------+     +----------+      |
|  | ANALYZE  |---->| FORECAST |---->|   PLAN   |---->|IMPLEMENT |      |
|  |          |     |          |     |          |     |          |      |
|  | - Current|     | - Trends |     | - Options|     | - Deploy |      |
|  | - Usage  |     | - Growth |     | - Budget |     | - Test   |      |
|  | - Trends |     | - Events |     | - Timeline    | - Verify |      |
|  +----------+     +----------+     +----------+     +----------+      |
|       ^                                                   |            |
|       |                                                   v            |
|       |                                            +----------+       |
|       +<-------------------------------------------| MONITOR  |       |
|                                                    |          |       |
|                    Continuous Loop                 | - Usage  |       |
|                                                    | - Alerts |       |
|                                                    | - Review |       |
|                                                    +----------+       |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Capacity Planning Phases

| Phase | Activities | Outputs | Frequency |
|-------|------------|---------|-----------|
| Analyze | Review current usage, identify trends | Utilization report | Monthly |
| Forecast | Predict future requirements | Demand forecast | Quarterly |
| Plan | Determine capacity changes | Capacity plan | Quarterly |
| Implement | Deploy capacity changes | Updated infrastructure | As needed |
| Monitor | Track usage against plan | Variance report | Continuous |

### Capacity Metrics

| Category | Metrics | Healthy Range | Alert Threshold |
|----------|---------|---------------|-----------------|
| Compute | CPU utilization | 40-70% | > 80% |
| Compute | CPU ready time (VMs) | < 5% | > 10% |
| Memory | RAM utilization | 50-80% | > 90% |
| Memory | Swap usage | < 5% | > 20% |
| Storage | Capacity used | 40-70% | > 80% |
| Storage | IOPS utilization | 40-70% | > 85% |
| Storage | Throughput | < 70% max | > 85% |
| Network | Bandwidth utilization | 30-50% | > 70% |
| Application | Connection pool usage | 50-70% | > 85% |

---

## Performance Management

### Performance Metrics Framework

```
+-----------------------------------------------------------------------+
|                    PERFORMANCE METRICS HIERARCHY                       |
+-----------------------------------------------------------------------+
|                                                                        |
|  BUSINESS METRICS                                                      |
|  +-------------------------------------------------------------------+|
|  | - Transaction success rate    - Revenue per hour                   ||
|  | - User satisfaction (Apdex)   - Customer conversions               ||
|  +-------------------------------------------------------------------+|
|                               |                                        |
|                               v                                        |
|  APPLICATION METRICS                                                   |
|  +-------------------------------------------------------------------+|
|  | - Response time (P50/P95/P99)  - Throughput (RPS)                 ||
|  | - Error rate                   - Apdex score                       ||
|  +-------------------------------------------------------------------+|
|                               |                                        |
|                               v                                        |
|  INFRASTRUCTURE METRICS                                                |
|  +-------------------------------------------------------------------+|
|  | - CPU utilization    - Memory usage    - Disk I/O                  ||
|  | - Network throughput - Connection counts - Queue depths            ||
|  +-------------------------------------------------------------------+|
|                                                                        |
+-----------------------------------------------------------------------+
```

### Key Performance Indicators

| Metric | Target | Measurement | Impact |
|--------|--------|-------------|--------|
| Response Time (P50) | < 100ms | Median request time | User experience |
| Response Time (P95) | < 500ms | 95th percentile | Consistency |
| Response Time (P99) | < 1000ms | 99th percentile | Tail latency |
| Throughput | Varies | Requests per second | Capacity |
| Error Rate | < 0.1% | Failed requests | Reliability |
| Availability | > 99.9% | Uptime percentage | SLA compliance |

### Apdex Score

Application Performance Index (Apdex) provides a standardized measure of user satisfaction:

```
Apdex = (Satisfied + (Tolerating × 0.5)) / Total Samples

Where:
- Satisfied: Response time ≤ T (threshold, e.g., 500ms)
- Tolerating: T < Response time ≤ 4T
- Frustrated: Response time > 4T
```

| Apdex Score | Rating | Action |
|-------------|--------|--------|
| 0.94 - 1.00 | Excellent | Maintain |
| 0.85 - 0.93 | Good | Monitor |
| 0.70 - 0.84 | Fair | Investigate |
| 0.50 - 0.69 | Poor | Prioritize fix |
| < 0.50 | Unacceptable | Immediate action |

---

## Performance Optimization

### Optimization Areas

| Area | Techniques | Tools |
|------|------------|-------|
| Compute | Right-sizing, instance type optimization | CloudWatch, Prometheus |
| Storage | Tiering, IOPS optimization, compression | iostat, fio |
| Network | CDN, compression, connection pooling | tcpdump, iperf |
| Database | Query optimization, indexing, caching | pg_stat, slow query log |
| Application | Code profiling, caching, async processing | APM tools, profilers |

### Database Performance Optimization

```sql
-- Identify slow queries (PostgreSQL)
SELECT
    query,
    calls,
    total_time / 1000 as total_seconds,
    mean_time / 1000 as mean_seconds,
    rows
FROM pg_stat_statements
WHERE mean_time > 100  -- queries taking > 100ms average
ORDER BY total_time DESC
LIMIT 20;

-- Check index usage
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0  -- unused indexes
ORDER BY pg_relation_size(indexrelid) DESC;

-- Table bloat analysis
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) as total_size,
    n_dead_tup as dead_tuples,
    n_live_tup as live_tuples,
    round(n_dead_tup * 100.0 / nullif(n_live_tup + n_dead_tup, 0), 2) as dead_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000
ORDER BY n_dead_tup DESC;
```

### Caching Strategies

| Cache Type | Use Case | TTL | Invalidation |
|------------|----------|-----|--------------|
| Application | Session data, computed values | Minutes to hours | On update |
| Database | Query results | Seconds to minutes | Time-based |
| CDN | Static assets | Hours to days | Version-based |
| API | Response caching | Seconds to minutes | Cache-Control headers |

```python
# Redis caching example
import redis
import json
from functools import wraps

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def cache_result(ttl_seconds=300):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Create cache key from function name and arguments
            cache_key = f"{func.__name__}:{hash(str(args) + str(kwargs))}"

            # Try to get cached result
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)

            # Execute function and cache result
            result = func(*args, **kwargs)
            redis_client.setex(cache_key, ttl_seconds, json.dumps(result))
            return result
        return wrapper
    return decorator

@cache_result(ttl_seconds=60)
def get_user_profile(user_id):
    # Expensive database query
    return database.query(f"SELECT * FROM users WHERE id = {user_id}")
```

---

## Auto-Scaling

### Scaling Types

| Type | Description | Use Case | Response Time |
|------|-------------|----------|---------------|
| Horizontal | Add/remove instances | Stateless workloads | 2-10 minutes |
| Vertical | Change instance size | Stateful workloads | Minutes (downtime) |
| Scheduled | Scale based on time | Predictable patterns | Pre-configured |
| Predictive | Scale based on forecast | ML-driven | Proactive |

### Auto-Scaling Architecture

```
+-----------------------------------------------------------------------+
|                    AUTO-SCALING ARCHITECTURE                           |
+-----------------------------------------------------------------------+
|                                                                        |
|  METRICS                         SCALING POLICIES                      |
|  +------------------+            +------------------+                   |
|  | CloudWatch       |            | Scale-Out Policy |                   |
|  | Prometheus       |----------->| - CPU > 70%      |                   |
|  | Custom Metrics   |            | - Wait: 300s     |                   |
|  +------------------+            | - Cooldown: 300s |                   |
|                                  +--------+---------+                   |
|                                           |                             |
|                                           v                             |
|                                  +------------------+                   |
|                                  | Auto Scaling     |                   |
|                                  | Group            |                   |
|                                  | Min: 2           |                   |
|                                  | Max: 20          |                   |
|                                  | Desired: 4       |                   |
|                                  +--------+---------+                   |
|                                           |                             |
|            +---------------+--------------+--------------+              |
|            |               |              |              |              |
|            v               v              v              v              |
|       +--------+      +--------+     +--------+     +--------+         |
|       |Instance|      |Instance|     |Instance|     |Instance|         |
|       |   1    |      |   2    |     |   3    |     |   4    |         |
|       +--------+      +--------+     +--------+     +--------+         |
|                                                                        |
+-----------------------------------------------------------------------+
```

### AWS Auto-Scaling Configuration

```hcl
# Terraform Auto-Scaling configuration
resource "aws_autoscaling_group" "web" {
  name                = "web-asg"
  vpc_zone_identifier = var.private_subnet_ids
  target_group_arns   = [aws_lb_target_group.web.arn]

  min_size         = 2
  max_size         = 20
  desired_capacity = 4

  health_check_type         = "ELB"
  health_check_grace_period = 300

  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }

  instance_refresh {
    strategy = "Rolling"
    preferences {
      min_healthy_percentage = 75
    }
  }

  tag {
    key                 = "Name"
    value               = "web-server"
    propagate_at_launch = true
  }
}

# Scale-out policy (add instances)
resource "aws_autoscaling_policy" "scale_out" {
  name                   = "web-scale-out"
  autoscaling_group_name = aws_autoscaling_group.web.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = 2
  cooldown               = 300
}

# Scale-out trigger
resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "web-high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 120
  statistic           = "Average"
  threshold           = 70
  alarm_actions       = [aws_autoscaling_policy.scale_out.arn]

  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.web.name
  }
}

# Scale-in policy (remove instances)
resource "aws_autoscaling_policy" "scale_in" {
  name                   = "web-scale-in"
  autoscaling_group_name = aws_autoscaling_group.web.name
  adjustment_type        = "ChangeInCapacity"
  scaling_adjustment     = -1
  cooldown               = 600
}

# Scale-in trigger
resource "aws_cloudwatch_metric_alarm" "low_cpu" {
  alarm_name          = "web-low-cpu"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = 4
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 120
  statistic           = "Average"
  threshold           = 30
  alarm_actions       = [aws_autoscaling_policy.scale_in.arn]

  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.web.name
  }
}

# Target tracking policy (alternative approach)
resource "aws_autoscaling_policy" "target_tracking" {
  name                   = "web-target-tracking"
  autoscaling_group_name = aws_autoscaling_group.web.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 50.0
  }
}
```

---

## Right-Sizing

### Right-Sizing Process

```
+-----------------------------------------------------------------------+
|                    RIGHT-SIZING WORKFLOW                               |
+-----------------------------------------------------------------------+
|                                                                        |
|  +----------+     +----------+     +----------+     +----------+      |
|  | ANALYZE  |---->| IDENTIFY |---->|RECOMMEND |---->|IMPLEMENT |      |
|  |          |     |          |     |          |     |          |      |
|  | - 30-90  |     | - Over-  |     | - Target |     | - Schedule|     |
|  |   days   |     |  prov    |     |   size   |     | - Execute|      |
|  | - Usage  |     | - Under- |     | - Savings|     | - Verify |      |
|  |   data   |     |   prov   |     | - Risk   |     |          |      |
|  +----------+     +----------+     +----------+     +----------+      |
|                                          |                             |
|                                          v                             |
|                                   +----------+                         |
|                                   | VALIDATE |                         |
|                                   |          |                         |
|                                   | - Perf   |                         |
|                                   | - Cost   |                         |
|                                   +----------+                         |
|                                                                        |
+-----------------------------------------------------------------------+
```

### Right-Sizing Recommendations

| Scenario | Indicators | Recommendation | Savings |
|----------|------------|----------------|---------|
| Over-provisioned | CPU < 30%, Memory < 50% | Downsize instance | 30-50% |
| Under-provisioned | CPU > 80%, Memory > 85% | Upsize or optimize | N/A |
| Burst workload | Low average, high peaks | Burstable instance | 20-40% |
| Steady workload | Consistent utilization | Reserved capacity | 30-60% |
| Idle resources | Near-zero utilization | Terminate or schedule | 100% |

### Cost Optimization Analysis

```python
# Right-sizing analysis script
import boto3
from datetime import datetime, timedelta

def analyze_instance_utilization(instance_id, days=30):
    """Analyze EC2 instance utilization for right-sizing."""
    cloudwatch = boto3.client('cloudwatch')
    ec2 = boto3.resource('ec2')

    instance = ec2.Instance(instance_id)
    instance_type = instance.instance_type

    end_time = datetime.utcnow()
    start_time = end_time - timedelta(days=days)

    # Get CPU metrics
    cpu_response = cloudwatch.get_metric_statistics(
        Namespace='AWS/EC2',
        MetricName='CPUUtilization',
        Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
        StartTime=start_time,
        EndTime=end_time,
        Period=3600,
        Statistics=['Average', 'Maximum']
    )

    cpu_datapoints = cpu_response['Datapoints']
    if cpu_datapoints:
        avg_cpu = sum(d['Average'] for d in cpu_datapoints) / len(cpu_datapoints)
        max_cpu = max(d['Maximum'] for d in cpu_datapoints)
    else:
        avg_cpu = max_cpu = 0

    # Generate recommendation
    recommendation = generate_recommendation(
        instance_type=instance_type,
        avg_cpu=avg_cpu,
        max_cpu=max_cpu
    )

    return {
        'instance_id': instance_id,
        'current_type': instance_type,
        'avg_cpu_utilization': round(avg_cpu, 2),
        'max_cpu_utilization': round(max_cpu, 2),
        'recommendation': recommendation
    }

def generate_recommendation(instance_type, avg_cpu, max_cpu):
    """Generate right-sizing recommendation based on utilization."""
    if avg_cpu < 20 and max_cpu < 50:
        return {
            'action': 'DOWNSIZE',
            'reason': 'Instance significantly over-provisioned',
            'confidence': 'HIGH'
        }
    elif avg_cpu < 40 and max_cpu < 70:
        return {
            'action': 'REVIEW',
            'reason': 'Instance may be over-provisioned',
            'confidence': 'MEDIUM'
        }
    elif avg_cpu > 80 or max_cpu > 95:
        return {
            'action': 'UPSIZE',
            'reason': 'Instance under-provisioned',
            'confidence': 'HIGH'
        }
    else:
        return {
            'action': 'MAINTAIN',
            'reason': 'Instance appropriately sized',
            'confidence': 'HIGH'
        }
```

---

## Capacity Forecasting

### Forecasting Methods

| Method | Description | Accuracy | Use Case |
|--------|-------------|----------|----------|
| Linear Regression | Trend-based projection | Medium | Steady growth |
| Exponential Smoothing | Weighted recent data | Medium | Seasonal patterns |
| Machine Learning | Pattern recognition | High | Complex patterns |
| Business Input | Known events | High | Launches, campaigns |

### Forecasting Dashboard

```
+-----------------------------------------------------------------------+
|                    CAPACITY FORECAST DASHBOARD                         |
+-----------------------------------------------------------------------+
|                                                                        |
|  Current Capacity: 200 vCPUs                                          |
|  Current Utilization: 65%                                              |
|  Projected Growth: 15% quarterly                                       |
|                                                                        |
|  +-------------------------------------------------------------------+|
|  |                                                                    ||
|  | Capacity                                                          ||
|  |   ^                                        ****                   ||
|  |   |                                   ****                        ||
|  |   |                              ****                             ||
|  | 300|                        ****  <-- Projected                   ||
|  |   |                   ****                                        ||
|  |   |              ****                                              ||
|  | 200|         ****  <-- Current                                    ||
|  |   |    ****                                                        ||
|  |   |****                                                            ||
|  |   +---------------------------------------------------------->    ||
|  |    Jan  Feb  Mar  Apr  May  Jun  Jul  Aug  Sep  Oct  Nov  Dec     ||
|  |                                                                    ||
|  +-------------------------------------------------------------------+|
|                                                                        |
|  Recommendations:                                                      |
|  - Q2: Add 50 vCPUs (25% increase)                                    |
|  - Q3: Review reserved capacity options                               |
|  - Q4: Plan infrastructure refresh                                    |
|                                                                        |
+-----------------------------------------------------------------------+
```

---

## Review Questions

1. What metrics should drive capacity planning decisions?
2. How do you balance auto-scaling responsiveness with cost efficiency?
3. What is the right-sizing process and when should it be performed?
4. How do you forecast capacity needs for a growing application?
5. What role does caching play in performance optimization?

---

## Summary

Effective capacity and performance management ensures infrastructure meets demand while optimizing costs:

- **Capacity planning** anticipates future needs
- **Performance management** optimizes service delivery
- **Auto-scaling** provides elastic capacity
- **Right-sizing** eliminates waste
- **Forecasting** enables proactive planning

---

## Key Takeaways

- **Capacity planning** ensures infrastructure meets current and future demand
- **Performance management** optimizes service delivery through monitoring and tuning
- **Auto-scaling** provides elastic capacity that responds to demand
- **Right-sizing** eliminates waste while maintaining performance
- **Capacity forecasting** enables proactive planning and budgeting

---

## Chapter Navigation

| Previous | Next |
|----------|------|
| [Chapter 13: Patch Management](/InfrastructureAndPlatformManagementHandbook/chapters/13-patch-management/) | [Chapter 15: Governance Framework](/InfrastructureAndPlatformManagementHandbook/chapters/15-governance-framework/) |
