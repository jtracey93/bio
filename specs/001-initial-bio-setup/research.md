# Research: Initial Bio Website Setup

**Feature**: 001-initial-bio-setup
**Date**: 2026-02-28

This document resolves all NEEDS CLARIFICATION items from the Technical Context and records technology decisions with rationale.

---

## 1. Hugo Extended Version

**Decision**: Hugo Extended v0.157.0

**Rationale**: Latest stable release as of 2026-02-25. The "Extended" variant is required because the almeida-cv theme uses SCSS (`assets/scss/main.scss` and component partials), which requires Hugo's built-in LibSass/Dart Sass compiler. The standard Hugo binary cannot process SCSS.

**Alternatives considered**:
- Hugo standard (non-extended): Rejected — theme SCSS would not compile.
- Older Hugo Extended versions: No benefit; v0.157.0 is stable and actively supported.

---

## 2. almeida-cv Theme (Commit `7bf8228`)

**Decision**: Copy the full theme directory from commit `7bf8228` of `ineesalmeida/almeida-cv` into `themes/almeida-cv/`.

**Rationale**: The constitution mandates copying the theme into the repository (no Git submodules). Commit `7bf8228` is the latest on the `master` branch. Copying gives full control for future overrides via Hugo's lookup order without forking.

**Theme file structure** (from commit `7bf8228`):
```
almeida-cv/
├── assets/scss/
│   ├── main.scss                 # Entry point; imports all components
│   ├── _avatar.scss
│   ├── _badges.scss
│   ├── _basic_info.scss
│   ├── _contacts.scss
│   ├── _custom.scss              # User customisation hook (empty by default)
│   ├── _diplomas.scss
│   ├── _education.scss
│   ├── _experience.scss
│   ├── _interests.scss
│   ├── _languages.scss
│   ├── _layout.scss
│   ├── _profile.scss
│   ├── _references.scss
│   └── _skills.scss
├── layouts/
│   ├── index.html                # Single-page layout entry point
│   ├── 404.html
│   ├── _default/baseof.html
│   └── partials/
│       ├── avatar.html
│       ├── contacts.html
│       ├── diplomas.html
│       ├── education.html
│       ├── experience.html
│       ├── interests.html
│       ├── languages.html
│       ├── profile.html
│       ├── references.html
│       └── skills.html
├── i18n/
│   ├── en.toml, de.toml, es.toml, fr.toml, pl.toml, eo.toml, zh-cn.toml
├── static/img/
│   └── avatar.jpg                # Default avatar placeholder
├── theme.toml                    # Theme metadata
└── exampleSite/                  # Reference only; not copied into project
    ├── config.toml
    └── data/content.yaml
```

**Alternatives considered**:
- Git submodule: Rejected per constitution (principle I) — makes custom overrides harder and adds external dependency.
- Hugo Module (go module): Rejected — adds Go toolchain dependency; overkill for a single-theme static site.

---

## 3. Content Data Model (`data/content.yaml`)

**Decision**: Use the almeida-cv theme's native YAML structure in `data/content.yaml`.

**Rationale**: The theme's Go templates read directly from `.Site.Data.content`. Using the exact expected structure means zero template modifications. The structure supports all required CV sections from the spec (FR-001, FR-008).

**Theme-expected YAML structure**:
```yaml
BasicInfo:
  FirstName: <string>
  LastName: <string>
  Photo: img/avatar.jpg
  Contacts:
    - Icon: <fontawesome class>    # e.g. fas fa-envelope
      Info: <string>               # e.g. email address, URL

Profile: <string>                  # Professional summary paragraph

Experience:
  - Employer: <string>
    Place: <string>
    Positions:
      - Title: <string>
        Date: <string>             # e.g. "Jan 2020 - Present"
        Details:
          - <string>               # Bullet point descriptions
        Badges:
          - <string>               # Technology/skill tags

Education:
  - Course: <string>
    Place: <string>
    Date: <string>
    Details:
      - <string>

Skills:
  - Family: <string>              # Skill category name
    Items:
      - <string>                  # Individual skills

Languages:
  - Name: <string>
    Level: <string>

Diplomas:
  - <string>                     # Certification/diploma names

Interests:
  - <string>                     # Interest/hobby names

References:
  - Name: <string>
    Relation: <string>
    Contacts:
      - Icon: <string>
        Info: <string>
```

**Alternatives considered**:
- Markdown content files: Rejected — theme expects data files, not Markdown. Would require template rewrites.
- TOML or JSON data files: Technically possible but YAML is the theme's documented/example format. Consistency wins.

---

## 4. Hugo Configuration (`config.toml`)

**Decision**: Use `config.toml` at root with the theme's expected `[params]` section.

