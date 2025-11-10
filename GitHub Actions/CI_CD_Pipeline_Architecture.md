``` mermaid
flowchart TB
  %% Roles
  dev[Developer]
  mgmt[Management Approvers]
  qa[QA Team]

  %% Git Repos
  subgraph GH[GitHub]
    feature[(Feature Branch)]
    main[(main)]
    preprod[(pre-prod branch)]
    prod[(release branch)]
  end

  %% Shared Stores
  gcs[(GCS - Artifacts)]
  gar[(GAR - Docker Images)]
  charts[(Helm Charts Repo)]

  %% Pull Request Checks
  subgraph DEV_CHECKS[Pull Request Checks]
    co[Code Checkout]
    deps[Install Dependencies]
    lint[Lint / Syntax Check]
    unit[Unit Tests]
    codeql[CodeQL Scan (SAST)]
  end

  dev --> |push code| feature
  feature --> |PR to main| DEV_CHECKS
  DEV_CHECKS --> |pass + review| main

  %% Development Pipeline
  subgraph DEV_PIPE[Development Pipeline (merge to main)]
    dev_gcs[Store Artifacts → GCS]
    dev_build[Docker Build]
    dev_trivy[Trivy Scan]
    dev_push[Push Image → GAR]
    dev_helm[Update Helm values.yaml (COMMIT_SHA)]
    dev_pr[PR → ArgoCD Manifests Repo]
    dev_sync[ArgoCD Sync → Dev Cluster]
  end

  main --> dev_gcs
  dev_gcs --> dev_build --> dev_trivy --> dev_push --> dev_helm --> dev_pr --> dev_sync
  dev_gcs --> gcs
  dev_push --> gar
  dev_helm --> charts

  %% Pre-Prod Pipeline
  feature --> |PR to pre-prod| preprod

  subgraph PRE_PROD[Pre-Prod Pipeline (merge to pre-prod)]
    pp_gcs[Store Artifacts → GCS]
    pp_build[Docker Build]
    pp_trivy[Trivy Scan]
    pp_push[Push Image → GAR]
    pp_dast[DAST Security Scan (OWASP ZAP)]
    pp_helm[Update Helm values.yaml (COMMIT_SHA)]
    pp_pr[PR → ArgoCD Manifests Repo]
    pp_sync[ArgoCD Sync → Pre-Prod Cluster]
  end

  preprod --> pp_gcs
  pp_gcs --> pp_build --> pp_trivy --> pp_push --> pp_dast --> pp_helm --> pp_pr --> pp_sync
  pp_gcs --> gcs
  pp_push --> gar
  pp_helm --> charts

  pp_sync --> qa
  qa --> |Functional & Regression Testing| preprod

  preprod -. Wait ~2 weeks + Approval .-> mgmt

  %% Production Pipeline
  mgmt --> |Approval| prod

  subgraph PROD[Production Pipeline (Release)]
    pr_gcs[Store Artifacts → GCS]
    pr_build[Docker Build]
    pr_trivy[Trivy Scan]
    pr_push[Push Image → GAR]
    pr_helm[Update Helm values.yaml (RELEASE TAG)]
    pr_branch[Create Release Tag + Branch]
    pr_pr[PR → ArgoCD Manifests Repo]
    pr_sync[ArgoCD Sync → Prod Cluster]
  end

  prod --> pr_gcs
  pr_gcs --> pr_build --> pr_trivy --> pr_push --> pr_helm --> pr_branch --> pr_pr --> pr_sync
  pr_gcs --> gcs
  pr_push --> gar
  pr_helm --> charts

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
