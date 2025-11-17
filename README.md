# Terraform Infrastructure Repository

## 📋 Overview

This repository manages AWS infrastructure for the medical application using Terraform. It follows a layered architecture pattern with separate environments and infrastructure layers.

## 🏗️ Architecture

This repository uses a **layered Terraform setup** with:

- **Separate environments**: dev, stage, prod (different AWS accounts)
- **Infrastructure layers**: platform, data, app
- **Isolated state management** per environment and layer (separate S3 backends)
- **OIDC authentication** for GitHub Actions (no long-lived credentials)
- **Reusable modules** for consistent configuration

## 📁 Repository Structure

```
terraform-infra/
├── modules/          # Reusable platform-owned modules
│   ├── vpc/         # VPC with subnets, NAT, IGW
│   ├── eks/         # EKS cluster with node groups
│   ├── rds/         # RDS databases
│   ├── redis/       # ElastiCache Redis
│   ├── msx-kafka/   # MSK (Managed Streaming for Kafka)
│   ├── alb/         # Application Load Balancer
│   ├── iam/         # IAM roles and policies
│   └── ecr/         # ECR repositories
│
├── global/          # Org-wide infra (deployed ONCE)
│   ├── logging/     # CloudTrail, Config, S3 logging
│   ├── sso/         # AWS SSO configuration
│   ├── security-baseline/  # GuardDuty, Inspector, Macie
│   └── dns/         # Route53 root and delegated zones
│
├── live/            # REAL environments (Terraform ROOTS)
│   ├── dev/
│   │   ├── platform/  # VPC, EKS, networking
│   │   ├── data/      # RDS, Redis, MSK, S3, ECR
│   │   └── app/       # ArgoCD, Ingress, CertManager
│   ├── stage/
│   │   ├── platform/
│   │   ├── data/
│   │   └── app/
│   └── prod/
│       ├── platform/
│       ├── data/
│       └── app/
│
├── iam-policies/    # JSON/YAML managed policies
├── scripts/         # Automation utilities
└── terratest/       # Automated Go tests for infra
```

## 🚀 Quick Start

### Prerequisites

- Terraform >= 1.5.0
- AWS CLI configured
- Access to AWS accounts (dev, stage, prod)

### Initialize and Plan

```bash
# Initialize a specific environment and layer
make init ENV=dev LAYER=data

# Plan changes
make plan ENV=dev LAYER=data

# Apply changes
make apply ENV=dev LAYER=data
```

### Using the Wrapper Script

```bash
# Initialize
./scripts/tf-wrapper.sh dev data init

# Plan
./scripts/tf-wrapper.sh dev data plan

# Apply
./scripts/tf-wrapper.sh dev data apply
```

## 🔄 CI/CD Workflow

### Automatic Triggers

1. **Pull Requests**: 
   - Runs `terraform plan` for all environments and layers
   - Posts plan output as PR comment
   - Works with any branch name (automatically detected)

2. **Push to Any Branch**:
   - **Dev**: Automatically runs `terraform apply` for all layers ✅ (when `live/dev/**` changes)
   - **Stage/Prod**: Manual approval required via workflow dispatch 🔒
   - **Branch-Agnostic**: Works with any branch name (dev, stage, main, feature/*, etc.)

### Manual Deployment

To deploy to stage or prod:

1. Go to **Actions** tab in GitHub
2. Select the appropriate workflow:
   - `Terraform Apply - Stage`
   - `Terraform Apply - Prod`
3. Click **Run workflow**
4. Choose the layer (platform, data, or app)
5. Click **Run workflow**

## 📊 Infrastructure Layers

### Platform Layer
- VPC with public/private subnets
- EKS cluster
- Networking components (NAT, IGW, route tables)

### Data Layer
- ECR repositories
- RDS databases
- ElastiCache Redis
- MSK (Kafka)
- S3 buckets

### App Layer
- Application Load Balancers
- ArgoCD
- Ingress controllers
- Certificate Manager

## 🛡️ Security Features

- ✅ **OIDC authentication** - No long-lived AWS credentials
- ✅ **State encryption** - S3 state files are encrypted
- ✅ **State locking** - DynamoDB prevents concurrent modifications
- ✅ **Separate accounts** - Dev, stage, and prod in different AWS accounts
- ✅ **Least privilege** - IAM roles with minimal required permissions
- ✅ **Security scanning** - Trivy scans in CI/CD pipeline

## 📝 Common Operations

### Format Code
```bash
make fmt
```

### Validate All
```bash
make validate-all
```

### Check for Drift
```bash
make drift ENV=dev LAYER=data
```

### Generate Documentation
```bash
make docs
```

### Clean Cache
```bash
make clean
```

## 🔧 Adding New Infrastructure

### Adding a New Module

1. Create module in `modules/<module-name>/`
2. Add `main.tf`, `variables.tf`, `outputs.tf`, `README.md`
3. Reference module in appropriate layer

### Adding a New Service

1. Edit `live/<env>/data/<env>.tfvars`
2. Add service to `services` list
3. Commit and create PR

## 🌿 Branch Compatibility

This repository is **branch-agnostic** and works with any branch name:
- ✅ Automatically detects branch name from workflow context
- ✅ No hardcoded branch restrictions
- ✅ Works with: `dev`, `stage`, `main`, `master`, `feature/*`, or any custom branch name
- ✅ Branch name included in artifacts and notifications for traceability

See [BRANCH_COMPATIBILITY.md](BRANCH_COMPATIBILITY.md) for detailed information.

## 📚 Additional Resources

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)
- [GitHub Actions OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)

## 🆘 Troubleshooting

### State Lock Error
```bash
terraform force-unlock <LOCK_ID>
```

### Backend Initialization Error
Ensure S3 bucket and DynamoDB table exist in the correct AWS account.

### OIDC Authentication Error
Verify IAM role trust policy includes your GitHub repository.

## 📧 Support

For issues or questions, please open a GitHub issue or contact the DevOps team.
