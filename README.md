# GitHub Actions Environment Deployment Demo

> **Prepared for:** NISC — Jason Brinkmann, Teddy Reinert
> **Topic:** Actions in practice — deploying to specific environments

---

## What This Demo Shows

| Feature | Where to Find It |
|---------|-----------------|
| Multi-environment pipeline (dev → staging → prod) | `deploy.yml` |
| Reusable workflows (DRY deployments) | `reusable-deploy.yml` |
| Environment protection rules & approvals | GitHub Settings → Environments |
| Deployment concurrency (no parallel deploys) | `concurrency` block |
| Manual rollback with audit trail | `rollback.yml` |
| OIDC authentication (no stored secrets) | Deploy step in reusable workflow |
| `workflow_dispatch` for manual control | All workflows |
| Environment-specific URLs & config | `environment.url` output |

---

## Demo Flow (Recommended)

### 1. Show the Pipeline Structure (2 min)

Open `deploy.yml` and walk through:
- **Build once, deploy many** — artifact is built once, reused across environments
- **Progressive promotion** — dev must pass before staging, staging before prod
- **Conditional logic** — PRs only build/test, pushes trigger the full pipeline
- **Manual override** — `workflow_dispatch` lets you target any environment directly

### 2. Environments & Protection Rules (3 min)

Go to **Settings → Environments** and show:
- **`dev`** — No protection (auto-deploys on push)
- **`staging`** — Wait timer (5 min) for soak testing
- **`production`** — Required reviewers + branch protection (only `main`)

> 💡 *"This means a developer can push to main and it flows to dev automatically, but production requires a human approval — configurable per team policy."*

### 3. Reusable Workflows (2 min)

Open `reusable-deploy.yml` and highlight:
- Single source of truth for deployment logic
- Called with `uses: ./.github/workflows/reusable-deploy.yml`
- Each environment passes its own inputs (region, version)
- **Concurrency groups** prevent parallel deploys to same environment

> 💡 *"If your team has 10 services, they can all call this same reusable workflow. Update it once, every service gets the improvement."*

### 4. Run It Live (3 min)

1. Click **Actions → Deploy to Environments → Run workflow**
2. Select `dev` environment
3. Watch the pipeline execute in real-time
4. Show the environment deployment history and URL output

### 5. Rollback Scenario (2 min)

Show `rollback.yml`:
- Manual trigger with required inputs (version, reason)
- Still goes through environment protection rules
- Creates an audit trail (who, when, why)

> 💡 *"This gives you a one-click rollback with full traceability — no SSH needed, no runbooks to follow."*

---

## Key Talking Points for NISC

### Why Environments vs. Just Branches?

| Branch-per-env (old way) | GitHub Environments (this way) |
|--------------------------|-------------------------------|
| Merge conflicts between branches | Single `main` branch, promote artifacts |
| Drift between environments | Same artifact in every env |
| Manual gates via human process | Automated protection rules |
| No audit trail | Full deployment history in GitHub |

### How Others Use This

- **Feature flags + environments** — deploy to dev with feature enabled, staging without
- **Blue/green with Actions** — deploy to inactive slot, swap traffic via workflow
- **Database migrations** — separate job with environment-specific connection strings
- **Multi-region** — same reusable workflow, different `region` input

### Security Highlights

- **OIDC (OpenID Connect)** — no long-lived cloud credentials stored in GitHub
- **Environment secrets** — prod database password only accessible in prod jobs
- **Required reviewers** — enforce separation of duties
- **Deployment branch rules** — only `main` (or release branches) can deploy to prod

---

## Setup Instructions (if NISC wants to try it)

1. Create a repo (or fork this demo)
2. Go to **Settings → Environments** and create:
   - `dev` (no protection)
   - `staging` (add a 5-min wait timer)
   - `production` (add required reviewers — pick 1-2 team members)
3. Copy the `.github/workflows/` files into the repo
4. Push to `main` → watch the pipeline flow

---

## Files in This Demo

```
demo/
├── app/
│   └── index.html              # Simple app showing environment info
├── .github/workflows/
│   ├── deploy.yml              # Main pipeline: build → test → deploy
│   ├── reusable-deploy.yml     # Shared deployment logic (called by deploy.yml)
│   └── rollback.yml            # Manual rollback with audit trail
└── README.md                   # This file (demo script)
```