**Rationale**: The theme reads colour variables and layout options from `config.toml` params. The example site provides a working reference configuration. Key params:

```toml
baseURL = "https://bio.jacktracey.co.uk/"
languageCode = "en"
theme = "almeida-cv"
disableKinds = ["page", "section", "taxonomy", "term", "RSS", "sitemap"]

[params]
  # Colour scheme (theme defaults)
  color = "#1B9CE2"               # Primary accent colour
  backgroundColor = "#F2F7F8"
  textPrimaryColor = "#4A4E69"
  textSecondaryColor = "#9A8C98"
  sectionColor = "#2B2D42"

  # Layout
  pages = 1                       # Number of A4 pages for print
  swapColumns = false              # Left/right column order
  footerNote = ""                  # Optional footer text
```

**Key decision — `baseURL`**: Set to `https://bio.jacktracey.co.uk/` (the custom domain). GitHub Pages will serve the site at this URL. The default `jtracey93.github.io/bio` URL will 301-redirect to the custom domain automatically once custom domain is configured in GitHub Pages settings.

**Alternatives considered**:
- `config.yaml` or `hugo.toml`: Either works with Hugo, but the theme's example uses `config.toml`. Matching the theme's convention reduces confusion.

---

## 5. GitHub Actions Workflow

**Decision**: Use `actions/deploy-pages` with the artifact-based deployment pattern.

**Rationale**: Spec FR-006 mandates this approach. The workflow uses `peaceiris/actions-hugo` for Hugo Extended installation in CI (widely adopted, actively maintained). The `actions/upload-pages-artifact` + `actions/deploy-pages` pattern is GitHub's official recommended approach for Pages deployment.

**Workflow structure**:
```yaml
name: Deploy Hugo site to GitHub Pages
on:
  push:
    branches: [main]
  workflow_dispatch:    # Manual trigger for debugging

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: '0.157.0'
          extended: true
      - run: hugo --minify
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

**Alternatives considered**:
- `gh-pages` branch deployment: Rejected — the artifact-based approach is newer, officially recommended, and doesn't pollute the repo with a deployment branch.
- `peaceiris/actions-gh-pages`: Rejected — requires a `gh-pages` branch; not compatible with the artifact-based approach specified in FR-006.

---

## 6. Custom Domain + GitHub Pages (Dual Access)

**Decision**: Configure custom domain `bio.jacktracey.co.uk` via CNAME file + GitHub Pages settings. Both URLs accessible.

**Rationale**: The spec requires the site at `bio.jacktracey.co.uk` (FR-007) and edge cases require fallback to the default GitHub Pages URL. GitHub Pages natively handles this:

- A `static/CNAME` file containing `bio.jacktracey.co.uk` is included in the Hugo project (Hugo copies `static/` contents to the output root)
- Custom domain is set in GitHub Pages settings
- `https://bio.jacktracey.co.uk` → serves the site
- `https://jtracey93.github.io/bio` → 301 redirects to `https://bio.jacktracey.co.uk`

Both URLs are accessible; the default URL redirects to the custom domain. This is standard GitHub Pages behaviour.

**DNS requirement**: CNAME record `bio.jacktracey.co.uk` → `jtracey93.github.io` (manual step for repository owner).

**Alternatives considered**:
- No custom domain (default URL only): Rejected — FR-007 requires custom domain.
- Apex domain (e.g. `jacktracey.co.uk`): Not requested; subdomain CNAME is simpler than apex A/AAAA records.

---

## 7. PDF Export / Print Parity

**Decision**: Use the theme's built-in print stylesheet via browser Print-to-PDF.

**Rationale**: The almeida-cv theme includes print-optimised CSS that hides non-essential elements and formats the page for A4 output. The `pages` param in `config.toml` controls pagination. This satisfies FR-005 and constitution principle IV without additional tooling.

The theme's README documents this approach: open the page → Ctrl+P → Save as PDF. The `pages = 1` param targets single-page output; adjust if content grows.

**Alternatives considered**:
- `wkhtmltopdf` in CI: Adds a heavy dependency; browser print already works per theme design. Violates principle VI (YAGNI).
- Puppeteer/Playwright headless PDF: Same concern; adds Node.js dependency to CI for something the browser handles natively.
- `hugo-to-pdf`: Not a widely adopted tool; uncertain maintenance status.

---

## 8. `.gitignore` Configuration

**Decision**: Ignore Hugo build output and cache directories.

**Rationale**: Standard Hugo `.gitignore` entries prevent committing generated files:

```
/public/
/resources/_gen/
/.hugo_build.lock
```

**Alternatives considered**: None — this is standard practice.
