# Quickstart: Initial Bio Website Setup

**Feature**: 001-initial-bio-setup
**Date**: 2026-02-28

---

## Prerequisites

- **Hugo Extended v0.157.0+** installed locally
  - Windows: `winget install Hugo.Hugo.Extended` or download from [Hugo releases](https://github.com/gohugoio/hugo/releases)
  - macOS: `brew install hugo`
  - Verify: `hugo version` should show `hugo v0.157.0...+extended`
- **Git** installed
- The repository `jtracey93/bio` cloned locally

---

## Local Development

### 1. Clone and switch to the feature branch

```bash
git clone https://github.com/jtracey93/bio.git
cd bio
git checkout 001-initial-bio-setup
```

### 2. Verify Hugo Extended is installed

```bash
hugo version
# Expected: hugo v0.157.0-...-...-..+extended ...
```

The `+extended` suffix is required — the theme uses SCSS.

### 3. Start the local development server

```bash
hugo server
```

The site is available at `http://localhost:1313`. Hugo watches for file changes and live-reloads.

### 4. Edit content

All CV content is in `data/content.yaml`. Edit this file to update:
- Name, title, employer
- Professional summary
- Work experience entries
- Skills, education, languages
- Contact details

Changes appear instantly in the browser via live reload.

### 5. Preview print/PDF output

1. Open `http://localhost:1313` in a browser
2. Press `Ctrl+P` (Windows) or `Cmd+P` (macOS)
3. Set destination to "Save as PDF"
4. Ensure "Background graphics" is enabled
5. Save — the output should be a clean, professional CV

### 6. Build the site (validation)

```bash
hugo --minify
```

The built site is output to `./public/`. A zero exit code means the build succeeded. This is the same command the CI workflow runs.

---

## Project File Quick Reference

| File | Purpose |
|------|---------|
| `config.toml` | Hugo configuration: base URL, theme, colour params |
| `data/content.yaml` | All CV content (single source of truth) |
| `static/CNAME` | Custom domain for GitHub Pages |
| `static/img/avatar.jpg` | Profile photo |
| `themes/almeida-cv/` | Theme files (do not modify directly; use Hugo overrides) |
| `.github/workflows/deploy.yml` | CI/CD: build and deploy to GitHub Pages |
| `.gitignore` | Ignores `public/`, `resources/`, build lock |

---

## Making Changes (Workflow)

1. Create a feature branch from `main`
2. Edit `data/content.yaml` (or other files as needed)
3. Run `hugo server` to preview locally
4. Commit and push the branch
5. Open a PR against `main`
6. Once merged, GitHub Actions builds and deploys automatically

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `hugo: command not found` | Install Hugo Extended: `winget install Hugo.Hugo.Extended` |
| `SCSS processing failed` | Ensure you have Hugo **Extended** (not standard). Check `hugo version` for `+extended`. |
| Build fails with YAML error | Check `data/content.yaml` for syntax errors. Use a YAML validator. |
| Site looks wrong locally | Ensure `themes/almeida-cv/` exists and is populated. |
| GitHub Pages 404 after deploy | Ensure GitHub Pages source is set to "GitHub Actions" in repo Settings → Pages. |
| Custom domain not working | Check DNS CNAME record points `bio.jacktracey.co.uk` → `jtracey93.github.io`. Allow up to 48h for propagation. |
