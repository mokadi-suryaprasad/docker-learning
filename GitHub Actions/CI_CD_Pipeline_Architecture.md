## CI/CD Pipeline Architecture (Dev → Pre-Prod → Prod)

```mermaid
flowchart TB
  dev[Developer] -->|Push code| feature((feature branch))
  feature -->|PR to main| pr_checks[PR Checks\nCheckout • Dependencies • Lint • Unit Tests • CodeQL]
  pr_checks -->|Approved & Merged| main[(main branch)]

  %% Development Environment Deployment
  main --> dev_gcs[Store Artifacts → GCS]
  dev_gcs --> dev_build[Docker Build]
  dev_build --> dev_trivy[Trivy Scan]
  dev_trivy --> dev_push[Push Image → GAR]
  dev_push --> dev_helm[Update Helm values.yaml with COMMIT SHA]
  dev_helm --> dev_sync[ArgoCD Sync → Deploy to DEV environment]

  %% Pre-Prod Deployment Stage
  feature -->|PR to pre-prod| preprod[(pre-prod branch)]
  preprod --> pp_gcs[Store Artifacts → GCS]
  pp_gcs --> pp_build[Docker Build]
  pp_build --> pp_trivy[Trivy Scan]
  pp_trivy --> pp_push[Push Image → GAR]
  pp_push --> pp_zap[DAST Scan → OWASP ZAP]
  pp_zap --> pp_helm[Update Helm values.yaml with COMMIT SHA]
  pp_helm --> pp_sync[ArgoCD Sync → Deploy to PRE-PROD environment]

  %% QA Testing
  pp_sync --> qa[QA Team Testing\nFunctional + Regression]
  qa -->|After ~2 Weeks Sign-off| mgmt[Management Approval]

  %% Production Deployment
  mgmt -->|Approve Release| prod[(release tag/branch)]
  prod --> pr_gcs[Store Artifacts → GCS]
  pr_gcs --> pr_build[Docker Build]
  pr_build --> pr_trivy[Trivy Scan]
  pr_trivy --> pr_push[Push Image → GAR]
  pr_push --> pr_helm[Update Helm values.yaml with RELEASE TAG]
  pr_helm --> pr_sync[ArgoCD Sync → Deploy to PRODUCTION environment]
```
# CI/CD Pipeline Architecture — Dev → Pre‑Prod → Prod (GitOps with ArgoCD)

This diagram shows a **GitHub‑first CI/CD** with **GAR** (images), **GCS** (artifacts), **Trivy** (image scan),
**CodeQL** (SAST), **OWASP ZAP** (DAST), **Helm** (values.yaml), and **ArgoCD** for **GitOps** sync into clusters.

## High‑Level Flow
1) **Developer** pushes code to a feature branch and opens a **PR to `main`**.  
   PR checks run: **checkout → dependencies → lint → unit tests → CodeQL**.  
   If everything passes and reviewers approve → **merge to `main`**.

2) **Development Pipeline** (on merge to `main`):
   - Store build **artifacts to GCS**  
   - **Docker build** → **Trivy scan**  
   - **Push image to GAR**  
   - **Update Helm `values.yaml` with COMMIT_SHA** (immutable deploy)  
   - Create PR to **ArgoCD manifests repo** → **ArgoCD sync** to **Dev cluster**

3) **Pre‑Prod**:
   - Raise **PR to pre‑prod branch** → merge triggers pre‑prod pipeline  
   - Store artifacts to GCS, Docker build, Trivy scan, **push to GAR**  
   - **DAST (OWASP ZAP)** against pre‑prod endpoint  
   - Update Helm values with **COMMIT_SHA**, PR to manifests → **ArgoCD sync** to **Pre‑Prod**  
   - **QA tests** run on the application

4) **After ~2 weeks** and **management approval**, raise **PR for production release**.

5) **Production Pipeline** (on release tag/branch):
   - Store artifacts to GCS, Docker build, Trivy scan, **push to GAR**  
   - Update Helm values with **RELEASE TAG** (traceable)  
   - **Create release branch/tag**  
   - PR to manifests → **ArgoCD sync** to **Prod**

## Notes & Tips
- Prefer **immutable image tags** (commit SHA) for Dev/Pre‑Prod and **annotated release tags** for Prod.
- Security gates: **CodeQL** (SAST) early, **Trivy** on images, **ZAP (DAST)** in Pre‑Prod.
- Everything deploys by **GitOps**: app version is driven by **Helm chart changes** that ArgoCD watches.
- Store test reports and build logs in **GCS**; images in **GAR**.
