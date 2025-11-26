# DevSecOps Overview

## ⭐ What is DevSecOps?
DevSecOps means **Development + Security + Operations**.  
It integrates security into every stage of the DevOps pipeline — from planning to coding, building, testing, deploying, and monitoring.

DevSecOps ensures:
- Security is **early**
- Security is **automated**
- Security is **continuous**
- Security is **everyone’s responsibility**

---

# 🎯 Goals of DevSecOps
- Detect vulnerabilities early  
- Automate security checks  
- Integrate security into CI/CD  
- Reduce risk of attacks  
- Improve collaboration between Dev, Sec, and Ops  
- Deliver secure applications faster  

---

# 🔄 DevSecOps Workflow 
```
Plan → Code → Build → Test → Secure → Deploy → Monitor → Feedback
```

```
PLAN → Jira, Confluence, Miro
CODE → Git, GitHub, CodeQL, SonarQube
BUILD → Docker, Maven, Trivy, Snyk
TEST → GitHub Actions, JUnit, pytest, CodeQL
SECURE → Trivy, OPA, Checkov, OWASP ZAP
DEPLOY → ArgoCD, Kubernetes, Terraform
MONITOR → Prometheus, Grafana, ELK, CloudWatch
FEEDBACK → Jira, Slack, GitHub Issues
```

### 1️⃣ Plan
- Define security requirements  
- Threat modeling  
- Risk assessment  

### 2️⃣ Code
- Secure coding practices  
- SAST (Static code scanning) using tools like CodeQL  
- Remove hardcoded secrets  

### 3️⃣ Build
- Dependency scanning  
- Container image scanning (Trivy)  
- Check packages & libraries  

### 4️⃣ Test
- Automated tests  
- SAST + secret scanning  
- Linting & quality checks  

### 5️⃣ Secure
- IaC scanning (Terraform, Kubernetes YAML)  
- Policy checks  
- API security reviews  

### 6️⃣ Deploy
- GitOps deployment with ArgoCD  
- Signed and verified images  
- RBAC + network security  

### 7️⃣ Monitor
- DAST scanning (OWASP ZAP)  
- Logs, metrics, alerts  
- Runtime threat detection  

### 8️⃣ Feedback
- Developers get security reports  
- Fix issues quickly  
- Improve pipeline continuously  

---

# 🛠 DevSecOps Tools (Simple Table)

| Stage | Purpose | Tools |
|-------|---------|-------|
| **SAST** | Code scanning | CodeQL, SonarQube |
| **Dependency Scanning** | Library vulnerabilities | Trivy, Snyk |
| **Container Scanning** | Scan Docker images | Trivy, Clair |
| **IaC Scanning** | Check Terraform/K8s YAML | Trivy, Checkov |
| **Secret Scanning** | Find leaked secrets | GitHub Secret Scan, TruffleHog |
| **DAST** | Scan running app | OWASP ZAP |
| **CI/CD** | Automation workflows | GitHub Actions |
| **GitOps Deploy** | Kubernetes deployment | ArgoCD |
| **Monitoring** | Logs, metrics | Prometheus, Grafana |

---

# 🔐 Why DevSecOps is Needed
Before DevSecOps:
- Security was checked at the end  
- Many vulnerabilities reached production  
- Fixing issues was slow and costly  
- Dev + Ops + Security worked separately  

With DevSecOps:
- Security starts early (shift-left)
- Continuous scanning  
- Faster detection and remediation  
- Automated and consistent security  
- Strong collaboration between teams  

---

# ⚡ Core Principles of DevSecOps

### ✔ 1. Shift Left Security  
Security checks as early as possible.

### ✔ 2. Automation  
Scan everything automatically:
- Code  
- Containers  
- Infrastructure  
- Dependencies  

### ✔ 3. Continuous Security  
Every commit → scanned  
Every build → scanned  

### ✔ 4. Security as Code  
Policies and rules are written as code.

### ✔ 5. Collaboration  
Dev + Sec + Ops work together.

---

# 🧩 DevOps vs DevSecOps

| DevOps | DevSecOps |
|--------|------------|
| Speed and automation | Speed **+ Security** |
| Security at the end | Security throughout |
| Dev + Ops | Dev + Sec + Ops |
| CI/CD | CI/CD + security scanning |

---

# 🚀 Example DevSecOps Pipeline

1. **Developer pushes code to GitHub**  
2. **CodeQL scans source code**  
3. **Trivy scans Docker image + IaC**  
4. **GitHub Actions builds & tests app**  
5. **ArgoCD deploys app to Kubernetes**  
6. **OWASP ZAP scans the running app**  
7. **Security alerts sent to developers**  

---

# 🔥 Summary
DevSecOps integrates security into every stage of DevOps so applications are:
- Faster to deploy  
- More secure  
- More reliable  
- Automatically scanned  
- Easier to maintain  

---

# ⭐ One-Line Answer (For Interview)
**“DevSecOps adds automated security checks like SAST, DAST, IaC scanning, and container scanning into the DevOps pipeline, ensuring secure and fast delivery from code to production.”**
