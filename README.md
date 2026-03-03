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
