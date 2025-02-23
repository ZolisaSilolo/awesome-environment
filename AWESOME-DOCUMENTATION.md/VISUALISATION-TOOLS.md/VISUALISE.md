---

# 3. VISUALIZATION.md

markdown
# Visualizing Your Infrastructure as Code (IaC)

Below are some recommended tools and methods to visualize your AWS CloudFormation–based architecture.

## 1. Mermaid Diagrams

You’ve seen Mermaid examples in our docs. Mermaid is a powerful, text-based way to create diagrams. You can view here:
 
---

## 🖼 Diagram – A Picture Worth a Thousand Boerewors

'''mermaid diagram
flowchart LR
    subgraph VPC[Digital Township]
    direction TB
        IGW(Internet Gateway) --> PublicSubnet1{{🏢 Public Subnet 1}}
        IGW --> PublicSubnet2{{🏢 Public Subnet 2}}
        NAT(NAT Gateway) --> PrivateSubnet1{{🔒 Private Subnet 1}}
        NAT --> PrivateSubnet2{{🔒 Private Subnet 2}}

        ASG[🦾 Auto Scaling EC2] --> ELB[🎯 Elastic Load Balancer]
        ELB --> RDS[(🏦 RDS MySQL)]
        ELB --> DynamoDB[(🛍 DynamoDB)]
        ELB --> S3[(🪣 S3 Bucket)]
    end

    CloudFront((🌍 CloudFront)) --> S3
    Route53((📍 Route 53)) --> ELB
    WAF((🛡 WAF)) --> ELB

    subgraph DR[Disaster Recovery Kraal]
        BackupVault[📀 Primary Backups] --> DRVault[📀 DR in Cape Town]
        SNS📱 --> |"🚨 Backup failed, china!"| DevOpsTeam
    end

This mean you can:

- Embed Mermaid code in your `.md` files (as we did).
- Use the [Mermaid Live Editor](https://mermaid.live) to prototype diagrams quickly.
- Integrate Mermaid in many static site generators (e.g., MkDocs, Docusaurus) for automated diagram rendering.

## 2. cfn-diagram

[cfn-diagram](https://github.com/awslabs/cfn-diagram) is an **open‐source** tool from AWS Labs. It:
- Parses your CloudFormation templates.
- Generates architecture diagrams automatically.
- Can output to formats like `png`, `svg`, or `draw.io` diagrams.

### Usage Example

1. Install:
   bash
   pip install cfn-diagram