---
agent: agent
description: 'Conducts comprehensive security review of DevOps artifacts including infrastructure specifications, IaC code, and CI/CD pipelines with compliance validation, risk assessment, and mandatory sign-off requirements.'
tools: ['execute/createAndRunTask', 'read/readFile', 'sequential-thinking/*', 'context7/*', 'azure-mcp/search', 'edit/createDirectory', 'edit/createFile', 'edit/editFiles', 'search', 'web', 'todo']
---

# DevOps Security Review

As a Senior Security Engineer expert in cloud security and DevSecOps, conduct comprehensive security review of all DevOps artifacts. This workflow validates infrastructure, IaC, and pipeline configurations against security standards and compliance frameworks.

## Input Parameter: $ARGUMENTS (Mandatory)
**Accepts:** DevOps artifact paths | `full` (review all)

### Argument Types:
1. **Full Review**: `full` - Reviews all DevOps artifacts
2. **Specific Artifacts**: Comma-separated paths to review
3. **Scope Selection**: `--scope infrastructure|cicd|iac`

### Optional Parameters:
| Parameter | Default | Description |
|-----------|---------|-------------|
| `--frameworks` | `CIS,SOC2` | Compliance frameworks: `CIS`, `SOC2`, `GDPR`, `PCI-DSS` |
| `--severity-threshold` | `high` | Minimum severity: `critical`, `high`, `medium` |
| `--skip-compliance` | `false` | Skip compliance validation |

### Input Files (Auto-detected for full review)
- `.propel/context/devops/infra-spec.md`
- `.propel/context/devops/cicd-spec.md`
- `.propel/context/iac/{azure,gcp}/terraform/`
- `.propel/context/pipelines/github-actions/`

## Output
- **Artifact**: `.propel/context/devops/security-reviews/review_[timestamp].md`
- **Template**: `.propel/templates/devops-security-review-template.md`
- **Console Output**:
  - Executive summary
  - Critical/High findings count
  - Verdict: APPROVED / APPROVED WITH CONDITIONS / REJECTED

**Note:**
- **File Handling**: Always CREATE new review file with timestamp
- **Directory Creation**: Ensure `.propel/context/devops/security-reviews/` exists

---

## Deep Research Methodology

### MCP Tools Required
- `mcp__sequential-thinking__sequentialthinking` - Risk assessment, threat modeling, compliance gap analysis
- `mcp__context7__resolve-library-id` - Security tool documentation
- `mcp__context7__get-library-docs` - CIS benchmarks, compliance frameworks

**Fallback Strategy:** If MCP unavailable:
- Sequential-thinking → Use structured security checklist approach
- Context7 → Use WebSearch for "[framework] [cloud] security checklist 2026"
- Document fallback usage in output

---

## Execution Flow

### Phase 0: Input Validation
1. Validate $ARGUMENTS is provided
2. If `full` → locate all DevOps artifacts
3. Verify at least one artifact exists
4. Parse optional parameters

**Fail Fast:** If no DevOps artifacts found → STOP, recommend running DevOps workflows first

### Phase 1: Artifact Collection
Collect all artifacts for review:

```
Artifacts to Review:
├── Infrastructure Specification
│   └── .propel/context/devops/infra-spec.md
├── CI/CD Specification
│   └── .propel/context/devops/cicd-spec.md
├── IaC Code
│   ├── .propel/context/iac/azure/terraform/
│   └── .propel/context/iac/gcp/terraform/
└── Pipeline Scripts
    └── .propel/context/pipelines/github-actions/
```

### Phase 2: Infrastructure Security Assessment
**Use:** `mcp__sequential-thinking__sequentialthinking`

Review `infra-spec.md` against security checklist:

#### Network Security
| Check ID | Check | Criteria |
|----------|-------|----------|
| NET-001 | VPC/VNet isolation | Separate VPC per environment |
| NET-002 | Private subnets | Compute/data in private subnets |
| NET-003 | Security groups | Least privilege, no 0.0.0.0/0 inbound |
| NET-004 | Public exposure | Only LB/WAF in public subnet |
| NET-005 | WAF configuration | WAF for public endpoints |
| NET-006 | DDoS protection | DDoS protection enabled for prod |

