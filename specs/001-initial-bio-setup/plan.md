# Implementation Plan: Initial Bio Website Setup

**Branch**: `001-initial-bio-setup` | **Date**: 2026-02-28 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `/specs/001-initial-bio-setup/spec.md`

## Summary

Set up a Hugo static site using the almeida-cv theme (commit `7bf8228`) to render Jack Tracey's professional CV/resume at `bio.jacktracey.co.uk`. All content is driven from a single YAML data file (`data/content.yaml`). The theme is copied directly into the repository's `themes/almeida-cv/` directory (no Git submodules). A GitHub Actions workflow deploys the built site to GitHub Pages via the artifact-based `actions/deploy-pages` action on every merge to `main`, with a `workflow_dispatch` trigger for out-of-band re-deployments during troubleshooting. The site is accessible on both the custom domain and the default GitHub Pages URL (which redirects to the custom domain).

## Technical Context

**Language/Version**: Hugo Extended v0.157.0 (latest stable) + Go templates + YAML data files
**Primary Dependencies**: almeida-cv theme (commit `7bf8228` from `ineesalmeida/almeida-cv`), GitHub Actions (`actions/checkout`, `peaceiris/actions-hugo`, `actions/upload-pages-artifact`, `actions/deploy-pages`)
**Storage**: N/A — static files only; content in `data/content.yaml`, configuration in `config.toml`
**Testing**: `hugo build` validation (zero-error exit code); local preview via `hugo server`; CI build gate
**Target Platform**: GitHub Pages (static web hosting); browsers: Chrome, Firefox, Safari, Edge (latest)
**Project Type**: static-site
**Performance Goals**: Page load < 3 seconds on standard connection (single HTML page + CSS, no JS frameworks)
**Constraints**: Single-page CV layout; Hugo Extended required for SCSS compilation; no JavaScript frameworks; theme print stylesheet used for PDF export
**Scale/Scope**: Single user (Jack Tracey); single page; ~1 YAML data file; 1 GitHub Actions workflow

## Constitution Check (Pre-Design)

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| # | Principle | Status | Evidence |
|---|-----------|--------|----------|
| I | Theme-First, Low Maintenance | PASS | Using almeida-cv theme as-is; no custom CSS/JS overrides planned. Theme copied to `themes/` per constitution (no submodules). |
| II | Content as Data | PASS | All CV content in `data/content.yaml`; no content hard-coded in templates. |
| III | Mobile-First Responsive Design | PASS | almeida-cv theme has built-in responsive CSS; no custom layout changes needed. |
| IV | PDF Export/Print Parity | PASS | Browser print-to-PDF via the theme's built-in `@media print` stylesheet is the primary export mechanism (constitution v1.1.0). CI-based PDF tooling is NOT required. Same `content.yaml` source for both web and print. |
| V | Automated Deployment (NON-NEGOTIABLE) | PASS | GitHub Actions workflow builds Hugo site and deploys via `actions/deploy-pages` on merge to `main`. `workflow_dispatch` trigger for out-of-band re-deployment. No manual deployment steps. |
| VI | Simplicity & YAGNI | PASS | Minimal configuration: Hugo + theme + YAML data + one workflow file. No JavaScript frameworks, no extra build tools. |

**Gate result**: ALL PASS — proceed to Phase 0.

## Project Structure

### Documentation (this feature)

```text
specs/001-initial-bio-setup/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output (GitHub Actions workflow contract)
└── tasks.md             # Phase 2 output (created by /speckit.tasks)
```

### Source Code (repository root)

```text
bio/                              # Repository root
├── config.toml                   # Hugo configuration (baseURL, theme, params)
├── data/
│   └── content.yaml              # All CV content (single source of truth)
├── static/
│   ├── CNAME                     # Custom domain: bio.jacktracey.co.uk
│   └── img/
│       └── avatar.jpg            # Profile photo (placeholder)
├── themes/
│   └── almeida-cv/               # Theme copied from commit 7bf8228
│       ├── assets/scss/          # Theme SCSS (compiled by Hugo Extended)
│       ├── layouts/              # Theme templates (index.html, partials/)
│       ├── i18n/                 # Internationalisation strings
│       ├── static/img/           # Theme static assets
│       └── theme.toml            # Theme metadata
├── .github/
│   └── workflows/
│       └── deploy.yml            # GitHub Actions: build + deploy to Pages
├── .gitignore                    # Ignore public/, resources/, hugo cache
└── README.md                     # Project overview and local dev instructions
```

