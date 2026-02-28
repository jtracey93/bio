# Contract: GitHub Actions Deployment Workflow

**Feature**: 001-initial-bio-setup
**Date**: 2026-02-28
**Type**: CI/CD Pipeline Contract

---

## Overview

The GitHub Actions workflow (`.github/workflows/deploy.yml`) is the primary external interface of this project. It defines how code changes in the repository translate to a live deployment on GitHub Pages. This contract specifies the workflow's triggers, inputs, outputs, and expected behaviours.

---

## Trigger Contract

| Trigger | Branch | Condition |
|---------|--------|-----------|
| `push` | `main` | Every push to `main` (occurs via PR merge; direct push blocked by branch protection) |
| `workflow_dispatch` | `main` | Manual trigger for debugging/re-deployment |

**Not triggered by**: pushes to feature branches, pull request creation/update, tag creation.

---

## Permissions Contract

| Permission | Scope | Reason |
|------------|-------|--------|
| `contents: read` | Repository | Checkout source code |
| `pages: write` | GitHub Pages | Upload and deploy pages artifact |
| `id-token: write` | OIDC | Required by `actions/deploy-pages` for authentication |

---

## Jobs Contract

### Job: `build`

| Step | Action | Version | Purpose |
|------|--------|---------|---------|
| Checkout | `actions/checkout@v4` | v4 | Clone repository |
| Setup Hugo | `peaceiris/actions-hugo@v3` | v3 | Install Hugo Extended v0.157.0 |
| Build | `hugo --minify` | — | Compile site to `./public/` |
| Upload artifact | `actions/upload-pages-artifact@v3` | v3 | Package `./public/` as Pages artifact |

**Build output**: `./public/` directory containing:
- `index.html` — rendered CV page
- CSS (compiled from SCSS by Hugo Extended)
- `img/avatar.jpg` — profile photo
- `CNAME` — custom domain file (from `static/CNAME`)

**Failure behaviour**: If `hugo --minify` exits non-zero (e.g. YAML syntax error, missing template), the workflow fails. The deploy job does not run. The previously deployed version remains live (FR-009).

### Job: `deploy`

| Step | Action | Version | Purpose |
|------|--------|---------|---------|
| Deploy | `actions/deploy-pages@v4` | v4 | Deploy uploaded artifact to GitHub Pages |

**Dependencies**: Requires `build` job to succeed.
**Environment**: `github-pages` (GitHub-managed deployment environment).
**Output**: `page_url` — the URL where the site is deployed.

---

## Concurrency Contract

```yaml
concurrency:
  group: pages
  cancel-in-progress: false
```

- Only one deployment runs at a time (serialised by group `pages`).
- In-progress deployments are NOT cancelled — they run to completion.
- Queued deployments wait for the current one to finish.

---

## Success Criteria

| Criteria | Validation |
|----------|------------|
| Hugo build succeeds | Exit code 0 from `hugo --minify` |
| Artifact uploaded | `actions/upload-pages-artifact` succeeds |
| Deployment completes | `actions/deploy-pages` returns `page_url` |
| Site accessible | Custom domain serves updated content within 5 minutes of merge (SC-003) |

---

## Error Scenarios

| Scenario | Expected Behaviour |
|----------|--------------------|
| YAML syntax error in `data/content.yaml` | Hugo exits non-zero; build job fails; deploy skipped; last-good version stays live |
| Missing theme files | Hugo exits non-zero; build job fails; deploy skipped |
| GitHub Pages not configured | `actions/deploy-pages` fails with permissions error |
| DNS not configured | Site accessible at default GitHub Pages URL; custom domain returns DNS error |
