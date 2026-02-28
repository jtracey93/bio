# Tasks: Initial Bio Website Setup

**Input**: Design documents from `/specs/001-initial-bio-setup/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Not requested — no test tasks included.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3, US4)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create the Hugo project skeleton with all required directories

- [x] T001 Create project directory structure per plan.md (data/, static/img/, themes/, .github/workflows/)
- [x] T002 [P] Create .gitignore ignoring public/, resources/, .hugo_build.lock at .gitignore

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Install the theme and configure Hugo — MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete. Hugo cannot build without a theme and config file.

- [x] T003 [P] Copy almeida-cv theme (commit 7bf8228) from ineesalmeida/almeida-cv into themes/almeida-cv/
- [x] T004 [P] Create Hugo site configuration with theme parameters in config.toml

### T003 Details

Download the full theme directory from commit `7bf8228` of `ineesalmeida/almeida-cv` on GitHub and place all files into `themes/almeida-cv/`. Include: `assets/scss/`, `layouts/`, `i18n/`, `static/img/`, `theme.toml`. Exclude the `exampleSite/` directory. See research.md §2 for the complete file listing.

### T004 Details

Create `config.toml` at repository root with the following configuration (from research.md §4):

- `baseURL = "https://bio.jacktracey.co.uk/"`
- `languageCode = "en"`
- `theme = "almeida-cv"`
- `disableKinds = ["page", "section", "taxonomy", "term", "RSS", "sitemap"]`
- `[params]`: colour scheme (color, backgroundColor, textPrimaryColor, textSecondaryColor, sectionColor), `pages = 1`, `swapColumns = false`

**Checkpoint**: Hugo site skeleton ready — `hugo build` should succeed with an empty/minimal data file

---

## Phase 3: User Story 1 — View Bio Online (Priority: P1) 🎯 MVP

**Goal**: A visitor navigates to `bio.jacktracey.co.uk` (or `localhost:1313` locally) and sees Jack Tracey's complete professional CV/resume rendered in the almeida-cv theme.

**Independent Test**: Run `hugo server`, open `http://localhost:1313`, and confirm the page renders Jack Tracey's professional information in the almeida-cv layout with all sections populated (name, title, summary, experience, skills, education, contacts).

### Implementation for User Story 1

- [x] T005 [P] [US1] Create CV content data file with Jack Tracey's professional information in data/content.yaml
- [x] T006 [P] [US1] Add placeholder avatar image in static/img/avatar.jpg
- [x] T007 [P] [US1] Create custom domain CNAME file containing bio.jacktracey.co.uk in static/CNAME

### T005 Details

Create `data/content.yaml` following the exact schema in data-model.md and the theme-expected YAML structure in research.md §3. Must include (per FR-008):

- **BasicInfo**: FirstName "Jack", LastName "Tracey", Photo "img/avatar.jpg", Contacts (email, LinkedIn, GitHub using Font Awesome icons)
- **Profile**: Professional summary paragraph for a Senior Cloud Solutions Architect at Microsoft
- **Experience**: At least one entry — Employer "Microsoft", Place "United Kingdom", Position Title "Senior Cloud Solutions Architect" with date, details (bullet points), and badges (technology tags)
- **Skills**: At least one family (e.g. "Cloud Platforms") with items (e.g. "Azure", "Terraform", "Bicep")
- **Education**: At least one entry (placeholder)
- Optional sections (Languages, Diplomas, Interests, References): Include with placeholder content or omit — theme gracefully hides empty sections

### T006 Details

Add a placeholder avatar image at `static/img/avatar.jpg`. This can be a simple placeholder image (e.g. a generic professional avatar or solid-colour square). The `BasicInfo.Photo` field in content.yaml references `img/avatar.jpg`. The theme's `partials/avatar.html` renders this image.

### T007 Details

Create `static/CNAME` containing a single line: `bio.jacktracey.co.uk` (no trailing newline). Hugo copies `static/` contents into `public/` at build time, so this file will appear as `public/CNAME` in the build output — which GitHub Pages reads for custom domain configuration (FR-010).

**Checkpoint**: User Story 1 complete — `hugo server` shows Jack Tracey's full CV at `localhost:1313` with all required sections, correct styling, and responsive layout

---

## Phase 4: User Story 2 — Automated Deployment on Merge (Priority: P2)

**Goal**: When a pull request is merged to `main`, GitHub Actions automatically builds the Hugo site and deploys it to GitHub Pages via the artifact-based `actions/deploy-pages` action. A `workflow_dispatch` trigger also supports out-of-band re-deployments during troubleshooting (FR-006).

