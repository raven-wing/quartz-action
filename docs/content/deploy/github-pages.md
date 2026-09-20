---
title: GitHub Pages
tags:
  - deploy
---

Free hosting straight from the repository that holds the notes. Nothing to sign up for, no secrets to configure.

## Workflow

`.github/workflows/deploy.yml`:

```yaml
name: Deploy site
on:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0 # full history, so page dates come from commits
          persist-credentials: false # the build never pushes

      - id: quartz
        uses: raven-wing/quartz-action@v1
        with:
          quartz-version: v5.0.0
          config: quartz.config.yaml
          content-dir: .

      - uses: actions/upload-pages-artifact@v3
        with:
          path: ${{ steps.quartz.outputs.output-dir }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

## Turn Pages on

Settings → Pages → Source: **GitHub Actions**. Without this the deploy job fails, because there is nothing listening for the artifact.

## baseUrl

A project site lives under the repository name, and `baseUrl` has to say so or every internal link breaks:

```yaml
baseUrl: username.github.io/my-vault
```

For a user or organisation site — the repository named `username.github.io` — drop the path: `baseUrl: username.github.io`.

## Custom domain

Point the domain at Pages, set it in Settings → Pages, and put the bare domain in `baseUrl`.
