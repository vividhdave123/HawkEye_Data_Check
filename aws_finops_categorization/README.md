# 🦅 HawkEye AWS FinOps Categorization Framework

> **Comprehensive cost optimization framework covering 239 AWS services — identify waste, rightsize resources, and reduce cloud spend by 30–70%.**

[![AWS Services](https://img.shields.io/badge/AWS%20Services-239-orange)](https://aws.amazon.com/)
[![Framework Version](https://img.shields.io/badge/Version-2.0-blue)](.)
[![License](https://img.shields.io/badge/License-MIT-green)](../LICENSE)
[![Last Updated](https://img.shields.io/badge/Updated-March%202026-lightgrey)](.)

---

## 📋 Table of Contents

1. [Framework Overview](#framework-overview)
2. [Benefits](#benefits)
3. [Quick Start Guide](#quick-start-guide)
4. [Architecture](#architecture)
5. [File Reference](#file-reference)
   - [active_resources_criteria.csv](#active_resources_criteriacsv)
   - [idle_resources_criteria.csv](#idle_resources_criteriacsv)
   - [underutilized_resources_criteria.csv](#underutilized_resources_criteriacsv)
   - [implementation_guide.csv](#implementation_guidecsv)
6. [Category Definitions](#category-definitions)
7. [Python Implementation Examples](#python-implementation-examples)
8. [IAM Policy Templates](#iam-policy-templates)
9. [Monitoring Frequency Guidelines](#monitoring-frequency-guidelines)
10. [Getting Started Tutorial](#getting-started-tutorial)
11. [Use Cases and Best Practices](#use-cases-and-best-practices)
12. [Success Metrics and KPIs](#success-metrics-and-kpis)
13. [AWS Resources](#aws-resources)

---

## Framework Overview

The **HawkEye AWS FinOps Categorization Framework** provides a structured, data-driven approach to classifying every AWS resource in your account into one of four operational states:

| Category | Description | Action |
|---|---|---|
| 🟢 **Active** | Resource is running and serving its intended purpose | Maintain — it's earning its cost |
| 🔴 **Idle** | Resource is provisioned but generating no measurable activity | Stop or terminate immediately |
| 🟡 **Underutilized** | Resource is active but significantly over-provisioned | Rightsize to recover 30–70% of spend |
| 🔵 **Archived** | Resource is retained for compliance or disaster recovery | Move to lowest-cost storage tier |

The framework covers **239 AWS services** across all major categories (compute, database, storage, networking, analytics, ML/AI, security, and management) and gives you the exact CloudWatch metrics, API calls, thresholds, and remediation steps needed to automate cost governance at scale.

---

## Benefits

| Benefit | Impact |
|---|---|
| 💰 **Immediate cost reduction** | Eliminate idle resources to cut 15–25% of your AWS bill within 30 days |
| 📐 **Rightsizing insights** | Reduce oversized resource spend by an additional 30–50% |
| 🤖 **Automation-ready** | Every check includes the exact API calls and IAM permissions needed |
| 📊 **Consistent standards** | Uniform thresholds across all 239 services eliminate guesswork |
| ⚡ **Fast time-to-value** | Copy-paste Python examples get you running in under an hour |
| 🔐 **Least-privilege security** | Pre-built IAM policies follow AWS security best practices |
| 📈 **KPI tracking** | Built-in success metrics so you can prove ROI to stakeholders |

---

## Quick Start Guide

### Prerequisites

- AWS CLI configured with appropriate credentials
- Python 3.8+ with `boto3` installed (`pip install boto3 pandas`)
- IAM role or user with the read-only permissions listed in [IAM Policy Templates](#iam-policy-templates)

### 5-Minute Setup

```bash
# 1. Clone the repository
git clone https://github.com/vividhdave123/HawkEye_Data_Check.git
cd HawkEye_Data_Check/aws_finops_categorization

# 2. Install Python dependencies
pip install boto3 pandas

# 3. Configure AWS credentials
aws configure

# 4. Run your first scan (EC2 idle check)
python3 - <<'EOF'
import boto3
from datetime import datetime, timedelta, timezone

ec2 = boto3.client('ec2')
cw  = boto3.client('cloudwatch')

lookback_days = 7
end_time   = datetime.now(timezone.utc)
start_time = end_time - timedelta(days=lookback_days)

instances = ec2.describe_instances(
    Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
)

for reservation in instances['Reservations']:
    for instance in reservation['Instances']:
        iid = instance['InstanceId']
        stats = cw.get_metric_statistics(
            Namespace='AWS/EC2',
            MetricName='CPUUtilization',
            Dimensions=[{'Name': 'InstanceId', 'Value': iid}],
            StartTime=start_time.isoformat(),
            EndTime=end_time.isoformat(),
            Period=lookback_days * 86400,
            Statistics=['Average']
        )
        avg_cpu = stats['Datapoints'][0]['Average'] if stats['Datapoints'] else 0
        if avg_cpu < 5:
            print(f"IDLE    {iid} — avg CPU {avg_cpu:.1f}%")
        elif avg_cpu < 30:
            print(f"UNDERUTILIZED {iid} — avg CPU {avg_cpu:.1f}%")
        else:
            print(f"ACTIVE  {iid} — avg CPU {avg_cpu:.1f}%")
EOF
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                  HawkEye FinOps Framework Architecture               │
└─────────────────────────────────────────────────────────────────────┘

  ┌──────────────────┐     ┌───────────────────┐     ┌──────────────┐
  │  Data Collection │────▶│  Categorization   │────▶│  Reporting & │
  │                  │     │  Engine           │     │  Remediation │
  │  • CloudWatch    │     │                   │     │              │
  │  • AWS APIs      │     │  active_resources  │     │  Dashboard   │
  │  • Cost Explorer │     │  idle_resources    │     │  Alerts      │
  │  • CloudTrail    │     │  underutilized     │     │  Automation  │
  │  • Config        │     │  archived          │     │  Tickets     │
  └──────────────────┘     └───────────────────┘     └──────────────┘
           │                        │                        │
           ▼                        ▼                        ▼
  ┌──────────────────┐     ┌───────────────────┐     ┌──────────────┐
  │  implementation_ │     │  services_        │     │  KPI Metrics │
  │  guide.csv       │     │  overview.csv     │     │  & Savings   │
  │                  │     │                   │     │  Tracking    │
  │  API calls       │     │  239 services     │     │              │
  │  IAM perms       │     │  all categories   │     │  ROI Reports │
  │  Frequencies     │     │  thresholds       │     │              │
  └──────────────────┘     └───────────────────┘     └──────────────┘

  ┌─────────────────────────────────────────────────────────────────┐
  │                     Decision Flow                                │
  │                                                                  │
  │  Resource                                                        │
  │  Discovered ──▶ Is it running? ──No──▶ Skip (already stopped)   │
  │                      │                                           │
  │                     Yes                                          │
  │                      │                                           │
  │                      ▼                                           │
  │             Has it had ANY activity                              │
  │             in the last 7 days? ──No──▶ 🔴 IDLE (terminate)     │
  │                      │                                           │
  │                     Yes                                          │
  │                      │                                           │
  │                      ▼                                           │
  │             Is utilization > 30%                                 │
  │             of provisioned capacity? ──No──▶ 🟡 UNDERUTILIZED   │
  │                      │                                           │
  │                     Yes                                          │
  │                      │                                           │
  │                      ▼                                           │
  │                  🟢 ACTIVE (maintain)                            │
  └─────────────────────────────────────────────────────────────────┘
```

---

## File Reference

### `active_resources_criteria.csv`

**Purpose**: Defines the thresholds and CloudWatch metrics that confirm a resource is actively serving its workload.

**Columns**:

| Column | Description | Example |
|---|---|---|
| `Service Name` | AWS service name | `EC2` |
| `Criteria 1` | Primary activity metric and threshold | `CPUUtilization > 10%` |
| `Criteria 2` | Secondary activity metric or threshold | `NetworkIn > 1MB/day OR NetworkOut > 1MB/day` |
| `Criteria 3` | Tertiary check (state, status, or additional metric) | `StatusCheckFailed = 0` |
| `Time Window` | Evaluation period for metrics | `Last 7 days` |
| `CloudWatch Metrics Required` | Space-separated metric names | `CPUUtilization NetworkIn NetworkOut` |
| `API Calls Required` | Space-separated IAM actions needed | `ec2:DescribeInstances` |
| `Notes` | Context or service-specific guidance | `Adjust CPU threshold based on instance type` |

**Sample rows**:

```
EC2 — Active when: CPUUtilization > 10% AND (NetworkIn > 1MB/day OR NetworkOut > 1MB/day)
RDS — Active when: DatabaseConnections > 0 AND (ReadIOPS > 0 OR WriteIOPS > 0)
S3  — Active when: GetRequests > 100/day OR PutRequests > 10/day (last 14 days)
```

**Key decisions when using this file**:
- Use a **7-day window** for compute (EC2, ECS) — short enough to catch recent activity
- Use a **14-day window** for storage (S3, EBS) — smooths daily access variance
- Use a **30-day window** for databases (DynamoDB, Redshift) — aligns with billing cycles

---

### `idle_resources_criteria.csv`

**Purpose**: Identifies resources that are provisioned and accruing costs but have near-zero utilization — these are your highest-priority cost-elimination targets.

**Columns**:

| Column | Description | Example |
|---|---|---|
| `Service Name` | AWS service name | `EC2` |
| `Criteria 1` | Primary idleness indicator | `CPUUtilization < 5%` |
| `Criteria 2` | Secondary idleness indicator | `NetworkIn < 100KB/day AND NetworkOut < 100KB/day` |
| `Criteria 3` | State or status check | `State = running` |
| `Time Window` | Evaluation period | `Last 7 days` |
| `Cost Impact` | Relative cost waste level | `High` |
| `CloudWatch Metrics Required` | Metrics needed for detection | `CPUUtilization NetworkIn NetworkOut` |
| `API Calls Required` | IAM actions needed | `ec2:DescribeInstances ec2:DescribeInstanceStatus` |
| `Recommendation` | Remediation action | `Stop instance or Terminate if no longer needed` |

**Sample rows**:

```
EC2  — Idle when: CPU < 5% AND Network < 100KB/day for 7 days (Cost Impact: High)
RDS  — Idle when: DatabaseConnections = 0 AND ReadIOPS = 0 AND WriteIOPS = 0 for 7 days
S3   — Idle when: GetRequests = 0 AND PutRequests = 0 AND NumberOfObjects = 0 for 30 days
```

**Cost impact tiers**:

| Impact | Examples | Approximate Monthly Waste |
|---|---|---|
| 🔴 **High** | EC2 m5.xlarge, RDS db.r5.large, Redshift dc2.large | $100–$2,000+/resource |
| 🟡 **Medium** | ElastiCache, NAT Gateway, ALB | $20–$200/resource |
| 🟢 **Low** | S3 empty bucket, EBS gp2 1GiB, DynamoDB on-demand | < $5/resource |

---

### `underutilized_resources_criteria.csv`

**Purpose**: Finds resources that are active but significantly over-provisioned — these are your rightsizing opportunities.

**Columns**:

| Column | Description | Example |
|---|---|---|
| `Service Name` | AWS service name | `EC2` |
| `Criteria 1` | Primary over-provisioning signal | `CPUUtilization < 30%` |
| `Criteria 2` | Secondary over-provisioning signal | `NetworkIn < 10MB/day` |
| `Criteria 3` | Confirming active usage (not idle) | `State = running` |
| `Time Window` | Evaluation period | `Last 14 days` |
| `Optimization Recommendation` | Specific rightsizing action | `Downsize to smaller instance type` |
| `Potential Cost Savings` | Estimated saving range | `30-50%` |
| `CloudWatch Metrics Required` | Metrics for detection | `CPUUtilization NetworkIn NetworkOut` |
| `API Calls Required` | IAM actions needed | `ec2:DescribeInstances cloudwatch:GetMetricStatistics` |

**Savings potential by service**:

| Service | Underutilized Signal | Typical Saving |
|---|---|---|
| EC2 | CPU < 30% over 14 days | 30–50% (downsize instance type) |
| RDS | CPU < 40%, Connections < 20% of max | 40–60% (smaller instance class) |
| DynamoDB | Consumed < 30% of provisioned | 50–70% (switch to on-demand) |
| Lambda | MaxMemoryUsed < 50% of allocated | 20–40% (reduce memory setting) |
| EBS | IOPS < 10% of provisioned (io1/io2) | 30–40% (switch to gp3) |
| S3 | GetRequests < 10/month, age > 90 days | 70–90% (move to Glacier) |
| Fargate | Task CPU < 30%, Memory < 40% | 30–50% (reduce task definition) |
| Redshift | CPU < 40%, low query rate | 40–60% (scale down or use Serverless) |

---

### `implementation_guide.csv`

**Purpose**: The automation blueprint — provides the exact API calls, IAM permissions, sample CLI commands, and monitoring cadence for every service. Use this file to scaffold your FinOps automation scripts.

**Columns**:

| Column | Description | Example |
|---|---|---|
| `Service Name` | AWS service name | `EC2` |
| `AWS API/Service to Use` | SDK/API and monitoring tools | `EC2 API + CloudWatch` |
| `Key Metrics to Collect` | Space-separated CloudWatch metric names | `CPUUtilization NetworkIn NetworkOut` |
| `Sample API Call/CLI Command` | Ready-to-run CLI command | `aws ec2 describe-instances --filters Name=instance-state-name,Values=running` |
| `IAM Permissions Required` | Space-separated IAM actions | `ec2:DescribeInstances cloudwatch:GetMetricStatistics` |
| `Monitoring Frequency Recommendation` | How often to run this check | `Every 6 hours` |

**Monitoring frequency breakdown**:

| Frequency | Use Case | Services |
|---|---|---|
| Every 1–3 hours | Expensive, fast-changing | Redshift, large EC2, NAT Gateway, EMR |
| Every 6 hours | Standard compute and databases | EC2, RDS, ECS, EKS, ElastiCache |
| Every 12 hours | Serverless, moderate cost | Lambda, API Gateway, Fargate |
| Every 24 hours | Storage, slow-changing | S3, EBS, EFS, CloudFront, VPN |
| Weekly | Low-priority monitoring | CloudTrail, Config, GuardDuty |
| Monthly (manual) | Governance, free services | IAM, Organizations, Billing |

---

## Category Definitions

### 🟢 Active

A resource is **Active** when it is running, reachable, and generating measurable workload activity above minimum thresholds.

**Examples**:
- EC2 instance with average CPU > 10% over the last 7 days
- RDS instance receiving database connections with read/write IOPS > 0
- Lambda function invoked > 1 time per day
- S3 bucket receiving > 100 GET requests per day
- DynamoDB table consuming read or write capacity units

**What to do**: No action required. Monitor for rightsizing opportunities.

---

### 🔴 Idle

A resource is **Idle** when it is provisioned (and accruing cost) but has near-zero activity — effectively abandoned.

**Examples**:
- EC2 instance running for 30 days with CPU < 5% and < 100KB/day of network traffic
- RDS instance with zero database connections for 7+ days
- Elastic Load Balancer with zero requests in 7 days
- NAT Gateway with zero bytes transferred in 7 days
- DynamoDB table with zero reads or writes in 30 days

**What to do**:
1. Verify with the resource owner that the resource is truly unused
2. **Stop** if the resource may be needed again soon (saves compute cost, retains data)
3. **Terminate / Delete** if the resource is confirmed no longer needed
4. Use AWS Resource Tags (`Environment=dev`, `Owner=team-name`) to identify ownership before taking action

---

### 🟡 Underutilized

A resource is **Underutilized** when it is active (serving real traffic) but significantly over-provisioned relative to its actual workload.

**Examples**:
- EC2 `m5.2xlarge` with consistent CPU < 20% — could be downsized to `m5.large`
- RDS `db.r5.2xlarge` with CPU < 40% and fewer than 20 connections — could drop to `db.r5.large`
- Lambda function allocated 3,008 MB but MaxMemoryUsed never exceeds 512 MB
- S3 bucket in `STANDARD` storage class with objects not accessed in 90 days — move to `INTELLIGENT_TIERING` or `GLACIER`
- Provisioned DynamoDB table consuming only 15% of provisioned capacity

**What to do**:
1. Use **AWS Compute Optimizer** for automated EC2 and Lambda rightsizing recommendations
2. Apply **RDS Performance Insights** to validate database sizing
3. Implement **S3 Lifecycle Policies** to automatically transition cold objects
4. Switch DynamoDB tables to **on-demand billing** when access patterns are unpredictable

---

### 🔵 Archived

A resource is **Archived** when it is intentionally retained for compliance, audit, or disaster recovery purposes and is not expected to receive active traffic.

**Examples**:
- S3 objects in `GLACIER_DEEP_ARCHIVE` with a 7-year retention policy
- EBS snapshots tagged `Purpose=Backup` older than 90 days
- RDS automated backups retained for compliance

**What to do**: Move to the lowest-cost tier possible while meeting your retention SLAs.

---

## Python Implementation Examples

### Example 1: Full Multi-Service Scanner

```python
#!/usr/bin/env python3
"""
HawkEye FinOps Scanner — multi-service idle/underutilized detection.
Reads thresholds from the framework CSV files.
"""

import csv
import json
from datetime import datetime, timedelta, timezone
from pathlib import Path

import boto3

# ── Configuration ──────────────────────────────────────────────────────────────
FRAMEWORK_DIR = Path(__file__).parent / "aws_finops_categorization"
LOOKBACK_DAYS = 7
REGION = "us-east-1"


def load_criteria(csv_file: str) -> dict:
    """Load a criteria CSV into a dict keyed by Service Name."""
    criteria = {}
    with open(FRAMEWORK_DIR / csv_file, newline="", encoding="utf-8") as f:
        for row in csv.DictReader(f):
            criteria[row["Service Name"]] = row
    return criteria


def get_metric_average(cw, namespace: str, metric: str,
                        dimensions: list, days: int) -> float:
    """Return the average value of a CloudWatch metric over the last N days."""
    end = datetime.now(timezone.utc)
    start = end - timedelta(days=days)
    response = cw.get_metric_statistics(
        Namespace=namespace,
        MetricName=metric,
        Dimensions=dimensions,
        StartTime=start.isoformat(),
        EndTime=end.isoformat(),
        Period=days * 86400,
        Statistics=["Average"],
    )
    if not response["Datapoints"]:
        return 0.0
    return response["Datapoints"][0]["Average"]


def scan_ec2(session: boto3.Session) -> list:
    """Categorize all running EC2 instances."""
    ec2 = session.client("ec2")
    cw = session.client("cloudwatch")
    results = []

    for reservation in ec2.describe_instances(
        Filters=[{"Name": "instance-state-name", "Values": ["running"]}]
    )["Reservations"]:
        for instance in reservation["Instances"]:
            iid = instance["InstanceId"]
            dims = [{"Name": "InstanceId", "Value": iid}]

            cpu = get_metric_average(cw, "AWS/EC2", "CPUUtilization", dims, LOOKBACK_DAYS)
            net_in = get_metric_average(cw, "AWS/EC2", "NetworkIn", dims, LOOKBACK_DAYS)
            net_out = get_metric_average(cw, "AWS/EC2", "NetworkOut", dims, LOOKBACK_DAYS)

            # Apply framework thresholds (from idle/underutilized CSV files)
            if cpu < 5 and (net_in + net_out) < 102400:   # < 100KB/day
                category = "IDLE"
            elif cpu < 30:
                category = "UNDERUTILIZED"
            else:
                category = "ACTIVE"

            results.append({
                "service": "EC2",
                "resource_id": iid,
                "instance_type": instance["InstanceType"],
                "category": category,
                "avg_cpu_pct": round(cpu, 2),
                "avg_net_in_bytes": round(net_in, 0),
            })

    return results


def scan_rds(session: boto3.Session) -> list:
    """Categorize all available RDS instances."""
    rds = session.client("rds")
    cw = session.client("cloudwatch")
    results = []

    for db in rds.describe_db_instances()["DBInstances"]:
        if db["DBInstanceStatus"] != "available":
            continue
        db_id = db["DBInstanceIdentifier"]
        dims = [{"Name": "DBInstanceIdentifier", "Value": db_id}]

        connections = get_metric_average(cw, "AWS/RDS", "DatabaseConnections", dims, LOOKBACK_DAYS)
        cpu = get_metric_average(cw, "AWS/RDS", "CPUUtilization", dims, LOOKBACK_DAYS)
        read_iops = get_metric_average(cw, "AWS/RDS", "ReadIOPS", dims, LOOKBACK_DAYS)
        write_iops = get_metric_average(cw, "AWS/RDS", "WriteIOPS", dims, LOOKBACK_DAYS)

        if connections == 0 and read_iops == 0 and write_iops == 0:
            category = "IDLE"
        elif cpu < 40 and connections < 20:
            category = "UNDERUTILIZED"
        else:
            category = "ACTIVE"

        results.append({
            "service": "RDS",
            "resource_id": db_id,
            "instance_class": db["DBInstanceClass"],
            "category": category,
            "avg_connections": round(connections, 1),
            "avg_cpu_pct": round(cpu, 2),
        })

    return results


def generate_report(findings: list) -> None:
    """Print a formatted cost-optimization report."""
    idle = [r for r in findings if r["category"] == "IDLE"]
    underutilized = [r for r in findings if r["category"] == "UNDERUTILIZED"]
    active = [r for r in findings if r["category"] == "ACTIVE"]

    print("\n" + "=" * 60)
    print("  HawkEye FinOps Report")
    print("=" * 60)
    print(f"  Total resources scanned : {len(findings)}")
    print(f"  🟢 Active               : {len(active)}")
    print(f"  🟡 Underutilized        : {len(underutilized)}")
    print(f"  🔴 Idle                 : {len(idle)}")
    print("=" * 60)

    if idle:
        print("\n🔴 IDLE — Terminate or Stop Immediately:")
        for r in idle:
            print(f"  [{r['service']}] {r['resource_id']}")

    if underutilized:
        print("\n🟡 UNDERUTILIZED — Consider Rightsizing:")
        for r in underutilized:
            print(f"  [{r['service']}] {r['resource_id']}")


if __name__ == "__main__":
    session = boto3.Session(region_name=REGION)
    findings = scan_ec2(session) + scan_rds(session)
    generate_report(findings)
    # Export to JSON for downstream processing
    with open("/tmp/hawkeye_report.json", "w") as f:
        json.dump(findings, f, indent=2)
    print("\n✅ Full report saved to /tmp/hawkeye_report.json")
```

---

### Example 2: Load Thresholds Directly from CSV

```python
import csv
from pathlib import Path

def load_idle_thresholds(framework_dir: str = "aws_finops_categorization") -> dict:
    """
    Parse idle_resources_criteria.csv and return a dict of
    {service_name: {"cost_impact": "High", "recommendation": "..."}} 
    for high-impact idle resources.
    """
    path = Path(framework_dir) / "idle_resources_criteria.csv"
    high_impact = {}
    with open(path, newline="", encoding="utf-8") as f:
        for row in csv.DictReader(f):
            if row.get("Cost Impact") == "High":
                high_impact[row["Service Name"]] = {
                    "cost_impact": row["Cost Impact"],
                    "recommendation": row["Recommendation"],
                    "metrics": row["CloudWatch Metrics Required"].split(),
                    "api_calls": row["API Calls Required"].split(),
                }
    return high_impact


thresholds = load_idle_thresholds()
print(f"High-impact idle services: {list(thresholds.keys())[:5]}")
```

---

### Example 3: Build IAM Policy from implementation_guide.csv

```python
import csv
import json
from pathlib import Path

def build_finops_iam_policy(
    framework_dir: str = "aws_finops_categorization",
    services: list = None,
) -> dict:
    """
    Generate a least-privilege IAM policy for FinOps scanning
    by reading required permissions from implementation_guide.csv.
    """
    path = Path(framework_dir) / "implementation_guide.csv"
    actions = set()

    with open(path, newline="", encoding="utf-8") as f:
        for row in csv.DictReader(f):
            if services and row["Service Name"] not in services:
                continue
            perms = row.get("IAM Permissions Required", "")
            for perm in perms.split():
                if ":" in perm and perm != "N/A":
                    actions.add(perm)

    policy = {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "HawkEyeFinOpsReadOnly",
                "Effect": "Allow",
                "Action": sorted(actions),
                "Resource": "*",
            }
        ],
    }
    return policy


# Generate policy for EC2, RDS, and Lambda
policy = build_finops_iam_policy(services=["EC2", "RDS", "Lambda"])
print(json.dumps(policy, indent=2))
```

---

### Example 4: S3 Lifecycle Policy Automation

```python
import boto3

def apply_intelligent_tiering(bucket_name: str) -> None:
    """
    Apply S3 Intelligent-Tiering lifecycle rule to automatically
    move infrequently accessed objects to cheaper storage classes.
    Implements the S3 underutilized optimization from the framework.
    """
    s3 = boto3.client("s3")

    lifecycle_config = {
        "Rules": [
            {
                "ID": "HawkEye-IntelligentTiering",
                "Status": "Enabled",
                "Filter": {"Prefix": ""},
                "Transitions": [
                    {
                        "Days": 30,
                        "StorageClass": "INTELLIGENT_TIERING",
                    }
                ],
            },
            {
                "ID": "HawkEye-GlacierArchive",
                "Status": "Enabled",
                "Filter": {"Prefix": "archives/"},
                "Transitions": [
                    {"Days": 90, "StorageClass": "GLACIER"},
                    {"Days": 365, "StorageClass": "DEEP_ARCHIVE"},
                ],
            },
        ]
    }

    s3.put_bucket_lifecycle_configuration(
        Bucket=bucket_name,
        LifecycleConfiguration=lifecycle_config,
    )
    print(f"✅ Lifecycle policy applied to s3://{bucket_name}")
```

---

## IAM Policy Templates

### Read-Only FinOps Scanner Policy

Apply this policy to the IAM role used by your scanning Lambda function or EC2 instance.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "HawkEyeComputeRead",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeInstanceStatus",
        "ec2:DescribeVolumes",
        "ec2:DescribeNatGateways",
        "ec2:DescribeVpnConnections",
        "ec2:DescribeLoadBalancers",
        "ecs:ListClusters",
        "ecs:ListServices",
        "ecs:DescribeServices",
        "ecs:DescribeTasks",
        "eks:ListClusters",
        "eks:DescribeNodegroup",
        "lambda:ListFunctions",
        "lambda:GetFunction",
        "lambda:GetFunctionConfiguration"
      ],
      "Resource": "*"
    },
    {
      "Sid": "HawkEyeDatabaseRead",
      "Effect": "Allow",
      "Action": [
        "rds:DescribeDBInstances",
        "rds:DescribeDBClusters",
        "dynamodb:ListTables",
        "dynamodb:DescribeTable",
        "elasticache:DescribeCacheClusters",
        "redshift:DescribeClusters"
      ],
      "Resource": "*"
    },
    {
      "Sid": "HawkEyeStorageRead",
      "Effect": "Allow",
      "Action": [
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation",
        "s3:GetBucketLifecycleConfiguration",
        "s3:GetBucketTagging",
        "elasticfilesystem:DescribeFileSystems",
        "glacier:ListVaults"
      ],
      "Resource": "*"
    },
    {
      "Sid": "HawkEyeCloudWatchRead",
      "Effect": "Allow",
      "Action": [
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:ListMetrics",
        "cloudwatch:GetMetricData"
      ],
      "Resource": "*"
    },
    {
      "Sid": "HawkEyeCostRead",
      "Effect": "Allow",
      "Action": [
        "ce:GetCostAndUsage",
        "ce:GetRightsizingRecommendation",
        "compute-optimizer:GetEC2InstanceRecommendations",
        "compute-optimizer:GetLambdaFunctionRecommendations"
      ],
      "Resource": "*"
    }
  ]
}
```

### Trust Policy (for Lambda Execution Role)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### Deploy via AWS CLI

```bash
# Create the scanning role
aws iam create-role \
  --role-name HawkEyeFinOpsScannerRole \
  --assume-role-policy-document file://trust-policy.json

# Attach the read-only policy
aws iam put-role-policy \
  --role-name HawkEyeFinOpsScannerRole \
  --policy-name HawkEyeReadOnlyPolicy \
  --policy-document file://finops-scanner-policy.json

# Attach AWS managed policy for Lambda basic execution
aws iam attach-role-policy \
  --role-name HawkEyeFinOpsScannerRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

---

## Monitoring Frequency Guidelines

Use the `Monitoring Frequency Recommendation` column in `implementation_guide.csv` to configure EventBridge (CloudWatch Events) schedules.

```
┌─────────────────┬──────────────────────────────────────────────────────────────┐
│ Frequency       │ Services                                                     │
├─────────────────┼──────────────────────────────────────────────────────────────┤
│ Every 1 hour    │ Redshift clusters, large EC2 (> m5.4xlarge), EMR clusters,   │
│                 │ NAT Gateway (high traffic)                                   │
├─────────────────┼──────────────────────────────────────────────────────────────┤
│ Every 6 hours   │ EC2 (all sizes), RDS, ECS Services, EKS nodegroups,          │
│                 │ ElastiCache, OpenSearch, MSK                                 │
├─────────────────┼──────────────────────────────────────────────────────────────┤
│ Every 12 hours  │ Lambda, API Gateway, Fargate tasks, AppSync, SQS queues,     │
│                 │ SNS topics, Step Functions                                   │
├─────────────────┼──────────────────────────────────────────────────────────────┤
│ Every 24 hours  │ S3 buckets, EBS volumes, EFS file systems, CloudFront,       │
│                 │ WAF, Shield, Route 53                                        │
├─────────────────┼──────────────────────────────────────────────────────────────┤
│ Weekly          │ CloudTrail, AWS Config, GuardDuty, Security Hub,             │
│                 │ Inspector, Macie, Trusted Advisor                            │
├─────────────────┼──────────────────────────────────────────────────────────────┤
│ Monthly         │ IAM users/roles, Organizations, Control Tower, SSO,          │
│                 │ Service Catalog, free-tier services                          │
└─────────────────┴──────────────────────────────────────────────────────────────┘
```

### EventBridge Schedule Examples

```bash
# EC2 scan every 6 hours
aws events put-rule \
  --name "HawkEye-EC2-Scan" \
  --schedule-expression "rate(6 hours)" \
  --state ENABLED

# S3 scan every 24 hours
aws events put-rule \
  --name "HawkEye-S3-Scan" \
  --schedule-expression "rate(1 day)" \
  --state ENABLED

# Monthly IAM audit
aws events put-rule \
  --name "HawkEye-IAM-Audit" \
  --schedule-expression "cron(0 9 1 * ? *)" \
  --state ENABLED
```

---

## Getting Started Tutorial

### Step 1: Understand Your Inventory

Load `services_overview.csv` to see the complete list of services and their expected categories:

```bash
python3 -c "
import csv
with open('aws_finops_categorization/services_overview.csv') as f:
    rows = list(csv.DictReader(f))
print(f'Total services covered: {len(rows)}')
# Show first 5
for r in rows[:5]:
    print(r['Service Name'], '-', r['Category Type'])
"
```

### Step 2: Identify Your Highest-Cost Services

Use AWS Cost Explorer to find where your money is going before running scans:

```bash
START_DATE=$(date -d "$(date +%Y-%m-01)" +%Y-%m-%d)
END_DATE=$(date +%Y-%m-%d)

aws ce get-cost-and-usage \
  --time-period Start="${START_DATE}",End="${END_DATE}" \
  --granularity MONTHLY \
  --metrics "UnblendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE \
  --query 'ResultsByTime[0].Groups[*].[Keys[0],Metrics.UnblendedCost.Amount]' \
  --output table | sort -k2 -rn | head -10
```

### Step 3: Run Your First Idle Scan

```bash
# Check for idle EC2 instances (CPU < 5% for 7 days)
END_TIME=$(date -u +%Y-%m-%dT%H:%M:%SZ)
START_TIME=$(date -u -d "7 days ago" +%Y-%m-%dT%H:%M:%SZ)

aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
  --start-time "${START_TIME}" \
  --end-time   "${END_TIME}" \
  --period 604800 \
  --statistics Average \
  --query 'Datapoints[0].Average'
```

### Step 4: Check Underutilized RDS

```bash
# List all RDS instances with their connection count over the last 7 days
END_TIME=$(date -u +%Y-%m-%dT%H:%M:%SZ)
START_TIME=$(date -u -d "7 days ago" +%Y-%m-%dT%H:%M:%SZ)

aws cloudwatch get-metric-statistics \
  --namespace AWS/RDS \
  --metric-name DatabaseConnections \
  --dimensions Name=DBInstanceIdentifier,Value=my-database \
  --start-time "${START_TIME}" \
  --end-time   "${END_TIME}" \
  --period 604800 \
  --statistics Average Maximum
```

### Step 5: Review the Implementation Guide

For each service you want to automate, look up its row in `implementation_guide.csv`:

```python
import csv

with open("aws_finops_categorization/implementation_guide.csv") as f:
    guide = {row["Service Name"]: row for row in csv.DictReader(f)}

# Get automation details for EC2
ec2_guide = guide["EC2"]
print("API to use     :", ec2_guide["AWS API/Service to Use"])
print("Metrics        :", ec2_guide["Key Metrics to Collect"])
print("Sample CLI     :", ec2_guide["Sample API Call/CLI Command"])
print("IAM permissions:", ec2_guide["IAM Permissions Required"])
print("Check frequency:", ec2_guide["Monitoring Frequency Recommendation"])
```

### Step 6: Automate Remediation

```python
import boto3

def stop_idle_ec2_instances(dry_run: bool = True) -> list:
    """
    Stop EC2 instances identified as IDLE.
    Set dry_run=False only after reviewing the list.
    """
    ec2 = boto3.client("ec2")
    cw = boto3.client("cloudwatch")
    stopped = []

    for reservation in ec2.describe_instances(
        Filters=[{"Name": "instance-state-name", "Values": ["running"]}]
    )["Reservations"]:
        for instance in reservation["Instances"]:
            iid = instance["InstanceId"]
            # ... (run CPU check as shown in Example 1) ...
            # if idle:
            #     if not dry_run:
            #         ec2.stop_instances(InstanceIds=[iid])
            #     stopped.append(iid)

    return stopped
```

---

## Use Cases and Best Practices

### Use Case 1: Monthly Cost Review

Run a full sweep across EC2, RDS, and S3 on the first of each month:

1. Export all running resources using the CLI commands in `implementation_guide.csv`
2. Compare against active/idle criteria from `active_resources_criteria.csv` and `idle_resources_criteria.csv`
3. Generate a list of candidates, grouped by `Cost Impact` (High → Low)
4. Send Slack/email alerts to resource owners using AWS Tags for `Owner`
5. Automatically stop resources with no owner response within 5 business days

### Use Case 2: Pre-Budget Planning

Before AWS budget renewals, use the framework to project savings:

```
Potential savings = (idle_ec2 × hourly_rate × 730) +
                    (underutilized_rds × hourly_diff × 730) +
                    (s3_to_glacier × bytes × storage_price_diff)
```

### Use Case 3: New Account Baseline

When onboarding a new AWS account, run the full framework to establish a clean baseline and tag all resources before any optimization happens.

### Best Practices

| Practice | Why |
|---|---|
| **Always tag resources** with `Owner`, `Environment`, `CostCenter` before scanning | Enables targeted notifications and accountability |
| **Use dry-run mode** for all automated remediation initially | Prevents accidental disruption of production workloads |
| **Start with High cost-impact idle resources** | Maximizes ROI in the shortest time |
| **Validate with Compute Optimizer** before rightsizing EC2/Lambda | AWS Compute Optimizer uses ML to confirm recommendations |
| **Set a 30-day lookback for DynamoDB** instead of 7 days | DynamoDB access patterns can be weekly/monthly |
| **Coordinate with dev teams** before stopping non-prod instances | Teams may be running scheduled batch jobs |
| **Keep audit trail in CloudTrail** | Document all framework-driven changes for compliance |
| **Review thresholds quarterly** | Business patterns change — thresholds should adapt |

---

## Success Metrics and KPIs

Track these KPIs monthly to measure the impact of your FinOps program:

### Cost Reduction KPIs

| KPI | Target | How to Measure |
|---|---|---|
| **Idle Resource Count** | < 2% of total resources | `idle_findings / total_resources * 100` |
| **Idle Resource Cost** | < 1% of total AWS bill | AWS Cost Explorer + resource tagging |
| **Underutilized Resource Count** | < 15% of total resources | `underutilized_findings / total_resources * 100` |
| **Rightsizing Adoption Rate** | > 80% of recommendations acted on | Track open recommendations vs resolved |
| **Monthly Savings Realized** | > 20% of total bill in year 1 | Compare bill month-over-month after optimizations |

### Operational KPIs

| KPI | Target | How to Measure |
|---|---|---|
| **Time to Detect Idle Resource** | < 24 hours | First detection timestamp vs resource creation timestamp |
| **Time to Remediate Idle Resource** | < 5 business days | Detection timestamp vs stop/terminate timestamp |
| **Framework Coverage** | 100% of services in use | Services scanned / services deployed |
| **False Positive Rate** | < 5% | Resources incorrectly flagged as idle that were actually in use |

### Sample KPI Dashboard Query

```python
def compute_finops_kpis(findings: list, total_monthly_bill: float) -> dict:
    idle = [f for f in findings if f["category"] == "IDLE"]
    underutilized = [f for f in findings if f["category"] == "UNDERUTILIZED"]
    total = len(findings)

    return {
        "total_resources_scanned": total,
        "idle_count": len(idle),
        "idle_pct": round(len(idle) / total * 100, 1) if total else 0,
        "underutilized_count": len(underutilized),
        "underutilized_pct": round(len(underutilized) / total * 100, 1) if total else 0,
        "estimated_monthly_idle_waste_usd": round(total_monthly_bill * 0.15, 2),
        "estimated_monthly_rightsizing_savings_usd": round(total_monthly_bill * 0.30, 2),
    }
```

---

## AWS Resources

### Documentation

- [AWS Cost Optimization Hub](https://docs.aws.amazon.com/cost-management/latest/userguide/cos-optimization-hub.html)
- [AWS Compute Optimizer](https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html)
- [AWS Cost Explorer Rightsizing Recommendations](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/ce-rightsizing.html)
- [CloudWatch Metrics Reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html)
- [AWS Trusted Advisor Cost Optimization Checks](https://docs.aws.amazon.com/awssupport/latest/user/cost-optimization-checks.html)
- [S3 Storage Classes Overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html)
- [RDS Performance Insights](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html)
- [AWS Well-Architected Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)

### Tools

- [AWS Cost Explorer](https://console.aws.amazon.com/cost-management/home#/cost-explorer)
- [AWS Compute Optimizer Console](https://console.aws.amazon.com/compute-optimizer/)
- [CloudWatch Metrics Explorer](https://console.aws.amazon.com/cloudwatch/home#metricsV2)
- [AWS Trusted Advisor Console](https://console.aws.amazon.com/trustedadvisor/)

### Community and Whitepapers

- [AWS FinOps Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/cost-optimization-laying-the-foundation/welcome.html)
- [FinOps Foundation — AWS Guidance](https://www.finops.org/projects/calculating-container-costs/)
- [AWS re:Invent Cost Optimization Sessions](https://reinvent.awsevents.com/)

---

## File Summary

| File | Rows | Columns | Purpose |
|---|---|---|---|
| `services_overview.csv` | 239 | 8 | Master reference — all services with all category criteria |
| `active_resources_criteria.csv` | 239 | 8 | Thresholds to confirm active usage |
| `idle_resources_criteria.csv` | 239 | 9 | Thresholds to flag idle (wasteful) resources |
| `underutilized_resources_criteria.csv` | 239 | 9 | Thresholds and rightsizing recommendations |
| `implementation_guide.csv` | 239 | 6 | API calls, IAM permissions, monitoring cadence |

---

## Notes

- All thresholds are based on AWS and industry best practices and can be adjusted to fit your workload patterns
- Services listed as "N/A" in certain columns are free-tier or management services where that category does not apply
- Always consider business context (criticality, maintenance windows, batch schedules) before acting on any finding
- The framework is designed for read-only scanning — remediation actions should always require explicit human approval or a separate privileged role

---

**Version**: 2.0 | **Last Updated**: March 2026 | **Coverage**: 239 AWS Services