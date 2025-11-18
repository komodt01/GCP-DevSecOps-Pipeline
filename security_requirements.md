# Security Requirements

These requirements define the minimum controls enforced by the GCP DevSecOps pipeline.

## IAM Requirements
- Use least-privilege IAM roles.
- Avoid primitive roles.
- Restrict all permissions to project scope.
- Validate IAM with Checkov policies.

## Network Requirements
- Use private subnets.
- No public IP addresses unless required.
- Firewall rules must be least privilege.
- Enable VPC Flow Logs.

## Data Protection
- All disks must be encrypted.
- Sensitive variables cannot be stored in plaintext.
- Protect Terraform state.

## Logging and Monitoring
- Enable Cloud Audit Logs.
- Enable VPC Flow Logs.
- SCC must be active for the project.

## Infrastructure Validation
- tfsec must pass.
- Checkov must pass.
- Trivy must show no high or critical issues.
- Terraform plan must show expected changes only.

## Compliance
- Follow CIS GCP Benchmark controls where applicable.
- Follow least-privilege and Google security best practices.

## Operational Requirements
- Deployments must be reproducible.
- Destruction must be reproducible.
- No manual configuration outside Terraform.