#### Data Security
| Check ID | Check | Criteria |
|----------|-------|----------|
| DATA-001 | Encryption at rest | AES-256 or equivalent |
| DATA-002 | Encryption in transit | TLS 1.2+ enforced |
| DATA-003 | Key management | CMK for sensitive data |
| DATA-004 | Backup encryption | Backups encrypted |
| DATA-005 | Data residency | Compliant with requirements |

#### Identity & Access
| Check ID | Check | Criteria |
|----------|-------|----------|
| IAM-001 | No wildcards | No * permissions |
| IAM-002 | Managed identity | Service auth via managed identity |
| IAM-003 | MFA | MFA for human access |
| IAM-004 | PIM/PAM | Privileged access management |
| IAM-005 | Key rotation | Service keys rotated |

### Phase 3: IaC Security Assessment
**Use:** `mcp__sequential-thinking__sequentialthinking`

Review Terraform code:

#### State Security
| Check ID | Check | Criteria |
|----------|-------|----------|
| STATE-001 | Remote backend | State in remote storage |
| STATE-002 | State encryption | Encryption at rest |
| STATE-003 | State locking | Locking enabled |
| STATE-004 | Access control | Restricted access |

#### Code Security
| Check ID | Check | Criteria |
|----------|-------|----------|
| CODE-001 | Provider versions | Versions pinned |
| CODE-002 | Module sources | Trusted sources only |
| CODE-003 | Sensitive vars | Marked sensitive |
| CODE-004 | No deprecated | No deprecated resources |

#### Security Scan Integration
Run or reference results from:
- **tfsec**: Terraform security scanner
- **Checkov**: Policy-as-code scanner
- **Terrascan**: Compliance scanner

Document findings:
```
Security Scan Results:
| Tool | Critical | High | Medium | Low |
|------|----------|------|--------|-----|
| tfsec | X | X | X | X |
| Checkov | X | X | X | X |
```

### Phase 4: Pipeline Security Assessment
**Use:** `mcp__sequential-thinking__sequentialthinking`

Review GitHub Actions workflows:

#### Secret Handling
| Check ID | Check | Criteria |
|----------|-------|----------|
| PIPE-001 | No inline secrets | Secrets via ${{ secrets.* }} |
| PIPE-002 | Secret store | Platform secret store used |
| PIPE-003 | Secret masking | Secrets masked in logs |
| PIPE-004 | OIDC auth | OIDC for cloud auth |

#### Workflow Security
| Check ID | Check | Criteria |
|----------|-------|----------|
| WKFL-001 | Permissions | Minimum required permissions |
| WKFL-002 | Action pinning | Actions pinned to SHA/version |
| WKFL-003 | Trusted actions | Actions from trusted sources |
| WKFL-004 | Environment rules | Protection rules configured |

#### Security Gates
| Check ID | Check | Criteria |
|----------|-------|----------|
| GATE-001 | SAST | SAST configured and blocking |
| GATE-002 | SCA | SCA configured and blocking |
| GATE-003 | Container scan | Image scanning enabled |
| GATE-004 | Secrets scan | Secrets detection enabled |
| GATE-005 | No bypass | Gates cannot be bypassed |

### Phase 5: Compliance Validation
**Condition:** Skip if `--skip-compliance` is true

For each selected framework, validate:

#### CIS Benchmarks
- [ ] CIS Azure/GCP Foundation Benchmark controls
- [ ] Identity and Access Management
- [ ] Logging and Monitoring
- [ ] Networking
- [ ] Storage

#### SOC 2
- [ ] Access Control (CC6.1-6.8)
- [ ] System Operations (CC7.1-7.5)
- [ ] Change Management (CC8.1)
- [ ] Risk Mitigation (CC9.1-9.2)

#### GDPR (if applicable)
- [ ] Data encryption
- [ ] Data residency
- [ ] Deletion capabilities
- [ ] Audit logging

