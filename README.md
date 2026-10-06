# Build Quartz Site

GitHub Action that builds a static site from a Markdown / Obsidian vault with [Quartz](https://github.com/jackyzha0/quartz).

Your repository only needs the content and (optionally) a `quartz.config.yaml`. The action fetches Quartz at the version you pin, installs its dependencies and plugins, builds the site and returns the output directory, ready for any deploy step (GitHub Pages, Cloudflare, Netlify, …).

**Full documentation: <https://raven-wing.github.io/quartz-action/>** — built from [`docs/`](docs) by [`.github/workflows/docs.yml`](.github/workflows/docs.yml) on every push, using this action itself.

## Usage

```yaml
steps:
  - uses: actions/checkout@v7
    with:
      fetch-depth: 0 # full history for git-based "created/modified" dates - use 1 for flat history if you want faster checkout
      persist-credentials: false # the build never pushes, so the token need not stay on disk

  - id: quartz
    uses: raven-wing/quartz-action@v0
    with:
      quartz-version: 97a2d05f80c4c50534959b1d0d41cc4b3895625e # v5 branch, 2026-09-20
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
| `quartz-version` | yes | | Quartz git ref: tag, branch or commit SHA. Pin a SHA for reproducible builds. Oldest supported: `97a2d05f80c4c50534959b1d0d41cc4b3895625e` (v5 branch, 2026-09-20); v5.0.0 is rejected. |
| `config` | no | Quartz default | Path to your `quartz.config.yaml`, relative to the workspace. |
| `content-dir` | no | `.` | Content directory, relative to the workspace. |
| `quartz-repository` | no | `jackyzha0/quartz` | Repository to fetch Quartz from, e.g. your own fork. |

## Outputs

| Output | Description |
|---|---|
| `output-dir` | Absolute path of the built site. |

## Notes

- If the content directory is your repository root, add everything that isn't content (`.github`, config folders, …) to `ignorePatterns` in your config.
- Plugins are npm dependencies of Quartz, pinned by its `package-lock.json`. Name them as `source: "@quartz-community/<name>"` (quoted: `@` cannot start a plain YAML value). A plugin Quartz doesn't depend on must be a `github:` source, which the build clones.
- Stuck on Quartz v5.0.0? Pin the last action commit that supported it, [`16a0abc`](https://github.com/raven-wing/quartz-action/tree/16a0abcb7fe2a68dc6e9d3f47868bc9043ae4853). It gets no further fixes.

## License

[MIT](LICENSE)
