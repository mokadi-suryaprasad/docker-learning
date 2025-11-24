# Interview Question: Deploying Specific Feature After Merge

**Question:**
You created a feature branch and merged it into the `development` branch. After your merge, someone pushed new commits directly to the `development` branch. Now you need to deploy the application with **only your feature changes**, without including the new commits. What steps will you follow to achieve this?

---

# Answer:

To deploy only the feature changes you merged into development, follow these steps:

### **1. Identify the merge commit**

Check the Git history to find the exact commit where your feature branch was merged:

```bash
git log --oneline --graph development
```

Copy the merge commit ID.

### **2. Create a new branch from the merge commit**

This new branch contains only the feature code you want to deploy:

```bash
git checkout -b deploy-feature <merge-commit-id>
```

### **3. Deploy this branch**

Run CI/CD, Docker build, or deployment pipeline using this branch.

### **4. (Optional) Reset development**

If your team agrees, reset development to the merge commit:

```bash
git checkout development
git reset --hard <merge-commit-id>
git push --force
```

---

This ensures deployment happens **only with your feature**, ignoring newer unwanted commits.
