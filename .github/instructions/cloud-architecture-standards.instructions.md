---
applyTo: '**/infra-spec.md, **/*.tf, **/*.bicep'
---

# Cloud Architecture Standards

## Well-Architected Pillars
- **Operational Excellence**: IaC for all resources, automated deployment/rollback
- **Security**: Defense in depth, least privilege, encryption everywhere
- **Reliability**: Multi-AZ for prod, auto-scaling, circuit breakers, DR with RTO/RPO
- **Performance**: Right-sizing from metrics, caching, read replicas
- **Cost**: Reserved instances for prod, spot for dev, auto-scaling, tagging

## Network Patterns
- Separate VNet/VPC per environment
- Private subnets for compute/data; public only for LB/NAT
- Default deny; explicit allow with justification
- WAF for public endpoints; DDoS protection for prod

## Compute Patterns
- AKS/GKE for stateless workloads with auto-scaling
- Serverless (Functions/Cloud Run) for event-driven
- Managed images with hardened OS for VMs

## Data Patterns
- Managed services preferred (Azure SQL, Cloud SQL)
- Encryption at rest (CMK for sensitive); TLS 1.2+ in transit
- Zone-redundant deployment; automated backups with tested restore

## Environment Matrix
| Aspect | dev | qa | staging | prod |
|--------|-----|----|---------| ---- |
| HA | Single AZ | Single AZ | Multi-AZ | Multi-AZ |
| Scale | Minimal | Reduced | Prod-like | Full |
| Approval | None | None | 1 | 2 |

## Azure Specifics
- Management Groups for governance; Azure Policy at MG level
- Managed Identity for service auth; Defender for Cloud enabled
- Log Analytics workspace per environment

## GCP Specifics
- Resource Hierarchy (Org → Folder → Project); Org Policies
- Workload Identity for service auth; Security Command Center
- Cloud Audit Logs export to BigQuery

## Anti-Patterns (Block)
- Public IP on databases
- Unencrypted data at rest
- Hardcoded credentials
- Single-AZ in production
- Missing backup configuration
