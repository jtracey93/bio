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
| `BasicInfo` | Name, photo path, and contact links (email, LinkedIn, GitHub) |
| `Profile` | Professional summary paragraph |
| `Experience` | Work history — employer, location, positions with dates, bullet points, and technology badges |
| `Education` | Academic history — course, institution, date |
| `Skills` | Skills grouped by category (e.g. "Cloud Platforms": Azure, Terraform) |
| `Languages` | Language proficiencies |
| `Diplomas` | Certifications and qualifications |
| `Interests` | Personal interests and hobbies |

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
