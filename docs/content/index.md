---
title: Build Quartz Site
---

This site is the self-documenting vault of [quartz-action](https://github.com/raven-wing/quartz-action) — built and deployed by the action itself on every push to `main`.

A GitHub Action that publishes a folder of Markdown notes as a wiki — searchable, cross-linked, hosted for free. You need your notes, a config file and a workflow; the action does the rest with [Quartz](https://github.com/jackyzha0/quartz).

```yaml
- uses: actions/checkout@v6
  with:
    fetch-depth: 0
    persist-credentials: false

- id: quartz
  uses: raven-wing/quartz-action@v1
  with:
    quartz-version: v5.0.0
    config: quartz.config.yaml
    content-dir: .

- uses: actions/upload-pages-artifact@v3
  with:
    path: ${{ steps.quartz.outputs.output-dir }}
```

That is the whole integration. The output is a directory, so any deploy step takes it — GitHub Pages above, or Cloudflare, Netlify, `rsync`.

## What is it for?

It turns a folder of Markdown notes into a searchable wiki for your team, with no CMS to run and nothing to pay for.

- **Write in the editor you already use.** Obsidian, VS Code, or GitHub's web editor — the source stays plain Markdown files.
- **The repository is the wiki.** History, pull requests and review work exactly as they do for code; publishing is a push.
- **Readers get a real site.** Full-text search, backlinks, a link graph, dark mode and mobile layout, all generated from the notes themselves.
- **Free, static hosting.** The build outputs a folder of HTML — GitHub Pages serves it for nothing, public or behind SSO on an internal host.
- **Nothing to maintain.** No database, no server, no plugin updates in your repository: one pinned version line in a workflow.

## Who is it for?

- A team or organisation handbook: onboarding, processes, decisions.
- Project and research notes that already live in an Obsidian vault.
- Public documentation for a small project or a foundation.
- A personal digital garden or lab notebook.

Less good for people who won't touch git.

## Want it private?

The site itself doesn't have to be public: put it behind an identity proxy such as Cloudflare Access and only your team gets in.

## Next

- [[quickstart]] — from empty repository to published site.
- [[deploy/github-pages]] — a workflow to copy.
- [[build-time]] — why the first build is slow and later ones aren't.
