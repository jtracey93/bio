# Feature Specification: Initial Bio Website Setup

**Feature Branch**: `001-initial-bio-setup`
**Created**: 2026-02-28
**Status**: Draft
**Input**: User description: "Build the initial bio website, setup GitHub Actions, copy the theme from the almeida-cv GitHub repo and set it up, and put some basic data in the content file about Jack Tracey who works at Microsoft. Advise on any manual steps needed to set up GitHub Pages."

## User Scenarios & Testing *(mandatory)*

### User Story 1 — View Bio Online (Priority: P1)

A visitor (recruiter, colleague, or hiring manager) navigates to `bio.jacktracey.co.uk` in any modern browser and sees a clean, professional CV/resume page for Jack Tracey. The page displays personal summary, work experience at Microsoft, skills, education, and contact details. The content loads quickly, renders correctly on both desktop and mobile viewports, and matches the visual style of the almeida-cv theme.

**Why this priority**: The entire purpose of the site is to present Jack's professional profile to visitors. Without a visible, well-rendered page, nothing else matters.

**Independent Test**: Navigate to `bio.jacktracey.co.uk` (or `localhost:1313` locally) and confirm the page renders Jack Tracey's professional information in the almeida-cv layout without errors.

**Acceptance Scenarios**:

1. **Given** the site is deployed, **When** a visitor opens `bio.jacktracey.co.uk` on a desktop browser, **Then** they see Jack Tracey's full CV/resume rendered in the almeida-cv theme with all sections populated.
2. **Given** the site is deployed, **When** a visitor opens `bio.jacktracey.co.uk` on a mobile device (viewport ≤ 480px), **Then** the page is fully readable with no horizontal scrolling and all content accessible.
3. **Given** the site is deployed, **When** a visitor views the page, **Then** it loads in under 3 seconds on a standard connection.

---

### User Story 2 — Automated Deployment on Merge (Priority: P2)

When Jack pushes a content change to the `main` branch (via a merged pull request), the site is automatically rebuilt and deployed to GitHub Pages without any manual steps. The updated content is live within minutes.

**Why this priority**: Automated deployment eliminates manual effort and ensures every merged change is immediately reflected on the live site. This is foundational infrastructure that all future content updates depend on.

**Independent Test**: Merge a trivial content change to `main` and verify GitHub Actions builds the site and the change appears at `bio.jacktracey.co.uk` without manual intervention.

**Acceptance Scenarios**:

1. **Given** a pull request with a content change is merged to `main`, **When** GitHub Actions runs, **Then** the Hugo site is built and deployed to GitHub Pages automatically.
2. **Given** the GitHub Actions workflow completes successfully, **When** a visitor loads `bio.jacktracey.co.uk`, **Then** the updated content is visible.
3. **Given** the Hugo build encounters an error (e.g. invalid front-matter), **When** GitHub Actions runs, **Then** the workflow fails and the previously deployed version remains live.

---

### User Story 3 — Export CV to PDF (Priority: P3)

Jack wants to export his bio/CV as a well-formatted PDF to upload to job applications. He can print the page to PDF from any browser and the output is clean, properly paginated, and free of navigation artefacts — ready to submit to an applicant tracking system.

**Why this priority**: PDF export is essential for job applications but depends on the website being set up and content being in place first (P1). It can be delivered as an enhancement after the core site is live.

**Independent Test**: Open the bio page in a browser, use the browser's Print → Save as PDF function, and confirm the output is a clean, well-formatted CV document.

**Acceptance Scenarios**:

1. **Given** the bio page is loaded in a browser, **When** Jack uses the browser print function (Ctrl+P / Cmd+P), **Then** the print preview shows a clean CV layout without website navigation chrome, headers, or footers.
2. **Given** the print preview is shown, **When** Jack saves to PDF, **Then** the resulting PDF is a single or multi-page document that faithfully represents the CV content.
3. **Given** the exported PDF, **When** Jack opens it in a PDF reader, **Then** all text is selectable and searchable (not rendered as images).

---

### User Story 4 — Update Content Easily (Priority: P4)

Jack wants to update his job title, add a new role, or modify his skills by editing a single data file rather than touching templates or HTML. Changes to the data file are reflected on the site after rebuild.

**Why this priority**: Ease of maintenance is a long-term quality-of-life concern. It matters after the initial setup is complete and Jack starts making real updates.

**Independent Test**: Edit the content data file to change the job title, run `hugo server`, and verify the change appears on the rendered page.

**Acceptance Scenarios**:

1. **Given** the content data file exists, **When** Jack edits a field (e.g. job title) and rebuilds the site, **Then** the updated value appears on the rendered page.
2. **Given** the content data file, **When** Jack adds a new work experience entry, **Then** it appears in the correct chronological position on the rendered page.

---

### Edge Cases