**Independent Test**: Merge a trivial content change to `main` and verify the GitHub Actions workflow completes successfully and the change appears at the GitHub Pages URL.

### Implementation for User Story 2

- [x] T008 [US2] Create GitHub Actions deployment workflow per contracts/deploy-workflow.md in .github/workflows/deploy.yml

### T008 Details

Create `.github/workflows/deploy.yml` matching the contract in `contracts/deploy-workflow.md` and the detailed structure in research.md §5:

- **Trigger**: `push` to `main` branch + `workflow_dispatch` for out-of-band deployments during troubleshooting (FR-006)
- **Permissions**: `contents: read`, `pages: write`, `id-token: write`
- **Concurrency**: group `pages`, `cancel-in-progress: false`
- **Job `build`**: `actions/checkout@v4` → `peaceiris/actions-hugo@v3` (hugo-version `0.157.0`, extended `true`) → `hugo --minify` → `actions/upload-pages-artifact@v3` (path `./public`)
- **Job `deploy`**: needs `build`, environment `github-pages`, `actions/deploy-pages@v4` with output `page_url`
- Build failure (non-zero exit from `hugo --minify`) MUST prevent deploy job from running (FR-009)

**Checkpoint**: User Story 2 complete — deployment workflow file exists and will trigger on push to `main`

---

## Phase 5: User Story 3 — Export CV to PDF (Priority: P3)

**Goal**: Jack can use the browser's Print → Save as PDF function and get a clean, professional CV document with proper A4 pagination, selectable text, and no navigation artefacts.

**Independent Test**: Open `localhost:1313` in a browser, press Ctrl+P, and confirm the print preview shows a clean CV layout. Save as PDF and verify the output is a well-formatted document.

### Implementation for User Story 3

- [x] T009 [US3] Verify and tune pages parameter in config.toml for correct A4 print pagination

### T009 Details

The almeida-cv theme includes a built-in print stylesheet (via `assets/scss/` with `@media print` rules) — this is the primary PDF export mechanism per constitution v1.1.0 (principle IV). CI-based PDF tooling is NOT required. The `[params].pages` value in `config.toml` (set to `1` in T004) controls how many A4 pages the print layout spans. After content is added in T005:

1. Run `hugo server` and open `localhost:1313`
2. Press Ctrl+P to open print preview
3. Verify the CV content fits within the configured page count
4. If content overflows, increase `pages` value in `config.toml` (e.g. `pages = 2`)
5. Confirm: no navigation chrome, clean layout, selectable text, proper pagination

**Checkpoint**: User Story 3 complete — browser Print-to-PDF produces a clean, professional CV document

---

## Phase 6: User Story 4 — Update Content Easily (Priority: P4)

**Goal**: Jack can update any piece of his professional profile by editing `data/content.yaml` — no template or HTML changes required. Changes are reflected on the site after rebuild.

**Independent Test**: Edit a field in `data/content.yaml` (e.g. change job title), run `hugo server`, and verify the change appears on the rendered page.

### Implementation for User Story 4

- [x] T010 [US4] Create project README with local development setup and content editing guide in README.md

### T010 Details

Create `README.md` at repository root covering:

- **Project overview**: Personal bio/CV website built with Hugo and almeida-cv theme
- **Prerequisites**: Hugo Extended v0.157.0+ (installation commands for Windows/macOS)
- **Local development**: `hugo server` command, live reload at `localhost:1313`
- **Content editing**: Explain that all content lives in `data/content.yaml`; document the structure (BasicInfo, Profile, Experience, Skills, etc.) and how to add/edit entries
- **Build validation**: `hugo --minify` to verify build succeeds
- **Deployment**: Automatic via GitHub Actions on merge to `main` — no manual deployment needed
- Reference quickstart.md for detailed setup/troubleshooting

The README itself is the deliverable for US4 — it documents the single-file editing workflow that makes content updates easy (SC-005).

**Checkpoint**: User Story 4 complete — README documents the content editing workflow; editing `data/content.yaml` and rebuilding reflects changes immediately

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Final validation across all user stories and requirements

- [x] T011 Run quickstart.md validation to verify Hugo builds successfully and page renders at localhost:1313
- [x] T012 [P] Verify all functional requirements (FR-001 through FR-010) are addressed by implementation

### T011 Details

Follow `specs/001-initial-bio-setup/quickstart.md` end-to-end:

1. Verify Hugo Extended v0.157.0 is installed (`hugo version`)
2. Run `hugo server` — confirm zero errors
3. Open `localhost:1313` — confirm CV page renders with all sections
4. Run `hugo --minify` — confirm zero-error build, `public/` directory created with `index.html` and `CNAME`

