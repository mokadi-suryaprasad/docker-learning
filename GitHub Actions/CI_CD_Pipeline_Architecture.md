
```mermaid
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

  %% Shared Artifacts/Registries
  gcs[(GCS - Build Artifacts)]
  gar[(GAR - Docker Images)]
  charts[(Helm Charts repo)]

  %% Dev Checks (on PR)
  subgraph DEV_CHECKS[Pull Request Checks]
    co[Code Checkout]
    deps[Install Dependencies]
    lint[Lint / Syntax Check]
    unit[Unit Tests]
    codeql[CodeQL (SAST)]
  end

  dev --> |push code| feature
  feature --> |open PR to main| DEV_CHECKS
  DEV_CHECKS --> |pass & review| main

  %% Development Pipeline
  subgraph DEV_PIPE[Development Pipeline (on merge to main)]
    dev_gcs[Store Artifacts -> GCS]
    dev_build[Docker Build]
    dev_trivy[Trivy Scan]
    dev_push[Push Image -> GAR]
    dev_helm[Update Helm values.yaml with COMMIT_SHA]
    dev_pr[Auto PR to ArgoCD Manifests Repo]
    dev_sync[ArgoCD Sync (GitOps) -> Dev Cluster]
  end

  main --> DEV_PIPE
  DEV_PIPE --> gcs & gar
  dev_helm --> charts
  dev_pr --> dev_sync

  %% Pre-Prod Pipeline
  feature --> |open PR to pre-prod| preprod
  subgraph PRE_PROD[Pre-Prod Pipeline (on pre-prod PR merge)]
    pp_gcs[Store Artifacts -> GCS]
    pp_build[Docker Build]
    pp_trivy[Trivy Scan]
    pp_push[Push Image -> GAR]
    pp_dast[DAST with OWASP ZAP]
    pp_helm[Update Helm values.yaml with COMMIT_SHA]
    pp_pr[Auto PR to ArgoCD Manifests Repo]
    pp_sync[ArgoCD Sync (GitOps) -> Pre-Prod Cluster]
  end

  preprod --> PRE_PROD
  PRE_PROD --> gcs & gar
  pp_helm --> charts
  pp_pr --> pp_sync
  pp_sync --> qa

  %% Manual testing window
  qa --> |functional/regression tests| preprod
  preprod -. after ~2 weeks & approvals .-> mgmt

  %% Prod Pipeline
  mgmt --> |approval & release planning| prod
  subgraph PROD[Production Pipeline (on release)]
    pr_gcs[Store Artifacts -> GCS]
    pr_build[Docker Build]
    pr_trivy[Trivy Scan]
    pr_push[Push Image -> GAR]
    pr_helm[Update Helm values.yaml with RELEASE TAG]
    pr_branch[Create Release Branch / Tag]
    pr_pr[Auto PR to ArgoCD Manifests Repo]
    pr_sync[ArgoCD Sync (GitOps) -> Prod Cluster]
  end

  prod --> PROD
  PROD --> gcs & gar
  pr_helm --> charts
  pr_branch --> pr_pr --> pr_sync
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