- What happens if the custom domain DNS is not yet configured? The site should still be accessible via the default GitHub Pages URL (`jtracey93.github.io/bio`).
- What happens if the Hugo build fails in CI? The workflow should fail without deploying, preserving the last known-good version.
- What happens if the content data file has a YAML syntax error? Hugo should report a clear build error and the CI pipeline should fail.
- How does the page look when all optional sections (e.g. certifications, projects) are empty? The theme should gracefully hide empty sections rather than showing blank headers.

## Clarifications

### Session 2026-02-28

- Q: What is Jack Tracey's current job title at Microsoft? → A: Senior Cloud Solutions Architect
- Q: Which GitHub Pages deployment mechanism should the workflow use? → A: Artifact-based deploy via `actions/deploy-pages` (GitHub Pages source = "GitHub Actions"), triggered on merge to `main` from PRs only; branch protection prevents direct pushes.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The site MUST render a single-page CV/resume displaying: personal summary, work experience, skills, education, and contact information.
- **FR-002**: All CV content MUST be sourced from a data file (YAML) — not hard-coded in templates.
- **FR-003**: The site MUST use the almeida-cv Hugo theme, copied into the repository's `themes/` directory.
- **FR-004**: The site MUST be responsive and render correctly on viewports from 320px width upward.
- **FR-005**: The site MUST include a print stylesheet that produces a clean, professional PDF when using the browser's Print to PDF function.
- **FR-006**: A GitHub Actions workflow MUST build the Hugo site and deploy to GitHub Pages using the artifact-based `actions/deploy-pages` action on every merge to `main`. The `main` branch MUST be protected with branch protection rules that prevent direct pushes; all changes arrive via pull requests.
- **FR-007**: The site MUST be served at the custom domain `bio.jacktracey.co.uk`.
- **FR-008**: The content data file MUST include placeholder/initial content for Jack Tracey including: name, current title (Senior Cloud Solutions Architect), current employer (Microsoft), a professional summary, at least one work experience entry, a skills section, and contact details.
- **FR-009**: The GitHub Actions workflow MUST fail (and not deploy) if the Hugo build produces errors.
- **FR-010**: The site MUST include a `CNAME` file in the published output for custom domain configuration.

### Key Entities

- **Content Profile**: The primary data entity representing Jack Tracey's professional information. Contains: name, title (Senior Cloud Solutions Architect), employer (Microsoft), summary/about, work experience entries (each with role, company, dates, description), skills (grouped by category), education entries, and contact links (LinkedIn, GitHub, email).
- **Theme**: The almeida-cv Hugo theme providing layout, styling, and responsive design. Referenced by the Hugo configuration and stored in the `themes/` directory.
- **Deployment Pipeline**: The GitHub Actions workflow that builds the site and deploys it. Uses the artifact-based `actions/deploy-pages` action with GitHub Pages source set to "GitHub Actions". Triggered on merge to `main` (via pull requests only; direct pushes are blocked by branch protection). Publishes built static files as a GitHub Pages artifact.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A visitor can view Jack Tracey's complete professional profile at `bio.jacktracey.co.uk` within 3 seconds of page load.
- **SC-002**: The site renders correctly on desktop (1920px), tablet (768px), and mobile (375px) viewports with no horizontal scrolling or overlapping content.
- **SC-003**: A content change merged to `main` is live on the public URL within 5 minutes without manual intervention.
- **SC-004**: The browser Print to PDF output produces a clean, professional document with selectable text and no navigation artefacts.
- **SC-005**: Jack can update any piece of his professional profile by editing a single data file — no template or HTML changes required.
- **SC-006**: The site passes basic accessibility checks (readable font sizes, sufficient colour contrast, semantic heading structure).

## Assumptions

- Jack Tracey currently works at Microsoft. Placeholder content will use this as the primary employer.
- The GitHub repository `jtracey93/bio` already exists and the user has admin access.
- DNS for `bio.jacktracey.co.uk` will be configured manually by the user (a CNAME record pointing to `jtracey93.github.io`). This is documented as a manual step.
- GitHub Pages will be configured to use the `gh-pages` branch or GitHub Actions deployment. This requires a one-time manual settings change in the repository.
- The almeida-cv theme supports Hugo's data-file driven content model. If it requires Markdown content files instead, the data structure will adapt accordingly.
- No authentication or access control is needed — this is a fully public site.

## Manual Steps Required (Post-Implementation)

The following steps cannot be automated and must be performed by the repository owner:

1. **Configure GitHub Pages**: In the repository Settings → Pages, set the source to "GitHub Actions".
2. **Configure Branch Protection**: In the repository Settings → Branches, add a branch protection rule for `main` that requires pull request reviews and prevents direct pushes.
3. **Configure Custom Domain DNS**: Create a CNAME record for `bio.jacktracey.co.uk` pointing to `jtracey93.github.io` at the DNS provider.
3. **Set Custom Domain in GitHub**: In the repository Settings → Pages → Custom domain, enter `bio.jacktracey.co.uk` and enable "Enforce HTTPS".
4. **Verify Domain (recommended)**: In GitHub account settings → Pages → Verified domains, add and verify `jacktracey.co.uk` to prevent others from using the domain with GitHub Pages.