**Structure Decision**: Standard Hugo site layout. The theme is vendored in `themes/almeida-cv/` per constitution principle I (no submodules). All content lives in `data/content.yaml` per principle II. The `static/CNAME` file ensures the custom domain persists across deployments. No additional build tools or JavaScript — principle VI.

## Complexity Tracking

No constitution violations. No complexity justifications needed.

## Manual Steps (Timing & Sequencing)

The following manual steps from the spec MUST be completed by the repository owner. **Timing matters**:

### Before First Deployment (BLOCKING)

1. **Configure GitHub Pages Source**: Repository Settings → Pages → Source → select "GitHub Actions". Without this, the `actions/deploy-pages` action will fail.

### After First Successful Deployment (Non-Blocking)

2. **Configure Custom Domain DNS**: At DNS provider, create a CNAME record: `bio.jacktracey.co.uk` → `jtracey93.github.io`. DNS propagation may take up to 48 hours.
3. **Set Custom Domain in GitHub**: Repository Settings → Pages → Custom domain → enter `bio.jacktracey.co.uk` → Save. Check "Enforce HTTPS" once the certificate is provisioned.
4. **Configure Branch Protection**: Repository Settings → Branches → Add rule for `main`: require pull request reviews, require status checks to pass (select the deploy workflow), prevent direct pushes.
5. **Verify Domain (recommended)**: GitHub account Settings → Pages → Verified domains → add and verify `jacktracey.co.uk`.

### Dual Domain Behaviour

Once the custom domain is configured:
- `https://bio.jacktracey.co.uk` → serves the site directly
- `https://jtracey93.github.io/bio` → 301 redirects to `https://bio.jacktracey.co.uk`

Both URLs are accessible. The default GitHub Pages URL automatically redirects to the custom domain — this is standard GitHub Pages behaviour and requires no additional configuration.

## Constitution Check (Post-Design)

*Re-check after Phase 1 design artifacts are complete.*

| # | Principle | Status | Evidence |
|---|-----------|--------|----------|
| I | Theme-First, Low Maintenance | PASS | Theme copied to `themes/almeida-cv/` from commit `7bf8228`. No custom CSS/JS overrides. No fork. Hugo lookup order available for future overrides. |
| II | Content as Data | PASS | All content in `data/content.yaml` (documented in data-model.md). Templates read from `.Site.Data.content`. No hard-coded content. |
| III | Mobile-First Responsive Design | PASS | Theme SCSS includes responsive breakpoints. No custom layout additions that could break mobile rendering. |
| IV | PDF Export/Print Parity | PASS | Browser print-to-PDF via the theme's built-in `@media print` stylesheet is the primary export mechanism (constitution v1.1.0). `pages` param controls A4 pagination. Same `content.yaml` source for both web and print. CI-based PDF tooling NOT required (principles IV + VI). |
| V | Automated Deployment (NON-NEGOTIABLE) | PASS | Workflow contract defined in `contracts/deploy-workflow.md`. Triggered on push to `main` (via PR merge) + `workflow_dispatch` for out-of-band re-deployment (FR-006). Build → upload artifact → deploy. Manual deployment steps: zero. |
| VI | Simplicity & YAGNI | PASS | Hugo + theme + YAML + one workflow file. Zero JavaScript frameworks. Zero additional build tools. Every dependency justified in research.md. |

**Gate result**: ALL PASS — design is constitution-compliant.

## Generated Artifacts

| Artifact | Path | Description |
|----------|------|-------------|
| Implementation Plan | `specs/001-initial-bio-setup/plan.md` | This file |
| Research | `specs/001-initial-bio-setup/research.md` | Technology decisions with rationale |
| Data Model | `specs/001-initial-bio-setup/data-model.md` | `content.yaml` and `config.toml` schemas |
| Workflow Contract | `specs/001-initial-bio-setup/contracts/deploy-workflow.md` | GitHub Actions deployment contract |
| Quickstart | `specs/001-initial-bio-setup/quickstart.md` | Local development setup guide |
| Agent Context | `.github/agents/copilot-instructions.md` | Updated with project technology stack |
