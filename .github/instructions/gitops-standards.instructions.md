---
applyTo: '**/k8s/**/*.yaml, **/k8s/**/*.yml, **/kubernetes/**/*.yaml, **/kubernetes/**/*.yml, **/helm/**/*.yaml, **/helm/**/*.yml, **/charts/**/*.yaml, **/charts/**/*.yml, **/manifests/**/*.yaml, **/manifests/**/*.yml, **/deploy/**/*.yaml, **/deploy/**/*.yml, **/argocd/**/*.yaml, **/argocd/**/*.yml, **/flux/**/*.yaml, **/flux/**/*.yml, **/kustomization.yaml, **/Chart.yaml, **/values*.yaml'
---

# GitOps Standards

## Core Principles
- **Declarative**: Desired state in Git, system converges automatically
- **Versioned**: All changes tracked with audit trail
- **Automated**: Reconciliation via operators (ArgoCD/Flux); no manual apply
- **Reviewed**: All changes via PR with approvals

## Repository Strategy
- Application code separate from infrastructure code
- Shared modules in dedicated repository
- Environment configs via overlays (Kustomize) or values files (Helm)

## Branching Model
```
main (prod) ← staging ← develop ← feature/*
                     ↖ hotfix/*
```

| Branch | Protection | Approvers |
|--------|------------|-----------|
| main | CI pass, reviews | 2 |
| staging | CI pass, reviews | 1 |
| develop | CI pass | 0 |

## Environment Promotion
```
feature → develop → staging → main
   ↓         ↓         ↓        ↓
  dev       qa     staging    prod
```
- Never skip environments
- Tag releases with semantic version

## Secrets Management
- Use External Secrets Operator or equivalent
- Never commit secrets (even encrypted)
- Secrets referenced, not stored

## Rollback Procedure
1. Revert commit in Git
2. Operator detects change, reconciles
3. Verify via health checks
4. Notify stakeholders

## Immutable Tags
- Production: specific version tags (`v1.2.3`)
- Never use `:latest` in production

## Anti-Patterns (Block)
- Manual `kubectl apply` / `terraform apply` in prod
- Environment-specific code in app repo
- Mutable tags in production
- Secrets in Git
- Skipping staging