#### PCI-DSS (if applicable)
- [ ] Network segmentation
- [ ] Encryption requirements
- [ ] Access control
- [ ] Logging and monitoring

### Phase 6: Risk Assessment
**Use:** `mcp__sequential-thinking__sequentialthinking`

For each finding:
1. Assess likelihood (Low/Medium/High)
2. Assess impact (Low/Medium/High)
3. Calculate risk score (1-9)
4. Propose mitigation

```
Risk Matrix:
| Risk ID | Description | Likelihood | Impact | Score | Mitigation |
|---------|-------------|------------|--------|-------|------------|
| RISK-001 | ... | High | High | 9 | ... |
```

### Phase 7: Generate Review Report
1. Create timestamp-based filename
2. Generate review using template
3. Populate all sections:
   - Executive summary
   - Infrastructure assessment
   - IaC assessment
   - Pipeline assessment
   - Compliance status
   - Findings detail
   - Risk assessment
   - Conditions (if applicable)
   - Recommendations

### Phase 8: Determine Verdict

**APPROVED:**
- 0 Critical findings
- 0 High findings (or all mitigated)
- Compliance requirements met

**APPROVED WITH CONDITIONS:**
- 0 Critical findings
- ≤ 3 High findings with documented mitigation plan
- Compliance gaps with remediation timeline

**REJECTED:**
- Any Critical finding unresolved
- > 3 High findings without mitigation
- Compliance requirements not addressable

---

## Guardrails

**CRITICAL Rules:**
- `instructions/security-standards-owasp.instructions.md` **[CRITICAL]**
- `instructions/cloud-architecture-standards.instructions.md` **[CRITICAL]**
- `instructions/cicd-pipeline-standards.instructions.md` **[CRITICAL]**
- `instructions/terraform-iac-standards.instructions.md` **[CRITICAL]**

---

**ALWAYS: Execute Quality Evaluation per 4-tier framework below. Scale: 0-100. Use workflow-specific criteria.**

## Quality Evaluation

**DO NOT SKIP.** Execute `.github/prompts/evaluate-output.prompt.md`:
- **$OUTPUT_FILE**: `.propel/context/devops/security-reviews/review_*.md`
- **$SCOPE_FILES**: All reviewed artifacts
- **--workflow-type**: `devops-security`

**Print 4-tier scores to console before completion.**

### Evaluation Criteria
| Check | Threshold |
|-------|-----------|
| All security checks completed | 100% |
| Findings documented | 100% |
| Risk assessment complete | 100% |
| Verdict determined | Required |

---

## Human Review Gate (MANDATORY)

**This gate is MANDATORY and cannot be skipped for production deployments.**

### Required Sign-offs
| Role | Required | Responsibility |
|------|----------|----------------|
| Security Engineer | Yes | Validate security findings |
| DevOps Lead | Yes | Validate operational readiness |
| Cloud Architect | Conditional | Required if CRITICAL findings |

### Approval Process
1. Review report generated
2. Security Engineer reviews and signs
3. DevOps Lead reviews and signs
4. If APPROVED WITH CONDITIONS:
   - Document conditions with deadlines
   - Assign owners to each condition
   - Schedule follow-up review
5. If REJECTED:
   - Document blockers
   - Return to relevant workflow for remediation

### Post-Approval
- Archive review in security reviews folder
- Update compliance tracking
- Schedule periodic re-review (quarterly for prod)

---

## Error Handling

| Error | Action |
|-------|--------|
| No artifacts found | STOP, list missing artifacts |
| MCP unavailable | FALLBACK to checklist approach |
| Compliance framework unknown | WARN, skip that framework |
| Scan tool unavailable | WARN, document manual review needed |

---

## Example Invocations

**Full Review:**
```
/review-devops-security full
```

**Infrastructure Only:**
```
/review-devops-security --scope infrastructure
```

**Specific Frameworks:**
```
/review-devops-security full --frameworks CIS,PCI-DSS
```

**Lower Severity Threshold:**
```
/review-devops-security full --severity-threshold medium
```
