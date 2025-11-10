# GitHub Actions Interview Questions 

GitHub Actions is a CI/CD tool built into GitHub. It helps us **automate** things like building code, running tests, and deploying applications.

---

## 1. What is GitHub Actions?
GitHub Actions is a **CI/CD automation service** inside GitHub.  
When we push code, it can automatically:
- Build the application
- Test it
- Create Docker images
- Deploy to cloud platforms

---

## 2. What is a Workflow?
A **workflow** is a **YAML** file that contains automation steps.
It lives inside:
```
.github/workflows/
```
Example workflow:
```yaml
name: Build App
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Building App"
```

---

## 3. What is a Job?
A **job** is a group of steps that run in one virtual machine.
Example:
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

---

## 4. What is a Runner?
A **runner** is the machine where your workflow runs.
Types:
- GitHub Hosted Runner (default)
- Self-Hosted Runner (your own VM)

---

## 5. What is a Step?
A **step** is one action inside a job.
Example:
```yaml
steps:
  - run: echo "Hello"
```

---

## 6. What is an Action?
Actions are reusable code pieces.
Example commonly used action:
```yaml
uses: actions/checkout@v4
```

---

## 7. What is `on:` in workflow?
`on:` tells **when** the workflow runs.
Examples:
```yaml
on: push
on: pull_request
on:
  schedule:
    - cron: "0 0 * * *"
```

---

## 8. What are Artifacts?
Artifacts are **files produced** by the workflow, like:
- Build outputs
- Test reports

They can be uploaded and downloaded later.

---

## 9. What are Secrets?
Secrets store **sensitive data** like:
- API keys
- Cloud credentials
They are stored in:
Settings → Secrets → Actions

Example:
```yaml
env:
  API_KEY: ${{ secrets.MY_API_KEY }}
```

---

## 10. How to Trigger Workflow Manually?
Use:
```yaml
on:
  workflow_dispatch:
```

---

## 11. What is Matrix Strategy?
Used to test in multiple environments.
Example:
```yaml
strategy:
  matrix:
    node-version: [14, 16, 18]
```

---

## 12. How to Cache Dependencies?
Caching speeds up workflows.
Example:
```yaml
uses: actions/cache@v3
```

---

## 13. How to Deploy Using GitHub Actions?
You simply add deployment steps after build stage.
Example:
```yaml
- run: kubectl apply -f deployment.yaml
```

---

## 14. Difference Between GitHub Actions and Jenkins
| Feature | GitHub Actions | Jenkins |
|--------|----------------|---------|
| Hosted by | GitHub | Self/Machine |
| Setup | Very Easy | Needs Setup |
| Plugins | Many built-in actions | Many plugins |
| Cost | Free limited usage | Depends on server cost |

---

## 15. One-Line Interview Summary
> GitHub Actions helps automate CI/CD directly inside GitHub using YAML workflows, runners, jobs, steps, and reusable actions.

---

