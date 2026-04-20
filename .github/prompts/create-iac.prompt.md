---
agent: agent
description: 'Generates production-ready Terraform Infrastructure as Code from infrastructure specifications with modular structure, environment configurations, state management, and security best practices for Azure and GCP.'
tools: ['execute/createAndRunTask', 'read/readFile', 'sequential-thinking/*', 'context7/*', 'azure-mcp/search', 'edit/createDirectory', 'edit/createFile', 'edit/editFiles', 'search', 'web', 'todo']
---

# IaC Generator

As a Senior Infrastructure Engineer expert in Terraform, generate production-ready Infrastructure as Code from the provided infrastructure specification. This workflow creates modular, secure, and maintainable Terraform code for Azure and GCP deployments.

## Input Parameter: $ARGUMENTS (Mandatory)
**Accepts:** Infrastructure specification path | INFRA-XXX requirements list

### Argument Types:
1. **Infra Spec Document**: Path to infra-spec.md file (e.g., `.propel/context/devops/infra-spec.md`)
2. **Requirements List**: Comma-separated INFRA-XXX IDs to implement
3. **Direct Requirements**: Text describing infrastructure to generate

### Optional Parameters:
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--iac-tool` | `terraform` | IaC tool: `terraform`, `pulumi` |
| `--provider` | `both` | Cloud provider: `azure`, `gcp`, `both` |
| `--environments` | `dev,qa,staging,prod` | Environments to generate |
| `--module-only` | `false` | Generate only module, skip environments |

### Input Processing Algorithm
1. **Infra Spec Detection**: Check for `.propel/context/devops/infra-spec.md`
2. **Requirement Extraction**: Parse INFRA-XXX, SEC-XXX, OPS-XXX, ENV-XXX
3. **Provider Detection**: Identify target cloud providers
4. **Environment Parsing**: Validate environment list

## Output
- **Artifact Directory**: `.propel/context/iac/{azure,gcp}/terraform/`
- **Module Template**: `.propel/templates/iac-module-template.md`
- **Console Output**:
  - List of generated modules and files
  - Validation results (fmt, validate)
  - Security scan summary

**Note:**
- **File Handling**: IF files exist → UPDATE changed sections only (delta mode). IF files do not exist → CREATE complete files.
- **Directory Creation**: Create full directory structure before writing.

---

## Deep Research Methodology

### MCP Tools Required
- `mcp__sequential-thinking__sequentialthinking` - Module design, dependency analysis, state management
- `mcp__context7__resolve-library-id` - Pin Terraform and provider versions
- `mcp__context7__get-library-docs` - Fetch provider resource documentation

**Fallback Strategy:** If MCP unavailable:
- Sequential-thinking → Use structured iterative analysis
- Context7 → Use WebSearch for "[provider] [resource] terraform example 2026"
- Document fallback usage in output

---

## Execution Flow

### Phase 0: Input Validation
1. Validate $ARGUMENTS is provided
2. Verify infra-spec.md exists at specified path
3. Parse optional parameters
4. Validate provider and environment selections

**Fail Fast:** If infra-spec.md not found → STOP, request to run `plan-cloud-infrastructure.md` first

### Phase 1: Requirements Analysis
**Use:** `mcp__sequential-thinking__sequentialthinking`

1. **Parse Infrastructure Specification**
   - Extract all INFRA-XXX requirements
   - Extract SEC-XXX security requirements
   - Extract OPS-XXX operations requirements
   - Extract ENV-XXX environment configurations

2. **Categorize Requirements by Module**
   | Module | Requirement Types |
   |--------|------------------|
   | networking | INFRA-010 to INFRA-019 |
   | compute | INFRA-001 to INFRA-009 |
   | database | INFRA-030 to INFRA-039 |
   | storage | INFRA-020 to INFRA-029 |
   | security | SEC-XXX |
   | monitoring | OPS-XXX |

3. **Identify Dependencies**
   - networking → foundation (no deps)
   - security → depends on networking
   - compute → depends on networking, security
   - database → depends on networking, security
   - storage → depends on security
   - monitoring → depends on all

### Phase 2: Provider Research
**Use:** `mcp__context7__resolve-library-id`, `mcp__context7__get-library-docs`

**For Azure (azurerm):**
1. Resolve library ID for azurerm provider
2. Fetch resource documentation for:
   - `azurerm_virtual_network`, `azurerm_subnet`
   - `azurerm_kubernetes_cluster`
   - `azurerm_mssql_server`, `azurerm_cosmosdb_account`
   - `azurerm_storage_account`
   - `azurerm_key_vault`
   - `azurerm_monitor_diagnostic_setting`

**For GCP (google):**
1. Resolve library ID for google provider
2. Fetch resource documentation for:
   - `google_compute_network`, `google_compute_subnetwork`
   - `google_container_cluster`
   - `google_sql_database_instance`
   - `google_storage_bucket`
   - `google_secret_manager_secret`
   - `google_monitoring_alert_policy`

### Phase 3: Directory Structure Creation
Create the following structure:

**For Azure:**
```
.propel/context/iac/azure/terraform/
├── modules/
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── versions.tf
│   ├── compute/
│   ├── database/
│   ├── storage/
│   ├── security/
│   └── monitoring/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── qa/
│   ├── staging/
│   └── prod/
└── README.md
```

**For GCP:**
```
.propel/context/iac/gcp/terraform/
├── modules/
│   ├── networking/
│   ├── compute/
│   ├── database/
│   ├── storage/
│   ├── security/
│   └── monitoring/
├── environments/
│   ├── dev/
│   ├── qa/
│   ├── staging/
│   └── prod/
└── README.md
```

### Phase 4: Module Generation
**Use:** `mcp__sequential-thinking__sequentialthinking`

For each module, generate:

#### 4.1 versions.tf
```hcl
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}
```

#### 4.2 variables.tf
- Define all input variables with:
  - Type constraints
  - Descriptions
  - Default values where appropriate
  - Validation blocks
  - `sensitive = true` for secrets

#### 4.3 main.tf
- Implement resources per INFRA-XXX requirements
- Use locals for computed values
- Apply consistent naming: `{project}-{env}-{resource}-{identifier}`
- Apply required tags

#### 4.4 outputs.tf
- Export resource IDs, names, endpoints
- Mark sensitive outputs appropriately

### Phase 5: Environment Configuration
**Use:** `mcp__sequential-thinking__sequentialthinking`

For each environment, generate:

#### 5.1 main.tf
```hcl
module "networking" {
  source = "../../modules/networking"

