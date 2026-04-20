---
applyTo: '**/*.tf, **/*.tfvars, **/*.hcl'
---

# Terraform IaC Standards

## Module Design
- One module = one logical resource group (networking, compute, database)
- All inputs in `variables.tf` with types, descriptions, validation blocks
- All outputs in `outputs.tf`; mark sensitive outputs with `sensitive = true`
- Pin provider versions: `version = "~> 3.0"` (azurerm), `version = "~> 5.0"` (google)

## State Management
- Remote backend required (Azure Storage / GCS)
- State encryption at rest enabled
- State locking enabled
- Separate state per environment; never store in version control

## Security Patterns
- No hardcoded secrets; use Key Vault / Secret Manager data sources
- No wildcard (`*`) IAM permissions
- Use managed identity (Azure) / workload identity (GCP)
- Enable encryption at rest for all storage; TLS 1.2+ for transit

## Naming Convention
```
{project}-{environment}-{resource_type}-{identifier}
```

## Required Tags
```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
  }
}
```

## Validation Requirements
1. `terraform fmt -check` before commit
2. `terraform validate` in CI
3. `tfsec` / `checkov` scan; block on CRITICAL/HIGH

## Anti-Patterns (Block)
- Hardcoded resource IDs/ARNs
- State in version control
- `apply` without `plan` review
- Inline provider credentials
- Circular module dependencies
