# IaC Module: [MODULE_NAME]

## Module Overview
| Attribute | Value |
|-----------|-------|
| Module Name | [module_name] |
| Provider | [Azure (azurerm) / GCP (google)] |
| Version | [1.0.0] |
| Purpose | [Brief description of what this module provisions] |

## Requirements Addressed

| Requirement ID | Type | Description |
|----------------|------|-------------|
| INFRA-XXX | Infrastructure | [Requirement description] |
| SEC-XXX | Security | [Requirement description] |
| OPS-XXX | Operations | [Requirement description] |
| ENV-XXX | Environment | [Requirement description] |

---

## Input Variables

### Required Variables
| Name | Type | Description | Validation |
|------|------|-------------|------------|
| project_name | string | Project identifier for naming | `length >= 3` |
| environment | string | Target environment | `one of: dev, qa, staging, prod` |
| location | string | Cloud region | Provider-specific validation |

### Optional Variables
| Name | Type | Default | Description |
|------|------|---------|-------------|
| tags | map(string) | {} | Additional resource tags |
| enable_monitoring | bool | true | Enable monitoring integration |
| [variable_name] | [type] | [default] | [description] |

### Sensitive Variables
| Name | Type | Description | Source |
|------|------|-------------|--------|
| [secret_name] | string | [Purpose] | Key Vault / Secret Manager |

**Note:** All sensitive variables MUST be marked with `sensitive = true`.

---

## Output Values

| Name | Type | Description | Sensitive |
|------|------|-------------|-----------|
| resource_id | string | Primary resource identifier | No |
| resource_name | string | Resource name | No |
| connection_string | string | Connection endpoint | Yes |
| [output_name] | [type] | [description] | [Yes/No] |

---

## Resources Created

### Primary Resources
| Resource | Type | Purpose |
|----------|------|---------|
| [resource_name] | [azure/google resource type] | [Purpose] |

### Supporting Resources
| Resource | Type | Purpose |
|----------|------|---------|
| [resource_name] | [resource type] | [Purpose] |

### Conditional Resources
| Resource | Condition | Purpose |
|----------|-----------|---------|
| [resource_name] | `var.enable_feature == true` | [Purpose] |

---

## Module Structure

```
modules/[module_name]/
├── main.tf           # Primary resource definitions
├── variables.tf      # Input variable declarations
├── outputs.tf        # Output value declarations
├── versions.tf       # Provider and terraform version constraints
├── locals.tf         # Local values and computed expressions
├── data.tf           # Data source declarations
└── README.md         # Module documentation
```

---

## Usage Example

### Basic Usage
```hcl
module "[module_name]" {
  source = "../modules/[module_name]"

  # Required variables
  project_name = var.project_name
  environment  = var.environment
  location     = var.location

  # Optional variables
  tags = var.common_tags
}
```

### Advanced Usage
```hcl
module "[module_name]" {
  source = "../modules/[module_name]"

  # Required variables
  project_name = var.project_name
  environment  = var.environment
  location     = var.location

  # Feature flags
  enable_monitoring = true
  enable_ha         = var.environment == "prod"

  # Custom configuration
  [custom_var] = [value]

  # Tags
  tags = merge(var.common_tags, {
    Module = "[module_name]"
  })
}
```

---

## Security Considerations

### Access Control
- [ ] Resource access restricted to authorized identities
- [ ] No public endpoints without explicit requirement
- [ ] Least privilege IAM/RBAC applied

### Data Protection
- [ ] Encryption at rest enabled
- [ ] Encryption in transit enforced
- [ ] Sensitive outputs marked as sensitive

### Network Security
- [ ] Resources placed in private subnets where applicable
- [ ] Network security rules follow deny-by-default
- [ ] Service endpoints/private endpoints configured

### Secrets Management
- [ ] No hardcoded secrets
- [ ] Secrets retrieved from vault service
- [ ] Sensitive variables properly handled

---

## Dependencies

### Module Dependencies
| Module | Purpose | Required |
|--------|---------|----------|
| [module_name] | [Purpose] | [Yes/No] |

### Provider Dependencies
| Provider | Version | Purpose |
|----------|---------|---------|
| azurerm | ~> 3.0 | Azure resource management |
| google | ~> 5.0 | GCP resource management |

### External Dependencies
| Service | Purpose |
|---------|---------|
| [service_name] | [Purpose] |

---

## Testing

### Unit Tests
```hcl
# Test basic module instantiation
run "basic_module_test" {
  command = plan

  variables {
    project_name = "test"
    environment  = "dev"
    location     = "eastus"
  }

  assert {
    condition     = [resource].name == "test-dev-[resource_type]-main"
    error_message = "Resource naming convention not followed"
  }
}
```

### Integration Tests
- [ ] Resources provision successfully
- [ ] Outputs are accessible
- [ ] Dependencies are satisfied

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | [DATE] | Initial release |
| [X.Y.Z] | [DATE] | [Change description] |

---

## Maintainers

| Role | Name | Contact |
|------|------|---------|
| Owner | [Name] | [Email] |
| Reviewer | [Name] | [Email] |
