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

Any other static host works the same way: hand it `output-dir`. Only the GitHub Pages route is documented end to end so far.


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

The first build takes several minutes: Quartz up to v5.0.0 ships no prebuilt distribution, so every plugin in its `quartz.lock.json` is cloned, installed and compiled from source — measured at around ten minutes and 2.2 GB for the default set of 42, even with a warm npm cache. The action caches the result, and later runs restore it in about a second. Each repository builds its own cache, and a new `quartz-version` starts from cold.

> [!note] Trimming your config does not shorten this
> `quartz plugin install` installs what Quartz's own `quartz.lock.json` pins, not what your config lists — so the same 42 plugins are built whatever you write. Deleting an entry or setting `enabled: false` changes which plugins your *site uses*, not which get installed. Trimming the lockfile instead is not an option either: Quartz's sources import named exports from the generated `.quartz/plugins/index.ts`, so dropping `og-image` fails the build with `No matching export ... CustomOgImagesEmitterName`. The cache is what makes later builds fast — a cold run is the price of a new `quartz-version`.

Editing the config rarely costs you much: the key hashes the whole file, so any edit misses it, but the run then falls back to any cache for the same `quartz-version`. `quartz plugin install` checks what it restored against `quartz.lock.json` and resets anything on the wrong commit, so the step finishes in seconds. Only a new `quartz-version` is a genuinely cold build.

> [!note] A restored cache is checked, not trusted
> The fallback can hand a run the plugins another config built. `quartz plugin install` reads each installed plugin's commit and compares it against `quartz.lock.json`, fetching and resetting the ones that differ, so what you end up building against is what the lockfile pins — whatever the cache happened to hold.
