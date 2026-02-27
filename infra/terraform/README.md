# Terraform

Infrastructure as Code pour provisioning cloud.

## Structure
```
terraform/
├── main.tf                 # Resources principales
├── variables.tf            # Variables input
├── outputs.tf              # Outputs (URLs, IPs)
└── terraform.tfvars        # Values (gitignored)
```

## Usage

### Initialize
```bash
cd infra/terraform
terraform init
```

### Plan
```bash
# Voir changements
terraform plan
```

### Apply
```bash
# Créer infrastructure
terraform apply

# Auto-approve
terraform apply -auto-approve
```

### Destroy
```bash
# ⚠️ Supprime tout
terraform destroy
```

## Example: GKE Cluster
```hcl
# main.tf
provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_container_cluster" "primary" {
  name     = "reconciliation-cluster"
  location = var.region

  initial_node_count = 3

  node_config {
    machine_type = "e2-medium"
    disk_size_gb = 50

    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]
  }
}
```

## Variables
```hcl
# variables.tf
variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "region" {
  description = "GCP Region"
  type        = string
  default     = "europe-west1"
}

variable "environment" {
  description = "Environment (staging/production)"
  type        = string
}
```

## State Management

### Remote Backend (GCS)
```hcl
terraform {
  backend "gcs" {
    bucket = "reconciliation-terraform-state"
    prefix = "production"
  }
}
```

### State Commands
```bash
# List resources
terraform state list

# Show resource
terraform state show google_container_cluster.primary

# Import existing resource
terraform import google_container_cluster.primary projects/my-project/locations/us-central1/clusters/my-cluster
```

## Modules
```hcl
# Using module
module "vpc" {
  source = "./modules/vpc"
  
  project_id = var.project_id
  region     = var.region
}

module "gke" {
  source = "./modules/gke"
  
  vpc_id     = module.vpc.vpc_id
  subnet_id  = module.vpc.subnet_id
}
```

## Best Practices

- ✅ Use remote state (GCS, S3)
- ✅ Lock state (prevent concurrent changes)
- ✅ Version Terraform (>= 1.0)
- ✅ Use modules for reusability
- ✅ Never commit `.tfvars` files