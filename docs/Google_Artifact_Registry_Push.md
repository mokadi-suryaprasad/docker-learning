
# Google Artifact Registry (Private Registry) — Push Images Step‑by‑Step

This guide walks you through **creating a private Docker repository** in **Google Artifact Registry (GAR)** and **pushing/pulling images**. It also includes a **GitHub Actions** pipeline and examples for **GKE** and **Cloud Run**.

> Tailored to your setup:  
> **Project:** "silken-oxygen-454215-v6" · **Region:** "asia-south1" · **Repository:** "microservices-demo" · **Example Image:** "spring-app:v1.0.0"

---

## 1) Why Google Artifact Registry?
- **Private & secure**: IAM‑based access (`reader`, `writer`, `admin`)
- **Regional endpoints** (lower latency, data locality)
- **Native integrations**: GKE, Cloud Run, Cloud Build, GitHub Actions
- **Lifecycle policies** for auto‑cleanup of old tags

---

## 2) Prerequisites
- Google Cloud CLI installed (`gcloud`)
- Docker installed and running
- Permissions to create/use Artifact Registry (`Artifact Registry Admin/Writer/Reader` as needed)

Enable APIs and set defaults:
```bash
gcloud services enable artifactregistry.googleapis.com containerregistry.googleapis.com

gcloud config set project silken-oxygen-454215-v6
gcloud config set artifacts/location asia-south1
```

---

## 3) Create a Private Repository
```bash
gcloud artifacts repositories create microservices-demo   --repository-format=docker   --location=asia-south1   --description="Private Docker repo for microservices"
```

Verify:
```bash
gcloud artifacts repositories list --location=asia-south1
```

---

## 4) IAM — Who Can Push/Pull?
Grant **writers** (push) and **readers** (pull). Example:
```bash
# Push permission to a developer
gcloud artifacts repositories add-iam-policy-binding microservices-demo   --location=asia-south1   --member="user:alice@example.com"   --role="roles/artifactregistry.writer"

# Pull permission to a runtime service account
gcloud artifacts repositories add-iam-policy-binding microservices-demo   --location=asia-south1   --member="serviceAccount:runtime-sa@silken-oxygen-454215-v6.iam.gserviceaccount.com"   --role="roles/artifactregistry.reader"
```

Common roles:
- `roles/artifactregistry.reader` — pull images
- `roles/artifactregistry.writer` — push images
- `roles/artifactregistry.admin` — manage repo settings

---

## 5) Authenticate Docker Locally
Use `gcloud` as Docker credential helper:
```bash
gcloud auth login                              # (or 'gcloud auth application-default login')
gcloud auth configure-docker asia-south1-docker.pkg.dev
```

---

## 6) Build, Tag, and Push
Assuming your project has a `Dockerfile`:
```bash
# Build
docker build -t spring-app:v1.0.0 .

# Tag to GAR path
docker tag spring-app:v1.0.0 asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0

# Push
docker push asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0
```

Now your image appears at: "asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo"

---

## 7) Pull the Image
```bash
docker pull asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0
```

---

## 8) Use from Kubernetes (GKE)
### Option A — Workload Identity (Recommended)
- Bind a Kubernetes ServiceAccount (KSA) to a Google Service Account (GSA) that has `roles/artifactregistry.reader`.
- No Docker secret needed.

### Option B — Image Pull Secret (if you must use JSON key)
```bash
kubectl create secret docker-registry gar-pull-secret   --docker-server=asia-south1-docker.pkg.dev   --docker-username=_json_key   --docker-password="$(cat key.json)"   --docker-email=not-required@example.com
```

Deployment example:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  replicas: 1
  selector:
    matchLabels: {{ app: spring-app }}
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      # If using imagePullSecret
      # imagePullSecrets:
      #   - name: gar-pull-secret
      containers:
        - name: app
          image: asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0
          ports:
            - containerPort: 8080
