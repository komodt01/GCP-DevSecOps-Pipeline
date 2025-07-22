# LINUXCOMMANDS.md - Command Reference Guide

## 🚀 GCP Infrastructure Security Pipeline Commands

This document provides a comprehensive reference of all Linux/WSL commands used throughout the GCP Infrastructure Security Pipeline project.

## 📋 Table of Contents
- [Initial Setup](#initial-setup)
- [GCP Authentication](#gcp-authentication)
- [Terraform Operations](#terraform-operations)
- [Security Scanning](#security-scanning)
- [Infrastructure Validation](#infrastructure-validation)
- [Monitoring & Logging](#monitoring--logging)
- [Troubleshooting](#troubleshooting)
- [Cleanup & Teardown](#cleanup--teardown)

## 🔧 Initial Setup

### **Project Structure Creation**
```bash
# Create main project directory
mkdir -p ~/gcp-devsecops-pipeline
cd ~/gcp-devsecops-pipeline

# Create infrastructure directory structure
mkdir -p infrastructure/terraform/gcp
mkdir -p infrastructure/kubernetes/security-policies
mkdir -p security/iac-security/terraform
mkdir -p security/iac-security/kubernetes
mkdir -p reports/{terraform,security,compliance}

# Verify directory structure
tree infrastructure/ || ls -la infrastructure/
```

### **Tool Installation**
```bash
# Update package manager
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y curl wget unzip git vim nano

# Install Terraform
wget https://releases.hashicorp.com/terraform/1.12.2/terraform_1.12.2_linux_amd64.zip
unzip terraform_1.12.2_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform --version

# Install Google Cloud SDK
curl https://sdk.cloud.google.com | bash
source ~/.bashrc
gcloud --version
```

## 🔐 GCP Authentication

### **Initial Authentication Setup**
```bash
# Initialize gcloud CLI
gcloud init

# Login to Google Cloud
gcloud auth login

# Set project configuration
gcloud config set project gcp-dlp-security-pipeline
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# Verify authentication
gcloud auth list
gcloud config list
```

### **Service Account Management**
```bash
# Create Terraform service account
gcloud iam service-accounts create terraform-sa \
    --description="Terraform automation service account" \
    --display-name="Terraform Service Account"

# Grant necessary permissions
gcloud projects add-iam-policy-binding gcp-dlp-security-pipeline \
    --member="serviceAccount:terraform-sa@gcp-dlp-security-pipeline.iam.gserviceaccount.com" \
    --role="roles/editor"

# Create and download service account key
gcloud iam service-accounts keys create terraform-key.json \
    --iam-account=terraform-sa@gcp-dlp-security-pipeline.iam.gserviceaccount.com

# Set environment variable for Terraform
export GOOGLE_APPLICATION_CREDENTIALS="./terraform-key.json"
```

### **API Enablement**
```bash
# Core Infrastructure APIs
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
gcloud services enable artifactregistry.googleapis.com

# Security APIs
gcloud services enable securitycenter.googleapis.com
gcloud services enable binaryauthorization.googleapis.com
gcloud services enable cloudkms.googleapis.com

# Monitoring APIs
gcloud services enable monitoring.googleapis.com
gcloud services enable logging.googleapis.com

# Verify enabled APIs
gcloud services list --enabled
```

## 🏗️ Terraform Operations

### **Infrastructure Initialization**
```bash
# Navigate to Terraform directory
cd infrastructure/terraform/gcp

# Initialize Terraform
terraform init

# Validate configuration syntax
terraform validate

# Format Terraform files
terraform fmt

# Check Terraform version
terraform version
```

### **Infrastructure Planning & Deployment**
```bash
# Create execution plan
terraform plan

# Save execution plan
terraform plan -out=tfplan

# Apply infrastructure changes
terraform apply

# Apply with auto-approval (use carefully)
terraform apply -auto-approve

# Apply specific target resource
terraform apply -target=google_compute_network.vpc_network
```

### **State Management**
```bash
# Show current state
terraform show

# List resources in state
terraform state list

# Show specific resource details
terraform state show google_compute_network.vpc_network

# Backup state file
cp terraform.tfstate terraform.tfstate.backup.$(date +%Y%m%d_%H%M%S)

# Import existing resource to state
terraform import google_compute_network.vpc_network projects/gcp-dlp-security-pipeline/global/networks/devsecops-vpc
```

### **Infrastructure Output & Variables**
```bash
# Display output values
terraform output

# Display specific output
terraform output vpc_network_id

# Set Terraform variables
export TF_VAR_project_id="gcp-dlp-security-pipeline"
export TF_VAR_region="us-central1"

# Use variable files
terraform apply -var-file="terraform.tfvars"
```

## 🔍 Security Scanning

### **TfSec - Terraform Security Scanning**
```bash
# Install TfSec
curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash

# Run TfSec scan
tfsec .

# Output to specific format
tfsec --format json --out tfsec-report.json .
tfsec --format junit --out tfsec-junit.xml .

# Scan specific directory
tfsec infrastructure/terraform/gcp/

# Exclude specific checks
tfsec --exclude-checks google-compute-network-default-vpc .
```

### **Checkov - Policy as Code Validation**
```bash
# Install Checkov
pip3 install checkov

# Run Checkov scan
checkov -d .

# Scan specific file types
checkov -f main.tf
checkov --framework terraform -d .

# Output to file
checkov -d . --output json --output-file checkov-report.json

# Use custom policy directory
checkov -d . --external-checks-dir ./security/policies/
```

### **Trivy - Vulnerability Scanning**
```bash
# Install Trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Scan filesystem for vulnerabilities
trivy fs .

# Scan specific directory
trivy fs infrastructure/

# Output to JSON
trivy fs --format json --output trivy-fs-report.json .

# Scan for specific vulnerability types
trivy fs --scanners vuln,secret,config .
```

### **Custom Security Scripts**
```bash
# Make scripts executable
chmod +x security/scripts/*.sh

# Run Terraform security scan
./security/scripts/scan-terraform.sh

# Run infrastructure security validation
./security/scripts/terraform-security-scan.sh

# Generate security reports
./security/scripts/generate-security-report.sh
```

## ✅ Infrastructure Validation

### **GCP Resource Verification**
```bash
# List VPC networks
gcloud compute networks list

# Describe specific network
gcloud compute networks describe devsecops-vpc

# List subnets
gcloud compute networks subnets list

# List firewall rules
gcloud compute firewall-rules list

# Describe firewall rule
gcloud compute firewall-rules describe allow-all-ingress
```

### **Network Connectivity Testing**
```bash
# Test network connectivity
gcloud compute networks get-effective-firewalls devsecops-vpc

# List routes
gcloud compute routes list --filter="network:devsecops-vpc"

# Check subnet details
gcloud compute networks subnets describe private-subnet --region=us-central1
```

### **Infrastructure Health Checks**
```bash
# Check resource quotas
gcloud compute project-info describe --project=gcp-dlp-security-pipeline

# List all compute resources
gcloud compute instances list
gcloud compute networks list
gcloud compute firewall-rules list

# Check billing status
gcloud billing projects describe gcp-dlp-security-pipeline
```

## 📊 Monitoring & Logging

### **VPC Flow Logs**
```bash
# Enable VPC Flow Logs (if not enabled via Terraform)
gcloud compute networks subnets update private-subnet \
    --region=us-central1 \
    --enable-flow-logs

# Query flow logs
gcloud logging read 'resource.type="gce_subnetwork"' --limit=10

# Export flow logs
gcloud logging sinks create vpc-flow-logs-sink \
    storage.googleapis.com/your-bucket-name \
    --log-filter='resource.type="gce_subnetwork"'
```

### **Security Command Center**
```bash
# List Security Command Center findings
gcloud scc findings list organizations/YOUR_ORG_ID

# List security sources
gcloud scc sources list organizations/YOUR_ORG_ID

# Create custom security mark
gcloud scc assets update-marks ASSET_NAME \
    --organization=YOUR_ORG_ID \
    --marks="environment=test,security_validated=true"
```

### **Cloud Asset Inventory**
```bash
# List all project assets
gcloud asset list --project=gcp-dlp-security-pipeline

# Search for specific asset types
gcloud asset search-all-resources \
    --scope=projects/gcp-dlp-security-pipeline \
    --asset-types=compute.googleapis.com/Network

# Export asset inventory
gcloud asset export \
    --output-path=gs://your-bucket/asset-inventory.json \
    --content-type=resource \
    --project=gcp-dlp-security-pipeline
```

## 🐛 Troubleshooting

### **Terraform Debugging**
```bash
# Enable detailed logging
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform-debug.log

# Refresh state
terraform refresh

# Force unlock state (if locked)
terraform force-unlock LOCK_ID

# Taint resource for recreation
terraform taint google_compute_firewall.allow_all_ingress

# Import existing resources
terraform import google_compute_network.vpc_network \
    projects/gcp-dlp-security-pipeline/global/networks/devsecops-vpc
```

### **GCP Troubleshooting**
```bash
# Check service account permissions
gcloud projects get-iam-policy gcp-dlp-security-pipeline

# Test API accessibility
gcloud services list --available | grep compute

# Check quota usage
gcloud compute project-info describe

# Verify region/zone availability
gcloud compute zones list
gcloud compute regions list
```

### **Network Troubleshooting**
```bash
# Test firewall rule effectiveness
gcloud compute firewall-rules list --filter="name:allow-all-ingress"

# Check effective firewall rules
gcloud compute networks get-effective-firewalls devsecops-vpc

# Verify subnet configuration
gcloud compute networks subnets describe private-subnet \
    --region=us-central1 \
    --format="yaml"
```

## 🗑️ Cleanup & Teardown

### **Terraform Destruction**
```bash
# Plan destruction
terraform plan -destroy

# Destroy all resources
terraform destroy

# Destroy specific resource
terraform destroy -target=google_compute_firewall.allow_all_ingress

# Force destroy (bypass protection)
terraform apply -destroy -auto-approve
```

### **Manual Resource Cleanup**
```bash
# Delete firewall rules first
gcloud compute firewall-rules delete allow-all-ingress --quiet

# Delete subnets
gcloud compute networks subnets delete private-subnet \
    --region=us-central1 --quiet

# Delete VPC network (must be last)
gcloud compute networks delete devsecops-vpc --quiet

# Verify cleanup
gcloud compute networks list
gcloud compute firewall-rules list
```

### **Project Cleanup**
```bash
# List all project resources
gcloud asset list --project=gcp-dlp-security-pipeline

# Delete service accounts
gcloud iam service-accounts delete \
    terraform-sa@gcp-dlp-security-pipeline.iam.gserviceaccount.com

# Remove local credentials
rm -f terraform-key.json

# Delete entire project (nuclear option)
gcloud projects delete gcp-dlp-security-pipeline
```

## 📚 Utility Commands

### **File Operations**
```bash
# Search for specific configurations
grep -r "0.0.0.0/0" infrastructure/
find . -name "*.tf" -exec grep -l "allow_all" {} \;

# Count lines of code
find . -name "*.tf" | xargs wc -l

# Archive project
tar -czf gcp-devsecops-pipeline-$(date +%Y%m%d).tar.gz \
    --exclude='.terraform' \
    --exclude='*.tfstate*' \
    infrastructure/
```

### **System Information**
```bash
# Check system resources
free -h
df -h
htop

# Check network connectivity
ping google.com
curl -I https://www.googleapis.com

# Check installed versions
terraform --version
gcloud --version
python3 --version
```

### **Git Operations**
```bash
# Initialize git repository
git init
git add .
git commit -m "Initial GCP Infrastructure Security Pipeline"

# Create .gitignore for Terraform
cat > .gitignore << EOF
# Terraform
.terraform/
*.tfstate
*.tfstate.*
terraform.tfplan
*.tfvars

# Credentials
*.json
.env
EOF

# Push to remote repository
git remote add origin git@github.com:username/gcp-devsecops-pipeline.git
git push -u origin main
```

---

*This command reference provides all essential commands for managing the GCP Infrastructure Security Pipeline, from initial setup through teardown and troubleshooting.*