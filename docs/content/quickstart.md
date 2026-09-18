---
title: Quickstart
tags:
  - guide
---

From an empty repository to a published site.

## 1. Put your notes in the repository

Anywhere you like — the repository root, or a subfolder such as `content/`. That path is the `directory` input.

## 2. Add a config (optional)

Without `config`, the site builds with Quartz's default configuration. To customise the title, theme or plugins, copy [`quartz.config.default.yaml`](https://github.com/jackyzha0/quartz/blob/v5.0.0/quartz.config.default.yaml) from the version you pin, and set at least:

```yaml
configuration:
  pageTitle: My vault
  baseUrl: username.github.io/my-vault # where the site will be served from
  ignorePatterns:
    - .github
    - .obsidian
    - private
```

> [!warning] Your config replaces the default, it is not merged into it
> If the file has no `plugins:` list, you get no plugins. Start from the default file rather than from a few lines.

## 3. Pick where to publish

The action builds the site and hands you a directory. What you do with it is one more step in the same workflow:

- [[deploy/github-pages]] — free hosting in the same repository, nothing to sign up for.
- [[deploy/cloudflare-pages]]
- [[deploy/other-hosts]]


## 4. Build it

Whichever target you pick, the middle of the workflow is the same:

```yaml
- uses: actions/checkout@v6
  with:
    fetch-depth: 0 # full history, so page dates come from commits

- id: quartz
  uses: raven-wing/quartz-action@v1
  with:
    quartz-version: v5.0.0
    config: quartz.config.yaml
    directory: .
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `quartz-version` | yes | | Quartz git ref: tag, branch or commit SHA. |
| `config` | no | Quartz default | Path to your `quartz.config.yaml`, relative to the workspace. |
| `directory` | no | `.` | Content directory, relative to the workspace. |
| `quartz-repository` | no | `jackyzha0/quartz` | Where to fetch Quartz from, e.g. your own fork. |

Output: `output-dir`, the absolute path of the finished site.

> [!tip] Pin a tag or a commit
> `quartz-version` takes a branch name too, but then a build can change under you with no commit on your side. Pin `v5.0.0` or a SHA and upgrade deliberately.