```

---

## 9) Deploy to Cloud Run
```bash
gcloud run deploy spring-app   --image asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0   --region asia-south1   --platform managed   --allow-unauthenticated
```

> Omit `--allow-unauthenticated` for private services.

---

## 10) GitHub Actions — Build & Push to GAR
Create `.github/workflows/push-to-gar.yml`:
```yaml
name: Build and Push to GAR

on:
  push:
    branches: [ "main" ]
    paths:
      - "src/**"
      - "pom.xml"
      - "Dockerfile"
  workflow_dispatch:

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      id-token: write  # Required for Workload Identity Federation

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Auth to Google Cloud (Workload Identity Federation)
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ secrets.GCP_WORKLOAD_IDENTITY_PROVIDER }}
          service_account: ${{ secrets.GCP_SA_EMAIL }}

      - name: Set up gcloud
        uses: google-github-actions/setup-gcloud@v2

      - name: Configure Docker for GAR
        run: gcloud auth configure-docker asia-south1-docker.pkg.dev --quiet

      - name: Set Image Vars
        id: vars
        run: |
          echo "IMAGE_REPO=asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app" >> $GITHUB_OUTPUT
          echo "IMAGE_TAG=${{ github.sha }}" >> $GITHUB_OUTPUT

      - name: Build
        run: docker build -t ${{ steps.vars.outputs.IMAGE_REPO }}:${{ steps.vars.outputs.IMAGE_TAG }} .

      - name: Push
        run: docker push ${{ steps.vars.outputs.IMAGE_REPO }}:${{ steps.vars.outputs.IMAGE_TAG }}
```

**Setup required:** In GitHub → Repo → Settings → *Secrets and variables* → *Actions*, add:
- `GCP_WORKLOAD_IDENTITY_PROVIDER`
- `GCP_SA_EMAIL`

> Prefer Workload Identity Federation instead of storing long‑lived JSON keys.

---

## 11) Maven/Gradle + JIB (Optional: Build without Dockerfile)
You can skip Dockerfile and use **JIB** to build/push directly:
```bash
# Maven
mvn -P jib -Djib.to.image=asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0 com.google.cloud.tools:jib-maven-plugin:build

# Gradle
./gradlew jib --image=asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0
```

---

## 12) Lifecycle Policies (Auto‑Delete Old Tags)
Example: keep last 15 tags, delete older:
```bash
gcloud artifacts docker images update asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo   --update-mode=KEEP_LAST_N   --keep-count=15   --location=asia-south1
```

---

## 13) Naming & Tagging Tips
- Use **immutable tags** for deploys (e.g., Git SHA): `:${{ github.sha }}`
- Use **semantic tags** for human readers: `:v1.2.3`
- Keep repo names short and per‑team/service

---

## 14) Troubleshooting
- **403 permission denied** → Add `roles/artifactregistry.writer` (push) or `reader` (pull) to correct principal.
- **denied: Permission "artifactregistry.repositories.downloadArtifacts"** → Add `roles/artifactregistry.reader`.
- **Repository not found** → Confirm region/project/repo. Run:
  ```bash
  gcloud artifacts repositories list --location=asia-south1
  ```
- **Auth errors** → Re‑run:
  ```bash
  gcloud auth configure-docker asia-south1-docker.pkg.dev
  ```

---

## 15) Quick Demo — End‑to‑End
```bash
# 0) Configure
gcloud config set project silken-oxygen-454215-v6
gcloud auth configure-docker asia-south1-docker.pkg.dev

# 1) Build
docker build -t spring-app:v1.0.0 .

# 2) Tag for GAR
docker tag spring-app:v1.0.0 asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0

# 3) Push
docker push asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0

# 4) Pull (anywhere with read access)
docker pull asia-south1-docker.pkg.dev/silken-oxygen-454215-v6/microservices-demo/spring-app:v1.0.0
```

**Done.** Your private image is now in Google Artifact Registry and ready for GKE/Cloud Run/VMs.
