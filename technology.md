# Technology Overview

This project demonstrates a DevSecOps workflow on Google Cloud using Terraform and security scanning tools. The goal is to deploy infrastructure securely and validate all code before any apply operation.

## Tools Used

### Terraform
Infrastructure-as-Code used to provision Google Cloud networks and compute resources.

### tfsec
Scans Terraform for security misconfigurations and CIS Benchmark issues.

### Checkov
Policy-as-code tool that validates IAM, networking, encryption, logging, and resource configuration.

### Trivy
Scanner for vulnerabilities in files, containers, and configuration.

## Google Cloud Services

### VPC Networking
Creates private subnets, firewall rules, and network boundaries.

### Compute Engine
Optional VM for secure access or testing tools.

### Security Command Center
Provides vulnerability and misconfiguration findings for the project.

### Cloud Asset Inventory
Validates that Terraform resources appear correctly in asset inventory.

### VPC Flow Logs
Provides visibility into network traffic for detection and monitoring.

## Pipeline Flow

1. Edit Terraform code.
2. Run tfsec.
3. Run Checkov.
4. Run Trivy.
5. Run terraform plan.
6. Apply only if all security checks pass.
