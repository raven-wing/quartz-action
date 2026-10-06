---
title: Quickstart
tags:
  - guide
---

From an empty repository to a published site.

## 1. Put your notes in the repository

Anywhere you like — the repository root, or a subfolder such as `content/`. That path is the `content-dir` input.

## 2. Add a config (optional)

Without `config`, the site builds with Quartz's default configuration. To customise the title, theme or plugins, copy [`quartz.config.default.yaml`](https://github.com/jackyzha0/quartz/blob/v5/quartz.config.default.yaml) from the version you pin, and set at least:

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

Any other static host works the same way: hand it `output-dir`. Only the GitHub Pages route is documented end to end so far.


## 4. Build it

Whichever target you pick, the middle of the workflow is the same:

```yaml
- uses: actions/checkout@v6
  with:
    fetch-depth: 0 # full history, so page dates come from commits
    persist-credentials: false # the build never pushes, so the token need not stay on disk

- id: quartz
  uses: raven-wing/quartz-action@v0
  with:
    quartz-version: 97a2d05f80c4c50534959b1d0d41cc4b3895625e # v5 branch, 2026-09-20
    config: quartz.config.yaml
    content-dir: .
```

`fetch-depth: 0` is worth keeping: pages without a date in their frontmatter take it from the last commit that changed the file. On a default shallow checkout git sees one squashed commit, so every page would show the date of your latest push.

`persist-credentials: false` keeps checkout's token off disk, where nothing in this build needs it. Leave it out if a later step in the same workflow needs to push.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `quartz-version` | yes | | Quartz git ref: tag, branch or commit SHA. `97a2d05` or later. |
| `config` | no | Quartz default | Path to your `quartz.config.yaml`, relative to the workspace. |
| `content-dir` | no | `.` | Content directory, relative to the workspace. |
| `quartz-repository` | no | `jackyzha0/quartz` | Where to fetch Quartz from, e.g. your own fork. |

Output: `output-dir`, the absolute path of the finished site.

> [!tip] Pin a tag or a commit
> `quartz-version` takes a branch name too, but then a build can change under you with no commit on your side. Pin a SHA and upgrade deliberately.

> [!note] Minimum supported version: `97a2d05`
> The oldest Quartz this action supports is `97a2d05f80c4c50534959b1d0d41cc4b3895625e` on the `v5` branch (2026-09-20). Earlier commits are not supported, and v5.0.0 is rejected outright: it compiles every plugin from git, which takes about ten minutes per cold build. Plugins come from npm only on the `v5` branch, with no release tag yet — so pin this commit or a later one from that branch.
>
> Still on v5.0.0 (`ab346fa66a895e12d63a308e70ce330ba795822a`)? Pin the last action commit that supported it, [`16a0abc`](https://github.com/raven-wing/quartz-action/tree/16a0abcb7fe2a68dc6e9d3f47868bc9043ae4853). It caches the plugins v5.0.0 compiles from git, so only the first build takes the full ten minutes. It gets no further fixes.
>
> ```yaml
> - uses: raven-wing/quartz-action@16a0abcb7fe2a68dc6e9d3f47868bc9043ae4853
>   with:
>     quartz-version: v5.0.0
> ```

## Build time

Plugins come from npm with the rest of Quartz's dependencies, so a build takes well under a minute, and the npm cache makes later ones faster still. Nothing to configure.
