# Awesome Environment – Detailed Architecture

This document provides a deep dive into the AWS architecture defined in main-template.yaml. It explains how each component interacts and outlines the rationale behind the design.

## Table of Contents

1. [High-Level Overview](#high-level-overview)
2. [Resource Breakdown](#resource-breakdown)
3. [Networking](#networking)
4. [Compute & Load Balancing](#compute--load-balancing)
5. [Storage & Databases](#storage--databases)
6. [Security Layers](#security-layers)
7. [Backup & Disaster Recovery (DR)](#backup--disaster-recovery-dr)
8. [Diagram](#diagram)

---

## High-Level Overview

- *VPC & Subnets:* A custom Virtual Private Cloud with two public subnets (for internet‐facing resources) and two private subnets (for internal services).
- *Internet & NAT Gateways:* Provides outbound internet access from private subnets while keeping them otherwise isolated.
- *EC2 Auto Scaling Group & ELB:* Ensures highly available and scalable compute instances, fronted by a load balancer.
- *Databases:* Includes both a *MySQL RDS* instance (relational) and *DynamoDB* (NoSQL).
- *S3 Bucket & CloudFront:* For object storage and global content delivery.
- *Route 53:* For DNS management of a custom domain (optional).
- *WAF & NACLs:* Adds security layers at the application and network levels.
- *VPC Peering:* Connects to a peer VPC in another region for cross-region traffic.
- *AWS Backup:* Automates backups, retention policies, cross‐region replication, and real‐time alerts for critical data.

## Resource Breakdown

Below is a quick summary of each major resource:

- **VPC (AwesomeEnvironmentVPC):** The logical boundary for all AWS resources in this stack.  
- *Subnets (Public & Private):* Four subnets, two public (with public IP auto‐assignment) and two private (internal).  
- *Internet Gateway & NAT Gateway:* Manages incoming/outgoing internet connectivity.  
- *EC2 + Auto Scaling Group:* Launches and scales application servers across public subnets.  
- *Elastic Load Balancer:* Distributes incoming HTTP requests across the EC2 instances.  
- *S3 Bucket:* Stores files, logs, or static website content.  
- *RDS (MySQL):* A managed relational database.  
- *DynamoDB Table:* A NoSQL table for fast key‐value lookups.  
- *Route 53 Hosted Zone & DNS Record:* Optionally manages DNS for your domain.  
- *CloudFront Distribution:* Speeds up content delivery from the S3 origin to global users.  
- *WAF (Web ACL):* Protects against common web exploits.  
- *Network ACL:* Additional traffic filtering at the subnet level.  
- *VPC Peering Connection:* Connects to another VPC in a different region.  
- *AWS Backup:* Automates backups for RDS (and other resources), enforces retention, copies snapshots to another region, and triggers alerts.

## Networking

- *CIDR Blocks:*  
  - VPC: 10.0.0.0/16 (default, can be customized)  
  - Public Subnets: 10.0.1.0/24, 10.0.2.0/24  
  - Private Subnets: 10.0.101.0/24, 10.0.102.0/24
- *Route Tables:*  
  - Public Route Table (default route to Internet Gateway)  
  - Private Route Table (default route to NAT Gateway)

## Compute & Load Balancing

- *Launch Configuration:* Defines the AMI and instance type for your EC2.  
- *Auto Scaling Group:* Spans the two public subnets for redundancy. Scales the number of EC2 instances based on demand or health checks.  
- *ELB:* Health checks the instances and routes HTTP traffic to healthy ones.

## Storage & Databases

- *S3 Bucket:* Must be *globally unique*. Can store static assets, logs, or be used as a website origin for CloudFront.  
- *RDS (MySQL):* Deployed in private subnets (non‐publicly accessible). Has a default 7‐day backup retention (in addition to AWS Backup).  
- *DynamoDB:* A fully managed NoSQL database for use cases requiring low‐latency key‐value lookups.

## Security Layers

1. *Security Groups*  
   - EC2 Security Group allows inbound SSH (22) and HTTP (80) from anywhere, plus outbound traffic as needed.  
   - RDS Security Group allows MySQL traffic only from within the VPC CIDR.

2. *Network ACL*  
   - An additional layer that allows inbound/outbound HTTP on port 80.  
   - You can expand these rules to meet your security posture.

3. *WAF*  
   - Currently created with an empty Rules array. You can add rules or managed rule groups (e.g., AWS Managed Rules for Common Vulnerabilities).

## Backup & Disaster Recovery (DR)

- *AWS Backup:* Creates daily snapshots of the RDS instance (and can be extended to EBS volumes or DynamoDB).  
- *Retention Policies:* By default, retains backups for 30 days in the primary region.  
- *Cross‐Region Copy:* Copies backups to another region’s backup vault (where they remain for 60 days).  
- *SNS & EventBridge:* Sends notifications when backup jobs start or complete, so you get real‐time alerts.

## Diagram

Below is a *Mermaid* diagram that shows a high‐level flow of how the infrastructure is laid out:

```mermaid
flowchart LR
    subgraph VPC
    direction TB
        IGW(Internet Gateway)
        NAT(NAT Gateway)
        PublicSubnet1((Public Subnet 1)) --> NAT
        PublicSubnet1 --> IGW
        PublicSubnet2((Public Subnet 2)) --> IGW

        PrivateSubnet1((Private Subnet 1))
        PrivateSubnet2((Private Subnet 2))

        ASG(Auto Scaling Group:EC2) 
        ELB(Elastic Load Balancer)

        PublicSubnet1 --> ASG
        PublicSubnet2 --> ASG
        ASG --> ELB

        RDS((RDS MySQL)) 
        PrivateSubnet1 --> RDS
        PrivateSubnet2 --> RDS

        DynamoDB[(DynamoDB)]
        S3((S3 Bucket))
    end

    CloudFront((CloudFront)) --> S3
    Route53((Route 53)) --> ELB
    WAF((WAF ACL)) --> ELB

    subgraph AWS Backup & DR
    direction TB
        BackupVault[Primary Backup Vault]
        DRVault[DR Vault in Another Region]
        BackupPlan[Backup Plan]
        BackupPlan --> RDS
        BackupPlan --> BackupVault
        BackupPlan --> DRVault
        SNS[(SNS Notifications)]
        EventBridge((EventBridge))
        BackupPlan --> EventBridge --> SNS
    end

    VPC --> BackupPlan