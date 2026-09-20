---
title: Quickstart
tags:
  - guide
---

From an empty repository to a published site.

## 1. Put your notes in the repository

Anywhere you like — the repository root, or a subfolder such as `content/`. That path is the `content-dir` input.

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
- [[deploy/cloudflare]] — a custom domain on the free plan, and SSO in front of the site if the wiki is internal.
- [[deploy/other-hosts]]


## 4. Build it

Whichever target you pick, the middle of the workflow is the same:

```yaml
- uses: actions/checkout@v6
  with:
    fetch-depth: 0 # full history, so page dates come from commits
    persist-credentials: false # the build never pushes, so the token need not stay on disk

- id: quartz
  uses: raven-wing/quartz-action@v1
  with:
    quartz-version: v5.0.0
    config: quartz.config.yaml
    content-dir: .
```

`fetch-depth: 0` is worth keeping: pages without a date in their frontmatter take it from the last commit that changed the file. On a default shallow checkout git sees one squashed commit, so every page would show the date of your latest push.

`persist-credentials: false` is not required, but there is no reason to leave it out: it stops checkout from storing its token in `.git/config`, where the plugin build scripts that run later in the job could read it. Drop the line only if a later step in the same workflow needs to push.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `quartz-version` | yes | | Quartz git ref: tag, branch or commit SHA. |
| `config` | no | Quartz default | Path to your `quartz.config.yaml`, relative to the workspace. |
| `content-dir` | no | `.` | Content directory, relative to the workspace. |
| `quartz-repository` | no | `jackyzha0/quartz` | Where to fetch Quartz from, e.g. your own fork. |

Output: `output-dir`, the absolute path of the finished site.

> [!tip] Pin a tag or a commit
> `quartz-version` takes a branch name too, but then a build can change under you with no commit on your side. Pin `v5.0.0` or a SHA and upgrade deliberately.

## Build time

The first build takes several minutes: Quartz up to v5.0.0 ships no prebuilt distribution, so every plugin in your config is cloned, installed and compiled from source — measured at around ten minutes and 2.2 GB for the default set, even with a warm npm cache. The action caches the result, and later runs restore it in about a second. Each repository builds its own cache, and a new `quartz-version` starts from cold.

To shorten that first run, **delete** plugins you don't use from the config rather than setting `enabled: false` — every entry gets built either way, and `latex`, `og-image` and `favicon` account for some 375 MB of it.

Editing the config does not usually cost you the cache: the key hashes only the plugin `source:` lines, so a new title, theme or `baseUrl` reuses what is already there. Adding a plugin falls back to the previous cache for the same Quartz version and builds just the new one.

> [!note] Removed plugins linger in the cache
> Nothing deletes them: `plugin install` only ever adds, and `plugin prune` compares the lockfile against your config, so in CI — where the lockfile is Quartz's own, listing every default plugin — it would delete everything a trimmed config leaves out, on every run. The files are harmless, since Quartz instantiates what your config lists rather than what sits on disk, but they do not disappear on their own: every run restores them and saves them back under the new key. GitHub's 7-day eviction counts from last *access*, so it only retires the older entries nobody reads. To be rid of them, bump `quartz-version` — a different key prefix means no fallback and a clean install — or delete the caches under **Actions → Caches** (`gh cache delete` does the same).
