# AWS FinOps Resource Categorization Framework

## Overview
This framework provides comprehensive categorization criteria for AWS resources across 250+ services into four categories: **Active**, **Idle**, **Underutilized**, and **Archived**.

## Category Definitions

### 1. ACTIVE
Resources that are currently running/enabled AND being actively used/consumed.
- Operational and serving their intended purpose
- Generating meaningful activity (traffic, compute, I/O operations)
- Meeting or exceeding usage thresholds
- Providing value to the business

### 2. IDLE
Resources that are provisioned/running but NOT being used or generating minimal to no activity.
- Still consuming costs (running/provisioned)
- Near-zero utilization or activity
- Not deleted but dormant
- Potential waste candidates

### 3. UNDERUTILIZED
Resources that are active but operating significantly below their provisioned capacity.
- Being used, but inefficiently
- Oversized for actual workload
- Opportunity for rightsizing or cost optimization
- Between "idle" and "active" - there's some usage but wasteful

### 4. ARCHIVED
Resources explicitly designated for long-term retention, compliance, or backup - not meant for active operational use.
- Intentionally stored for historical/regulatory purposes
- Typically in lower-cost storage tiers
- Rarely accessed (if at all)
- Necessary for compliance/disaster recovery but not day-to-day operations

## Files Included

1. **services_overview.csv** - Master overview with categorization criteria for all AWS services
2. **active_resources_criteria.csv** - Detailed criteria for identifying active resources
3. **idle_resources_criteria.csv** - Detailed criteria for identifying idle resources
4. **underutilized_resources_criteria.csv** - Detailed criteria for identifying underutilized resources
5. **archived_resources_criteria.csv** - Detailed criteria for identifying archived resources
6. **implementation_guide.csv** - APIs, metrics, and implementation details
7. **metric_thresholds_reference.csv** - Threshold values and evaluation periods

## How to Use

### Import to Excel
1. Download all CSV files
2. Open Excel and create a new workbook
3. For each CSV file:
   - Go to Data → Get Data → From File → From Text/CSV
   - Select the CSV file
   - Click "Load" to import into a new sheet
4. Rename sheets according to file names

### Implementation Steps
1. Review the `services_overview.csv` to understand criteria for each service
2. Use `implementation_guide.csv` to identify required AWS APIs and permissions
3. Refer to `metric_thresholds_reference.csv` for specific threshold values
4. Implement data collection using CloudWatch APIs and service-specific APIs
5. Apply categorization logic based on the criteria files

## Key Metrics Sources

- **CloudWatch Metrics** - CPUUtilization, NetworkIn/Out, IOPS, etc.
- **AWS APIs** - Service-specific APIs (EC2, RDS, S3, etc.)
- **AWS Config** - Resource configuration and compliance
- **AWS Cost Explorer** - Cost and usage data
- **CloudTrail** - API activity and access patterns

## Recommended Evaluation Periods

- **Active/Idle Detection**: 7-14 days
- **Underutilization Analysis**: 14-30 days
- **Archive Candidates**: 90+ days

## Notes

- Thresholds are based on industry best practices and can be adjusted based on your requirements
- Some services may have "N/A" for certain categories where categorization doesn't apply
- Services marked as management/governance services may require different evaluation criteria
- Always consider business context and criticality when categorizing resources

## Support

For questions or issues with this framework, refer to AWS documentation for specific service metrics and APIs.

---

**Version**: 1.0
**Last Updated**: March 2026
**Coverage**: 250+ AWS Services