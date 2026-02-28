# Data Model: Initial Bio Website Setup

**Feature**: 001-initial-bio-setup
**Date**: 2026-02-28
**Source**: almeida-cv theme data structure (commit `7bf8228`) + spec FR-008

---

## Overview

The site has a single data entity: the CV content stored in `data/content.yaml`. The almeida-cv theme reads this file via Hugo's `.Site.Data.content` accessor. There is no database, no API, and no dynamic state — the data model is a static YAML file that Hugo compiles into HTML at build time.

---

## Entity: Content Profile (`data/content.yaml`)

### Top-Level Structure

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `BasicInfo` | Object | Yes | Name, photo, and contact details |
| `Profile` | String | Yes | Professional summary paragraph |
| `Experience` | Array\<Experience\> | Yes | Work history entries |
| `Education` | Array\<Education\> | No | Academic history |
| `Skills` | Array\<SkillFamily\> | Yes | Skills grouped by category |
| `Languages` | Array\<Language\> | No | Language proficiencies |
| `Diplomas` | Array\<String\> | No | Certifications and qualifications |
| `Interests` | Array\<String\> | No | Personal interests/hobbies |
| `References` | Array\<Reference\> | No | Professional references |

### BasicInfo

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `FirstName` | String | Yes | Given name (e.g. "Jack") |
| `LastName` | String | Yes | Family name (e.g. "Tracey") |
| `Photo` | String | Yes | Path to avatar image relative to `static/` (e.g. `img/avatar.jpg`) |
| `Contacts` | Array\<Contact\> | Yes | Contact methods |

### Contact

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Icon` | String | Yes | Font Awesome icon class (e.g. `fas fa-envelope`, `fab fa-linkedin-in`) |
| `Info` | String | Yes | Contact value — email address, URL, or display text |

### Experience

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Employer` | String | Yes | Company name (e.g. "Microsoft") |
| `Place` | String | Yes | Location (e.g. "United Kingdom") |
| `Positions` | Array\<Position\> | Yes | Roles held at this employer |

### Position

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Title` | String | Yes | Job title (e.g. "Senior Cloud Solutions Architect") |
| `Date` | String | Yes | Date range (e.g. "Jan 2020 - Present") |
| `Details` | Array\<String\> | No | Bullet-point description of responsibilities/achievements |
| `Badges` | Array\<String\> | No | Technology/skill tags displayed as badges |

### Education

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Course` | String | Yes | Degree or programme name |
| `Place` | String | Yes | Institution name |
| `Date` | String | Yes | Date range or graduation year |
| `Details` | Array\<String\> | No | Additional details or achievements |

### SkillFamily

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Family` | String | Yes | Category name (e.g. "Cloud Platforms", "Programming") |
| `Items` | Array\<String\> | Yes | Individual skill names |

### Language

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | String | Yes | Language name (e.g. "English") |
| `Level` | String | Yes | Proficiency level (e.g. "Native", "Professional") |

### Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | String | Yes | Reference's full name |
| `Relation` | String | Yes | Professional relationship |
| `Contacts` | Array\<Contact\> | Yes | How to reach the reference |

---

## Entity: Site Configuration (`config.toml`)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `baseURL` | String | Yes | Site root URL: `https://bio.jacktracey.co.uk/` |
| `languageCode` | String | Yes | Language: `en` |
| `theme` | String | Yes | Theme directory name: `almeida-cv` |
| `disableKinds` | Array\<String\> | Yes | Disabled Hugo page kinds (single-page site) |
| `[params].color` | String | No | Primary accent colour (hex) |
| `[params].backgroundColor` | String | No | Page background colour (hex) |
| `[params].textPrimaryColor` | String | No | Main text colour (hex) |
| `[params].textSecondaryColor` | String | No | Secondary text colour (hex) |
| `[params].sectionColor` | String | No | Section header colour (hex) |
| `[params].pages` | Integer | No | Number of A4 pages for print layout (default: 1) |
| `[params].swapColumns` | Boolean | No | Swap left/right column order (default: false) |
| `[params].footerNote` | String | No | Optional footer text |

---

## Validation Rules

1. **BasicInfo.FirstName** and **BasicInfo.LastName** MUST be non-empty strings.
2. **BasicInfo.Photo** MUST reference a file that exists in `static/img/`.
3. **BasicInfo.Contacts** MUST contain at least one entry (per FR-008).
4. **Profile** MUST be a non-empty string (per FR-008).
5. **Experience** MUST contain at least one entry with employer "Microsoft" and title "Senior Cloud Solutions Architect" (per FR-008).
6. **Skills** MUST contain at least one family with at least one item (per FR-008).
7. All YAML MUST be valid — Hugo build will fail on syntax errors (CI gate).

---

## State Transitions

Not applicable. This is a static data file with no runtime state. Content changes are made via file edits, committed to Git, and deployed via CI.

---

## Relationship Diagram

```
config.toml
  └── references theme: "almeida-cv"
        └── themes/almeida-cv/layouts/
              └── reads: .Site.Data.content (data/content.yaml)
                    ├── BasicInfo → partials/avatar.html, partials/contacts.html
                    ├── Profile → partials/profile.html
                    ├── Experience → partials/experience.html
                    ├── Education → partials/education.html
                    ├── Skills → partials/skills.html
                    ├── Languages → partials/languages.html
                    ├── Diplomas → partials/diplomas.html
                    ├── Interests → partials/interests.html
                    └── References → partials/references.html
```

Each section of `content.yaml` maps to exactly one theme partial. The theme's `layouts/index.html` orchestrates the partials in display order.
