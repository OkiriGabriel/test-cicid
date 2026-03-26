# 🚀 Production Deployment Workflow

Automates the full 9-step production deployment process via GitHub Actions.

---

## Prerequisites

Before using this workflow, complete these one-time setup steps:

1. **Add the workflow file** to your repo at `.github/workflows/prod-deploy.yml`
2. **Enable write permissions** → Settings → Actions → General → *Read and write permissions* + *Allow GitHub Actions to create and approve pull requests*
3. **Protect the `main` branch** → Settings → Branches → Require PR + approvals before merging
4. **Create a `production` environment** → Settings → Environments → Add required reviewers (this is the approval gate for Step 7)
5. **Enable auto-merge** → Settings → General → Pull Requests → *Allow auto-merge*

---

## ▶How to Run

1. Go to the **Actions** tab in your repository
2. Select **Production Deployment** from the left sidebar
3. Click **Run workflow** and fill in the inputs:

| Input | Required | Example | Description |
|---|---|---|---|
| `change_details` | ✅ | `PSO` | Appended to the branch name |
| `version_tag` | ✅ | `vPROD-5.0` | Git tag applied after merge |
| `ritm_ticket` | ❌ | `RITM2106103` | Added to PR title if provided |

4. Click the green **Run workflow** button

---

## 🔄 What It Does (All 9 Steps)

```
main ──────────────────────────────────────────────────► main (tagged)
         │                                          ▲
         │  checkout                                │ merge + tag
         ▼                                          │
  PROD-Deploy_Jul-11-2025_PSO ◄── merge UAT ────────┘
```

| Step | Action | How |
|---|---|---|
| 1 | Checkout `main` & pull latest | `actions/checkout@v4` |
| 2 | Create `PROD-Deploy_<DATE>_<change_details>` branch | Auto-generated from inputs |
| 3 | Merge UAT → PROD branch (`-X theirs`) | Prioritises UAT changes |
| 4 | Push PROD branch to remote | `git push origin <branch>` |
| 5 | Create Pull Request into `main` | `peter-evans/create-pull-request@v6` |
| 6 | Confirm PR targets `main` | Enforced by workflow (hardcoded) |
| 7 | **⏸ Pause for review & approval** | GitHub Environment approval gate |
| 8 | Merge PR to `main` | Auto-merge after approval |
| 9 | Tag the build on `main` | `git tag -a <version_tag>` |

> **Step 7 is the only manual step.** Reviewers will receive a GitHub notification and must approve under **Settings → Environments → production** before the workflow continues.

---

## Branch Naming Convention

```
PROD-Deploy_<MMM-DD-YYYY>_<change_details>

Example:  PROD-Deploy_Jul-11-2025_PSO
```

---

## Tagging Convention

After merge, the build is tagged on `main`:

```bash
git tag -a vPROD-5.0 -m "PROD-Deploy-Jul-11-2025_PSO"
git push origin tag vPROD-5.0
```

---

## Troubleshooting

**Merge fails at Step 3**
UAT has unresolvable conflicts. Resolve them in UAT, push, then re-run.

**Auto-merge not working**
Enable it under Settings → General → Pull Requests → *Allow auto-merge*.

**No permission to push branches**
Set workflow permissions to *Read and write* under Settings → Actions → General.

**Approval notification not received**
Check Settings → Environments → production for pending approvals manually.

---

## File Location

```
.github/
└── workflows/
    └── prod-deploy.yml
```
