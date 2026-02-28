<!--
  Sync Impact Report
  ==================
  Version change: N/A → 1.0.0 (initial ratification)
  Modified principles: N/A (first version)
  Added sections:
    - Core Principles (I–VI)
    - Technology Stack & Constraints
    - Development Workflow
    - Governance
  Removed sections: N/A
  Templates requiring updates:
    - .specify/templates/plan-template.md ✅ compatible (no changes needed)
    - .specify/templates/spec-template.md ✅ compatible (no changes needed)
    - .specify/templates/tasks-template.md ✅ compatible (no changes needed)
  Follow-up TODOs: None
-->

# Bio Website Constitution

## Core Principles

### I. Theme-First, Low Maintenance

All visual design and layout MUST leverage the
[almeida-cv](https://github.com/ineesalmeida/almeida-cv) Hugo
theme. Custom CSS/JS overrides MUST be minimised and scoped to
a single override file per type (e.g. `assets/css/custom.css`).
New features MUST NOT fork or vendor the theme; use Hugo's
standard theme-override mechanism instead.

**Rationale**: A pre-built theme keeps long-term maintenance
trivial and defers design decisions to an established,
community-tested foundation.

### II. Content as Data

All bio/CV/resume content MUST be authored in Hugo data files
(YAML/TOML/JSON in `data/`) or Markdown front-matter — never
hard-coded in templates or HTML. This ensures the PDF export
and the website always render from a single source of truth.

**Rationale**: Single-source content eliminates drift between
the web view and the exported PDF, and makes updates a
data-only change.

### III. Mobile-First Responsive Design

Every page MUST render correctly and legibly on viewports from
320px width upward. The theme's responsive capabilities MUST be
validated; any custom additions MUST use relative units and
responsive breakpoints. No horizontal scrolling is permitted on
any target device.

**Rationale**: A personal bio site will be viewed on phones and
tablets as often as desktops; broken mobile rendering directly
harms the user's professional image.

### IV. PDF Export/Print Parity

The site MUST provide a mechanism to export the full
bio/CV/resume to a well-formatted PDF or print to PDF. The PDF/Print output MUST
faithfully reflect the same content rendered on the website.
The export process MUST be automated (reproducible via a single
command or CI step) and MUST NOT require manual formatting.

**Rationale**: Job applications universally require PDF uploads;
automated export from the same source content eliminates manual
copy-paste errors and formatting drift.

### V. Automated Deployment (NON-NEGOTIABLE)

The site MUST be built and deployed to GitHub Pages at
`bio.jacktracey.co.uk` exclusively via GitHub Actions in this
repository. Manual deployment steps are forbidden. The CI
pipeline MUST:
- Build the Hugo site.
- Run any configured validations (broken links, HTML checks).
- Deploy to GitHub Pages on push to `main` when pull requests are merged.

**Rationale**: Fully automated CI/CD ensures every merged change
is live without human error and provides an auditable deployment
history.

### VI. Simplicity & YAGNI

Start with the minimal viable configuration for each feature.
Do NOT add JavaScript frameworks, build toolchains beyond Hugo,
or complex integrations unless a concrete, documented
requirement demands it. Every dependency MUST be justified.

**Rationale**: A personal static site has a single maintainer;
unnecessary complexity increases the maintenance burden with no
proportional benefit.

## Technology Stack & Constraints

- **Static Site Generator**: Hugo extended (latest stable release)
- **Theme**: almeida-cv
  (`https://github.com/ineesalmeida/almeida-cv`), installed as
  a Hugo module by copying the `theme/` directory from the theme repo into this project and referencing it in `config.toml`. Do not use Git submodules or external dependencies for the theme so custom overrides can be easily managed within the same repository.
- **Hosting**: GitHub Pages, served at `bio.jacktracey.co.uk`
- **CI/CD**: GitHub Actions (workflow files in `.github/workflows/`)
- **PDF Export**: Browser-based print stylesheet or a Hugo/CLI
  tool (e.g. `hugo-to-pdf`, `wkhtmltopdf`, Puppeteer); chosen
  tool MUST run in CI without manual intervention
- **Content Format**: YAML data files in `data/` and/or
  Markdown with front-matter
- **Version Control**: Git; `main` branch is the deployment
  source

## Development Workflow

1. **Content changes**: Edit data files or Markdown in a feature
   branch; open a PR against `main`.
2. **Theme overrides**: Place override files under the project's
   `layouts/` or `assets/` directories following Hugo's lookup
   order. Document the reason for each override in a comment at
   the top of the file.
3. **Local preview**: Run `hugo server` locally before pushing.
   The site MUST build without errors or warnings.
4. **PR review**: All PRs MUST pass the GitHub Actions build
   before merge. Force-pushes to `main` are prohibited.
5. **PDF validation**: After any content change, verify the PDF
   export renders correctly (automated in CI where feasible).

## Governance

This constitution is the authoritative reference for all
decisions in this repository. It supersedes ad-hoc preferences
or external guides when they conflict.

- **Amendments** MUST be documented in a PR with a clear
  rationale and updated version number.
- **Version numbering** follows Semantic Versioning:
  - MAJOR: Principle removed or fundamentally redefined.
  - MINOR: New principle or section added, or material
    expansion of existing guidance.
  - PATCH: Wording, typo, or non-semantic clarification.
- **Compliance review**: Every PR SHOULD be checked against the
  principles above. The CI pipeline enforces what it can
  (build success, deployment); reviewers enforce the rest.

**Version**: 1.0.0 | **Ratified**: 2026-02-28 | **Last Amended**: 2026-02-28