### T012 Details

Cross-reference each functional requirement against the implemented files:

| FR | Requirement | Addressed By |
|----|-------------|--------------|
| FR-001 | Single-page CV with all sections | T005 (content.yaml) + T003 (theme) + T004 (config.toml) |
| FR-002 | Content from YAML data file | T005 (data/content.yaml) |
| FR-003 | almeida-cv theme copied to themes/ | T003 (themes/almeida-cv/) |
| FR-004 | Responsive 320px+ | T003 (theme responsive SCSS) |
| FR-005 | Print stylesheet for PDF | T003 (theme print CSS) + T009 (pages param) |
| FR-006 | GitHub Actions with deploy-pages + workflow_dispatch | T008 (deploy.yml) |
| FR-007 | Custom domain bio.jacktracey.co.uk | T007 (CNAME) + manual DNS step |
| FR-008 | Jack Tracey placeholder content | T005 (content.yaml) |
| FR-009 | Workflow fails on build errors | T008 (hugo --minify exit code gates deploy) |
| FR-010 | CNAME in published output | T007 (static/CNAME → public/CNAME) |

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 (directories must exist) — **BLOCKS all user stories**
- **US1 (Phase 3)**: Depends on Phase 2 (theme + config must exist for Hugo to build)
- **US2 (Phase 4)**: Depends on Phase 1 (.github/workflows/ directory) — can run in parallel with US1
- **US3 (Phase 5)**: Depends on Phase 2 (config.toml) + Phase 3 T005 (content must exist to test print pagination)
- **US4 (Phase 6)**: Depends on Phase 3 (site must be functional to document the editing workflow)
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **US1 (P1)**: Depends only on Foundational — no dependencies on other stories
- **US2 (P2)**: Depends only on Phase 1 (directory exists) — **can run in parallel with US1**
- **US3 (P3)**: Depends on US1 (content.yaml must exist to verify print pagination)
- **US4 (P4)**: Depends on US1 (must have a working site to document the editing workflow)

### Within Each User Story

- All tasks marked [P] within a phase can run in parallel (different files)
- Complete all tasks in a phase before moving to the next
- Validate the checkpoint before proceeding

### Parallel Opportunities

- **Phase 1**: T001 and T002 can run in parallel
- **Phase 2**: T003 and T004 can run in parallel (theme copy + config creation are independent files)
- **Phase 3**: T005, T006, and T007 can ALL run in parallel (content.yaml, avatar.jpg, CNAME are independent files)
- **Phase 3 + Phase 4**: US1 (T005–T007) and US2 (T008) can run in parallel — they touch completely different files

---

## Parallel Example: Phase 2 (Foundational)

```
# Launch both foundational tasks together:
Task T003: "Copy almeida-cv theme into themes/almeida-cv/"
Task T004: "Create Hugo configuration in config.toml"
```

## Parallel Example: Phase 3 (User Story 1)

```
# Launch all US1 tasks together:
Task T005: "Create data/content.yaml with Jack Tracey's content"
Task T006: "Add placeholder avatar in static/img/avatar.jpg"
Task T007: "Create CNAME in static/CNAME"
```

## Parallel Example: US1 + US2

```
# After Phase 2 is complete, US1 and US2 can start together:
Thread A (US1): T005, T006, T007
Thread B (US2): T008
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001–T002)
2. Complete Phase 2: Foundational (T003–T004) — **CRITICAL, blocks all stories**
3. Complete Phase 3: User Story 1 (T005–T007)
4. **STOP and VALIDATE**: Run `hugo server` — confirm Jack Tracey's CV renders at `localhost:1313`
5. This is a deployable MVP — the bio site works locally

### Incremental Delivery

1. Setup + Foundational → Hugo skeleton ready
2. Add US1 → Test: CV renders locally → **MVP!**
3. Add US2 → Test: Merge to `main` triggers deploy → **Site is live!**
4. Add US3 → Test: Print-to-PDF is clean → **PDF ready**
5. Add US4 → Test: README guides content edits → **Fully documented**
6. Polish → Validate all FRs and quickstart → **Complete**

Each story adds value without breaking previous stories.

---

## Notes

- [P] tasks = different files, no dependencies on incomplete parallel tasks
- [Story] label maps task to specific user story for traceability
- Each user story is independently completable and testable at its checkpoint
- Commit after each task or logical group
- Stop at any checkpoint to validate the story independently
- **Manual steps** (GitHub Pages source, DNS, branch protection) are documented in spec.md and plan.md — they are NOT automated tasks but must be completed by the repository owner
