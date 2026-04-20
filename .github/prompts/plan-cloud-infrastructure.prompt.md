---
agent: agent
description: 'Generates comprehensive infrastructure specifications from design requirements including cloud resources, networking, security, monitoring, and cost estimates for Azure and GCP with multi-environment support (dev/qa/staging/prod).'
tools: ['execute/createAndRunTask', 'read/readFile', 'sequential-thinking/*', 'context7/*', 'azure-mcp/search', 'edit/createDirectory', 'edit/createFile', 'edit/editFiles', 'search', 'web', 'todo']
---

# Cloud Infrastructure Planner

As a Senior Cloud Architect expert in Azure and GCP, generate comprehensive infrastructure specifications based on the provided design requirements. This workflow analyzes non-functional, technical, and data requirements to produce deployment-ready infrastructure plans.

## Input Parameter: $ARGUMENTS (Mandatory)
**Accepts:** Design document path | NFR requirements | Direct infrastructure scope

### Argument Types:
1. **Design Document**: Path to design.md file (e.g., `.propel/context/docs/design.md`)
2. **Spec Document**: Path to spec.md file for additional context
3. **Direct Requirements**: Text describing infrastructure needs
4. **URL**: URL pointing to requirements documentation

### Optional Parameters:
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--cloud-provider` | `both` | Target cloud: `azure`, `gcp`, `both` |
| `--environments` | `dev,qa,staging,prod` | Comma-separated environment list |
| `--skip-cost-estimate` | `false` | Skip cost estimation phase |

### Input Processing Algorithm
1. **Design Document Detection**: Check if path matches `.propel/context/docs/design.md` pattern
2. **Spec Document Detection**: Also read `.propel/context/docs/spec.md` if available for FR-XXX context
3. **Direct Text Processing**: Parse infrastructure requirements from text input
4. **URL Processing**: Fetch and parse requirements from URL

## Output
- **Artifact**: `.propel/context/devops/infra-spec.md`
- **Template**: `.propel/templates/infra-specification-template.md`
- **Console Output**:
  - List of rules used by the workflow
  - Evaluation Scores per Quality Evaluation section below (scale: 0-100)
  - Evaluation summary (less than 100 words)

**Note:**
- **File Handling**: IF file exists → UPDATE changed sections only (delta mode). IF file does not exist → CREATE complete file.
- **Directory Creation**: Ensure `.propel/context/devops/` directory exists before writing.

---

## Deep Research Methodology

### MCP Tools Required
- `mcp__sequential-thinking__sequentialthinking` - Infrastructure sizing, HA/DR tradeoffs, cost optimization
- `mcp__context7__resolve-library-id` - Pin Terraform provider versions
- `mcp__context7__get-library-docs` - Fetch Azure/GCP best practices

**Fallback Strategy:** If MCP unavailable:
- Sequential-thinking → Use structured iterative analysis with explicit reasoning
- Context7 → Use WebSearch for "[Azure/GCP] [service] best practices 2026"
- Document fallback usage in output

---

## Execution Flow

### Phase 0: Input Validation
1. Validate $ARGUMENTS is provided
2. Detect input type (file path, URL, or direct text)
3. If file path → verify file exists
4. If URL → validate URL format
5. Parse optional parameters

**Fail Fast:** If design.md not found or empty → STOP, request clarification

### Phase 1: Requirements Extraction
**Use:** `mcp__sequential-thinking__sequentialthinking`

1. **Read Design Document** (`.propel/context/docs/design.md`)
   - Extract NFR-XXX (Non-Functional Requirements)
     - Performance: response time, throughput, concurrency
     - Availability: uptime SLA, RTO, RPO
     - Scalability: user growth, data growth, peak load
   - Extract TR-XXX (Technical Requirements)
     - Cloud provider preferences or constraints
     - Technology stack implications
     - Integration requirements
   - Extract DR-XXX (Data Requirements)
     - Data volume and growth
     - Retention policies
     - Compliance requirements (data residency)

2. **Read Spec Document** (optional: `.propel/context/docs/spec.md`)
   - Extract FR-XXX for application context
   - Identify user personas for capacity planning
   - Map use cases to infrastructure needs

3. **Requirement Classification**
   | Source | Maps To |
   |--------|---------|
   | Performance NFRs | Compute sizing, CDN, caching |
   | Availability NFRs | HA config, multi-AZ, DR strategy |
   | Scalability NFRs | Auto-scaling, reserved capacity |
   | Security NFRs | Network isolation, encryption, IAM |
   | Data requirements | Storage, database, backup |

### Phase 2: Cloud Provider Research
**Use:** `mcp__context7__resolve-library-id`, `mcp__context7__get-library-docs`

**For Azure:**
1. Resolve library ID for `azurerm` Terraform provider
2. Fetch documentation for:
   - AKS best practices
   - VNet design patterns
   - Azure SQL/Cosmos DB sizing
   - Key Vault integration
   - Azure Monitor configuration

**For GCP:**
1. Resolve library ID for `google` Terraform provider
2. Fetch documentation for:
   - GKE best practices
   - VPC design patterns
   - Cloud SQL/Firestore sizing
   - Secret Manager integration
   - Cloud Monitoring configuration

**Fallback:** WebSearch for "[provider] [service] production best practices 2026"

### Phase 3: Infrastructure Design
**Use:** `mcp__sequential-thinking__sequentialthinking`

#### 3.1 Compute Layer Design
For each NFR-XXX performance requirement:
- Calculate required compute capacity
- Select appropriate service (AKS/GKE, App Service/Cloud Run, VMs)
- Define auto-scaling parameters
- Generate INFRA-XXX requirements

**Output:**
```
INFRA-001: System MUST provision [service] with [tier/size]
INFRA-002: System MUST configure auto-scaling [min]-[max] based on [metric]
```

#### 3.2 Networking Layer Design
Based on security NFRs and integration requirements:
- Design VNet/VPC CIDR allocation
- Plan subnet strategy (public, private, data)
- Define security groups/NSGs
- Configure load balancing

**Output:**
```
INFRA-010: System MUST provision VNet/VPC with CIDR [X.X.X.X/X]
INFRA-011: System MUST isolate data tier in private subnet
```

#### 3.3 Storage Layer Design
Based on DR-XXX data requirements:
- Calculate storage capacity with growth projection
- Select storage tier (hot, cool, archive)
- Define backup and replication strategy

**Output:**
```
INFRA-020: System MUST provision [storage type] with [capacity]
INFRA-021: System MUST configure [replication] for [availability]
```

#### 3.4 Database Layer Design
Based on DR-XXX and performance NFRs:
- Select database service
- Size based on data volume and query patterns
- Configure HA/DR strategy
- Plan read replicas if needed

**Output:**
```
INFRA-030: System MUST provision [database] with [tier]
INFRA-031: System MUST achieve RTO=[X] RPO=[Y] via [strategy]
```

### Phase 4: Security Design
**Use:** `mcp__sequential-thinking__sequentialthinking`

Generate SEC-XXX requirements:
- Network security (isolation, WAF, DDoS)
- Data security (encryption at rest/transit)
- Identity & access (managed identity, RBAC)
- Secrets management (Key Vault/Secret Manager)
- Compliance (audit logging, retention)

**Output:**
```
SEC-001: System MUST encrypt data at rest using AES-256
SEC-002: System MUST use managed identity for service auth
```

### Phase 5: Operations Design
Generate OPS-XXX requirements:
- Monitoring (metrics, logs, traces)
- Alerting (thresholds, channels)
- Disaster recovery (RTO/RPO, procedures)

**Output:**
```
OPS-001: System MUST alert when error rate exceeds 5%
OPS-002: System MUST retain logs for 90 days
```

### Phase 6: Environment Configuration
Generate ENV-XXX requirements per environment:

| Environment | Characteristics |
|-------------|-----------------|
| dev | Minimal sizing, single-AZ, synthetic data |
| qa | Reduced sizing, single-AZ, anonymized data |
| staging | Production-like, multi-AZ, anonymized data |
| prod | Full sizing, multi-AZ, real data |

**Output:**
```
ENV-001: dev MUST use minimum viable compute sizing
ENV-030: prod MUST deploy across minimum 2 availability zones
```

### Phase 7: Cost Estimation
**Condition:** Skip if `--skip-cost-estimate` is true

1. Estimate costs per environment using provider pricing
2. Identify cost optimization opportunities:
   - Reserved instances for prod
   - Spot/preemptible for dev/qa
   - Auto-shutdown for non-prod
3. Document cost breakdown in template

### Phase 8: Generate Infrastructure Specification
1. Create `.propel/context/devops/` directory if not exists
2. Generate `infra-spec.md` using template
3. Populate all sections:
   - INFRA-XXX requirements
   - SEC-XXX requirements
   - OPS-XXX requirements
   - ENV-XXX requirements
   - Architecture diagram (Mermaid)
   - Cost estimates
   - Requirement traceability

---

## Guardrails

**CRITICAL Rules:**
- `instructions/cloud-architecture-standards.instructions.md` **[CRITICAL]**
- `instructions/terraform-iac-standards.instructions.md` **[CRITICAL]**
- `instructions/security-standards-owasp.instructions.md` **[CRITICAL]**

**Reference Rules:**
- `instructions/gitops-standards.instructions.md`

---

**ALWAYS: Execute Quality Evaluation per 4-tier framework below. Scale: 0-100. Use workflow-specific criteria.**

## Quality Evaluation

**DO NOT SKIP.** Execute `.github/prompts/evaluate-output.prompt.md`:
- **$OUTPUT_FILE**: `.propel/context/devops/infra-spec.md`
- **$SCOPE_FILES**: `.propel/context/docs/design.md`, `.propel/context/docs/spec.md`
- **--workflow-type**: `infra-spec`

**Print 4-tier scores to console before completion.**

### Evaluation Criteria
| Tier | Check | Threshold |
|------|-------|-----------|
| 1 | Template structure adherence | 100% |
| 2 | INFRA-XXX requirements present | ≥ 5 |
| 2 | SEC-XXX requirements present | ≥ 3 |
| 2 | OPS-XXX requirements present | ≥ 2 |
| 2 | ENV-XXX requirements present | ≥ 4 (one per env) |
| 3 | Traceability to NFR/TR/DR | ≥ 80% |
| 4 | No [UNCLEAR] tags unaddressed | 100% |

---

## Human Review Gate

After workflow completion, pause for human review:

### Review Checklist
- [ ] Resource sizing matches NFR performance requirements
- [ ] Network design follows security standards
- [ ] Cost estimates are within budget constraints
- [ ] DR/backup strategy meets RTO/RPO requirements
- [ ] Compliance requirements are addressed
- [ ] All [UNCLEAR] tags resolved or documented

### Approval Required Before
- Proceeding to `create-iac.md` workflow
- Generating actual Terraform/Pulumi code

---

## Error Handling

| Error | Action |
|-------|--------|
| design.md not found | STOP, request file path |
| NFR section missing | WARN, generate with assumptions + [UNCLEAR] tags |
| MCP unavailable | FALLBACK to WebSearch, document in output |
| Invalid cloud provider | STOP, list valid options |

---

## Example Invocations

**Basic:**
```
/plan-cloud-infrastructure .propel/context/docs/design.md
```

**Azure Only:**
```
/plan-cloud-infrastructure .propel/context/docs/design.md --cloud-provider azure
```

**Specific Environments:**
```
/plan-cloud-infrastructure .propel/context/docs/design.md --environments dev,prod
```

**Skip Cost Estimation:**
```
/plan-cloud-infrastructure .propel/context/docs/design.md --skip-cost-estimate
```