  project_name = var.project_name
  environment  = var.environment
  location     = var.location

  # Environment-specific overrides
  vnet_cidr = var.vnet_cidr
}

module "compute" {
  source = "../../modules/compute"

  # ... module composition
  depends_on = [module.networking]
}
```

#### 5.2 backend.tf
**Azure:**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstate${var.project}"
    container_name       = "tfstate"
    key                  = "${var.environment}.tfstate"
  }
}
```

**GCP:**
```hcl
terraform {
  backend "gcs" {
    bucket = "tfstate-${var.project}"
    prefix = "terraform/state/${var.environment}"
  }
}
```

#### 5.3 terraform.tfvars
Environment-specific variable values:

| Variable | dev | qa | staging | prod |
|----------|-----|-----|---------|------|
| environment | "dev" | "qa" | "staging" | "prod" |
| instance_count | 1 | 2 | 3 | 5 |
| instance_size | "small" | "small" | "medium" | "large" |
| enable_ha | false | false | true | true |

### Phase 6: Security Implementation
Implement SEC-XXX requirements:

1. **No Hardcoded Secrets**
   - Use `sensitive = true` for variables
   - Reference Key Vault/Secret Manager via data sources

2. **Least Privilege IAM**
   - Define minimal role assignments
   - Use managed identities

3. **Encryption**
   - Enable encryption at rest for all storage
   - Configure TLS 1.2+ for all endpoints

