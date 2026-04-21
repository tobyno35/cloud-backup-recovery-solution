# Cloud Backup & Recovery Solution

An automated AWS backup and disaster recovery pipeline with defined RTO/RPO targets, scheduled snapshot management, and a documented recovery runbook. Built to simulate enterprise-grade data protection for cloud workloads.

---

## Overview

Data loss and prolonged downtime are business-ending events. This project implements a layered backup strategy using native AWS services — automated snapshots, cross-region replication, and Lambda-triggered recovery workflows — with clear recovery time and recovery point objectives defined upfront.

---

## Recovery Objectives

| Target | Value | Description |
|---|---|---|
| **RTO** (Recovery Time Objective) | < 1 hour | Maximum acceptable downtime before systems restored |
| **RPO** (Recovery Point Objective) | < 4 hours | Maximum acceptable data loss window |

---

## Architecture

```
Production Environment
        │
        ├──▶ EBS Snapshots (automated, daily)
        │         └──▶ Cross-Region Copy (us-west-2 DR region)
        │
        ├──▶ S3 Versioning + Lifecycle Rules
        │         └──▶ S3 Cross-Region Replication
        │
        ├──▶ RDS Automated Backups (7-day retention)
        │
        └──▶ CloudWatch Events → Lambda (backup orchestration)
                    │
                    └──▶ SNS Notification (backup success/failure alerts)
```

---

## Features

- **Automated EBS Snapshots** — daily snapshots via CloudWatch Events + Lambda, 7-day retention with auto-cleanup
- **Cross-Region Replication** — S3 bucket replication to secondary region for geographic redundancy
- **S3 Versioning** — full object version history with lifecycle policies moving old versions to Glacier
- **RDS Backup Integration** — documents RDS automated backup configuration and point-in-time recovery procedure
- **Lambda Orchestration** — serverless backup trigger and cleanup functions
- **SNS Alerting** — email/SMS notifications on backup completion and failures
- **Recovery Runbook** — step-by-step documented restore procedures for each component

---

## Tech Stack

| Component | Service |
|---|---|
| Snapshot Management | AWS Lambda + CloudWatch Events |
| Object Storage | AWS S3 (versioning + replication) |
| Block Storage | AWS EBS Snapshots |
| Database | AWS RDS (automated backups) |
| Alerting | AWS SNS |
| Archival | AWS S3 Glacier |
| Monitoring | AWS CloudWatch |

---

## Project Structure

```
cloud-backup-recovery-solution/
├── lambda/
│   ├── create_snapshots.py          # EBS snapshot creation and tagging
│   ├── cleanup_old_snapshots.py     # Retention policy enforcement
│   └── notify_backup_status.py     # SNS notification handler
├── scripts/
│   ├── enable_s3_versioning.sh      # S3 versioning and replication setup
│   ├── configure_rds_backup.sh      # RDS backup window configuration
│   └── setup_cloudwatch_events.sh   # Scheduled trigger configuration
├── runbooks/
│   ├── ebs_restore_procedure.md     # Step-by-step EBS volume restore
│   ├── s3_object_recovery.md        # S3 versioned object restore procedure
│   └── rds_point_in_time_restore.md # RDS PITR walkthrough
├── config/
│   └── backup_policy.yaml           # Retention rules and schedule config
└── README.md
```

---

## Deployment

### Step 1 — Configure S3 Backup Bucket
```bash
bash scripts/enable_s3_versioning.sh --bucket <your-backup-bucket> --region us-east-1
```

### Step 2 — Deploy Lambda Snapshot Functions
```bash
# Package and deploy via AWS CLI
zip create_snapshots.zip lambda/create_snapshots.py
aws lambda create-function --function-name CreateEBSSnapshots \
  --runtime python3.11 --handler create_snapshots.lambda_handler \
  --zip-file fileb://create_snapshots.zip --role <lambda-execution-role-arn>
```

### Step 3 — Schedule via CloudWatch Events
```bash
bash scripts/setup_cloudwatch_events.sh --schedule "cron(0 2 * * ? *)"
```

### Step 4 — Configure Alerts
```bash
aws sns create-topic --name backup-alerts
aws sns subscribe --topic-arn <topic-arn> --protocol email --notification-endpoint your@email.com
```

---

## Recovery Procedure (Summary)

Full runbooks are in `/runbooks/`. High-level steps:

**EBS Volume Recovery:**
1. Identify most recent snapshot in target region
2. Create new volume from snapshot
3. Attach to replacement EC2 instance
4. Verify data integrity and mount

**S3 Object Recovery:**
1. List object versions via AWS CLI or console
2. Restore specific version or delete marker
3. Verify checksum against known good state

---

## Author

**Toby Orisadare** — AWS Cloud Practitioner | [LinkedIn](https://linkedin.com/in/toby-orisadare) | [GitHub](https://github.com/tobyno35)
