---

# 2. BACKUP-AND-DR.md

markdown
# Backup and Disaster Recovery (DR)

This document focuses on how **AWS Backup** is configured to protect your resources, enforce retention policies, and replicate data to a secondary region for disaster recovery (DR).

## Key Features

1. **Automated Backups**  
   - Backups of your RDS instance (and potentially other services) run daily at 03:00 UTC.
   - Uses a `BackupPlan` with a `cron(0 3 * * ? *)` schedule.

2. **Retention Policies**  
   - Backups are kept in the primary region for 30 days.
   - After 30 days, they are deleted from the primary vault (unless you change the default).

3. **Cross-Region Copy**  
   - A copy of each backup is sent to a DR vault in `DRRegion` (default `us-west-2`).
   - DR backups remain for 60 days, providing an extended window for recovery.

4. **Real-Time Alerts**  
   - **SNS** topic + **EventBridge** rule triggers email notifications for:
     - **Backup Job Started**  
     - **Backup Job Completed**  
     - **Copy Job Started**  
     - **Copy Job Successful**  
     - **Copy Job Failed**  

## Vaults and Plans

- **Primary Vault** (`PrimaryBackupVault`):  
  Stores the backups in the same region as your main stack.
- **DR Vault**:  
  You must have a vault named `AwesomeEnvDRVault` in the DR region. The `BackupPlan` references that vault’s ARN to copy backups across regions.

## Recovery Steps

1. **In the Primary Region:**  
   - If a failure occurs, restore from the most recent backup in `PrimaryBackupVault`.
2. **In the DR Region:**  
   - If the primary region is unavailable, restore from the backups in `AwesomeEnvDRVault` (DR region).  
   - Spin up your resources in the DR region, attach them to the restored database, and resume operations.

## Diagram

mermaid
flowchart LR
    A[AWS Backup Plan] -- Schedules & Rules --> B[Backup Job]
    B -->|Backs up daily| RDS((RDS Instance))
    B -->|Creates snapshot| VaultPrimary[Primary Vault]
    B -->|Copies snapshot| VaultDR[DR Vault in Another Region]

    subgraph Notifications
        E((EventBridge)) --> SNS((SNS Topic))
        B -->|Job Started/Completed| E
    end