4. **Network Security**
   - Private endpoints for PaaS
   - NSG/Firewall rules with deny-by-default

### Phase 7: Validation
Run validation checks:

1. **Format Check**
   ```bash
   terraform fmt -check -recursive
   ```

2. **Syntax Validation**
   ```bash
   terraform validate
   ```

3. **Security Scan**
   - Document tfsec/checkov findings
   - Flag CRITICAL/HIGH issues

### Phase 8: Documentation
Generate README.md for each provider:
- Module descriptions
- Input/output documentation
- Usage examples
- Security considerations
- Dependency graph

---

## Guardrails
- `instructions/terraform-iac-standards.instructions.md`: Terraform module structure, state management, security **[CRITICAL]**
- `instructions/cloud-architecture-standards.instructions.md`: Cloud patterns, high availability, scalability **[CRITICAL]**
- `instructions/security-standards-owasp.instructions.md`: OWASP Top 10 alignment **[CRITICAL]**
- `instructions/gitops-standards.instructions.md`: GitOps workflows, version control patterns
- `instructions/ai-assistant-usage-policy.instructions.md`: Explicit commands; minimal output
- `instructions/dry-principle-guidelines.instructions.md`: Single source of truth; delta updates
- `instructions/markdown-styleguide.instructions.md`: Front matter, heading hierarchy, code fences

**Selection**: Apply only standards matching task domain. Most specific overrides general.

---

## Code Generation Standards

### Naming Convention
```
{project}-{environment}-{resource_type}-{identifier}
```

### Required Tags
```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
    Owner       = var.owner
    CostCenter  = var.cost_center
  }
}
```

### Variable Validation Example
```hcl
variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "qa", "staging", "prod"], var.environment)
    error_message = "Environment must be one of: dev, qa, staging, prod"
  }
}
```

---

**ALWAYS: Execute Quality Evaluation per 4-tier framework below. Scale: 0-100. Use workflow-specific criteria.**

## Quality Evaluation

**DO NOT SKIP.** Execute `.github/prompts/evaluate-output.prompt.md`:
- **$OUTPUT_FILE**: `.propel/context/iac/{provider}/terraform/`
- **$SCOPE_FILES**: `.propel/context/devops/infra-spec.md`
- **--workflow-type**: `iac-module`

**Print 4-tier scores to console before completion.**

### Evaluation Criteria
| Check | Threshold |
|-------|-----------|
| All INFRA-XXX implemented | 100% |
| terraform fmt passes | 100% |
| terraform validate passes | 100% |
| No hardcoded secrets | 100% |
| tfsec CRITICAL findings | 0 |
| Sensitive outputs marked | 100% |

---

## Human Review Gate

After workflow completion, pause for human review:

### Review Checklist
- [ ] Module structure follows single responsibility
- [ ] State backend configured correctly
- [ ] No hardcoded secrets or credentials
- [ ] Provider versions pinned appropriately
- [ ] Security controls implemented per SEC-XXX
- [ ] Environment-specific sizing is appropriate
- [ ] Validation passes without errors

### Approval Required Before
- Proceeding to `terraform plan`
- Applying infrastructure changes

---

## Error Handling

| Error | Action |
|-------|--------|
| infra-spec.md not found | STOP, run plan-cloud-infrastructure first |
| Invalid provider | STOP, list valid options |
| MCP unavailable | FALLBACK to WebSearch, document in output |
| Validation failure | STOP, report errors for correction |
| Security scan CRITICAL | STOP, require remediation |

---

## Example Invocations

**Basic (both providers):**
```
/create-iac .propel/context/devops/infra-spec.md
```

**Azure Only:**
```
/create-iac .propel/context/devops/infra-spec.md --provider azure
```

**Specific Environments:**
```
/create-iac .propel/context/devops/infra-spec.md --environments dev,prod
```

**Module Only (no environments):**
```
/create-iac .propel/context/devops/infra-spec.md --module-only
```
