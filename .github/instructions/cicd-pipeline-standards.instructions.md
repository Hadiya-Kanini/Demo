---
applyTo: '**/.github/workflows/*.yml, **/azure-pipelines*.yml, **/.gitlab-ci.yml'
---

# CI/CD Pipeline Standards

## Pipeline Structure
- Atomic stages with single responsibility
- Fail fast: security gates early in pipeline
- Stage order: Build → Lint → Test → Scan → Deploy

## Mandatory Security Gates
| Gate | Tools | Threshold | Blocking |
|------|-------|-----------|----------|
| SAST | CodeQL, SonarQube | 0 Critical/High | Yes |
| SCA | Snyk, OWASP Dep-Check | 0 Critical/High | Yes |
| Container | Trivy, Grype | 0 Critical | Yes |
| Secrets | GitLeaks | 0 findings | Yes |
| IaC | tfsec, Checkov | 0 Critical | Yes |

## Secret Management
- Never inline secrets; use `${{ secrets.NAME }}`
- Platform secret stores only (GitHub Secrets, Key Vault)
- Mask secrets in logs; rotate regularly

## Deployment Practices
- Build once, deploy many (immutable artifacts)
- Environment promotion: dev → qa → staging → prod
- Manual approval: staging (1), prod (2)
- Automated rollback on smoke test failure

## Rollback Triggers
- Smoke test failure
- Error rate > 5%
- Health check failures > 3 consecutive

## Notifications
- Build failure → Team + commit author
- Security gate failure → Security team (Critical)
- Deployment complete → Operations
- Rollback → On-call + management

## Anti-Patterns (Block)
- `continue-on-error: true` on security stages
- Skip tests for "urgent" deployments
- Direct deploy to prod without staging
- Shared credentials across environments
