# bio.jacktracey.co.uk

Personal bio/CV website built with [Hugo](https://gohugo.io/) and the [almeida-cv](https://github.com/ineesalmeida/almeida-cv) theme. Deployed to GitHub Pages at [bio.jacktracey.co.uk](https://bio.jacktracey.co.uk).

## Prerequisites

- **Hugo Extended** v0.157.0+ ([installation guide](https://gohugo.io/installation/))
  - Windows: `winget install Hugo.Hugo.Extended`
  - macOS: `brew install hugo`

Hugo Extended is required for SCSS compilation.

## Local Development

```sh
hugo server
```

Open [http://localhost:1313](http://localhost:1313) to preview the site. Hugo watches for file changes and reloads the browser automatically.

## Editing Content

All content lives in a single file: `data/content.yaml`. Edit this file to update any part of the CV.

### Structure

| Section | Description |
|---------|-------------|
| `BasicInfo` | Name, subtitle, photo path, and contact links (see below) |
| `Profile` | Professional summary paragraph |
| `Highlights` | Key achievements displayed with icons below the profile (see below) |
| `Experience` | Work history — employer, location, positions with dates, bullet points, and technology badges |
| `Education` | Academic history — course, institution, date |
| `Skills` | Skills grouped by category (e.g. "Cloud Platforms": Azure, Terraform) |
| `Languages` | Language proficiencies |
| `Diplomas` | Certifications and qualifications |
| `Interests` | Personal interests and hobbies |

#### `BasicInfo`

```yaml
BasicInfo:
  FirstName: Jack
  LastName: Tracey
  SubTitle: "Technical Engineering Leader & Product Evangelist"   # optional
  Photo: /img/avatar.jpg
  Contacts:
    - Icon: fas fa-envelope
      Info: <a href="mailto:you@example.com">you@example.com</a>
```

`SubTitle` is rendered as a tagline beneath the name heading. Omit the key to hide it.

#### `Highlights`

```yaml
Highlights:
  - Icon: fas fa-landmark        # Font Awesome icon class
    Title: Short Highlight Title
    Description: A brief description of the achievement or responsibility
```

Each item renders as an icon + title + description row. The section appears below the Profile and is expanded by default on both mobile and desktop. Omit the `Highlights` key entirely to hide the section.

Sections left empty or omitted are automatically hidden by the theme.

### Icons

The site uses **Font Awesome 5.15.3 Free** (loaded via CDN). Icons are used in two places:

| Location | YAML key | Example |
|----------|----------|---------|
| Contact links (right-hand sidebar) | `BasicInfo.Contacts[].Icon` | `fab fa-github` |
| Highlights section | `Highlights[].Icon` | `fas fa-landmark` |

Each `Icon` value is a full Font Awesome class string with two parts: the **style prefix** and the **icon name**.

#### Available style prefixes

| Prefix | Style | Example |
|--------|-------|---------|
| `fas`  | Solid | `fas fa-envelope` |
| `far`  | Regular (outline) | `far fa-envelope` |
| `fab`  | Brands | `fab fa-linkedin-in` |

#### Searching for icons

Browse and search the full icon gallery at:

- **[fontawesome.com/v5/search?m=free](https://fontawesome.com/v5/search?m=free)** — filtered to free icons only

On the search page, click an icon to see its class name. Copy the style prefix + name into the `Icon` field. For example, the search result page for the "users" icon shows `<i class="fas fa-users"></i>`, so you would set:

```yaml
Icon: fas fa-users
```

> **Tip:** Brand icons (social media logos, company marks) always use the `fab` prefix. General-purpose icons typically use `fas` (solid) or `far` (regular/outline).

### Updating the Avatar

Replace `static/img/avatar.jpg` with your photo. The image is referenced by `BasicInfo.Photo` in `content.yaml`.

## Build Validation

```sh
hugo --minify
```

This compiles the site into the `public/` directory. A zero exit code confirms the build is valid.

## Deployment

Deployment is fully automated via GitHub Actions. When a pull request is merged to `main`, the workflow:

1. Checks out the repository
2. Installs Hugo Extended
3. Runs `hugo --minify`
4. Deploys the built site to GitHub Pages

No manual deployment steps are needed. A `workflow_dispatch` trigger is also available for manual re-deployment.

## Export to PDF

Open the site in a browser and press **Ctrl+P** (or **Cmd+P** on macOS) to print. Use "Save as PDF" for a clean, professional CV document. The theme includes a built-in print stylesheet that handles pagination and layout.

The `pages` parameter in `config.toml` controls how many A4 pages the print layout spans. Adjust if content overflows.
