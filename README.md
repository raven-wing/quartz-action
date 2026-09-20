# Build Quartz Site

GitHub Action that builds a static site from a Markdown / Obsidian vault with [Quartz](https://github.com/jackyzha0/quartz).

Your repository only needs the content and (optionally) a `quartz.config.yaml`. The action fetches Quartz at the version you pin, installs its dependencies and plugins, builds the site and returns the output directory, ready for any deploy step (GitHub Pages, Cloudflare, Netlify, …).

**Full documentation: <https://raven-wing.github.io/quartz-action/>** — built from [`docs/`](docs) by [`.github/workflows/docs.yml`](.github/workflows/docs.yml) on every push, using this action itself.

## Usage

```yaml
steps:
  - uses: actions/checkout@v6
    with:
      fetch-depth: 0 # full history for git-based "created/modified" dates - use 1 for flat history if you want faster checkout
      persist-credentials: false # the build never pushes, so the token need not stay on disk

  - id: quartz
    uses: raven-wing/quartz-action@v1
    with:
      quartz-version: v5.0.0
      config: quartz.config.yaml
      content-dir: .
```

## Deploying

The action only builds; publishing the result is one more step in the same workflow, pointed at `${{ steps.quartz.outputs.output-dir }}`. Each target has its own page in the docs, with the secrets, permissions and settings it needs:

- [GitHub Pages](https://raven-wing.github.io/quartz-action/deploy/github-pages) — free hosting from the repository that holds the notes, nothing to sign up for.

Anything else that serves static files works too — the output is a plain folder of HTML, CSS and images — but GitHub Pages is the only path verified end to end so far.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `quartz-version` | yes | | Quartz git ref: tag, branch or commit SHA. Pin a tag or SHA for reproducible builds. |
| `config` | no | Quartz default | Path to your `quartz.config.yaml`, relative to the workspace. |
| `content-dir` | no | `.` | Content directory, relative to the workspace. |
| `quartz-repository` | no | `jackyzha0/quartz` | Repository to fetch Quartz from, e.g. your own fork. |

## Outputs

| Output | Description |
|---|---|
| `output-dir` | Absolute path of the built site. |

## Notes

- If the content directory is your repository root, add everything that isn't content (`.github`, config folders, …) to `ignorePatterns` in your config.
- Quartz is checked out and immediately moved to `$RUNNER_TEMP`, so the build never sees it and it can't end up in your content.
- The npm cache is keyed on the Quartz version. If you pin a branch rather than a tag or SHA, the cache may be stale, but `npm ci` still installs exactly what the lockfile says.
- Plugins are cached separately, keyed on the Quartz version plus a hash of your whole config file, and any cache for the same Quartz version is accepted as a fallback. Editing the config therefore costs seconds, not a cold install: at v5.0.0 `quartz plugin install` compares each plugin against the commit `quartz.lock.json` pins and resets the ones that differ, so a cache from a different config is corrected rather than trusted.
- `docs/` doubles as the test suite: every push and pull request builds it and fails if no `index.html` comes out.

## License

[MIT](LICENSE)
