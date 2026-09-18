# Build Quartz Site

GitHub Action that builds a static site from a Markdown / Obsidian vault with [Quartz](https://github.com/jackyzha0/quartz).

Your repository only needs the content and (optionally) a `quartz.config.yaml`. The action fetches Quartz at the version you pin, installs its dependencies and plugins, builds the site and returns the output directory, ready for any deploy step (GitHub Pages, Cloudflare Pages, Netlify, …).

**Full documentation: <https://raven-wing.github.io/quartz-action/>** — built from [`docs/`](docs) by [`.github/workflows/docs.yml`](.github/workflows/docs.yml) on every push, using this action itself.

## Usage

```yaml
steps:
  - uses: actions/checkout@v6
    with:
      fetch-depth: 0 # full history for git-based "created/modified" dates - use 1 for flat history if you want faster checkout

  - id: quartz
    uses: raven-wing/quartz-action@v1
    with:
      quartz-version: v5.0.0
      config: quartz.config.yaml
      directory: .

```

### Deploy to Cloudflare Pages

```yaml
  - uses: cloudflare/wrangler-action@v3
    with:
      apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
      accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
      command: pages deploy ${{ steps.quartz.outputs.output-dir }} --project-name=my-site
```

### Deploy to GitHub Pages

```yaml
  - uses: actions/upload-pages-artifact@v3
    with:
      path: ${{ steps.quartz.outputs.output-dir }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `quartz-version` | yes | | Quartz git ref: tag, branch or commit SHA. Pin a tag or SHA for reproducible builds. |
| `config` | no | Quartz default | Path to your `quartz.config.yaml`, relative to the workspace. |
| `directory` | no | `.` | Content directory, relative to the workspace. |
| `quartz-repository` | no | `jackyzha0/quartz` | Repository to fetch Quartz from, e.g. your own fork. |

## Outputs

| Output | Description |
|---|---|
| `output-dir` | Absolute path of the built site. |

## Notes

- If the content directory is your repository root, add everything that isn't content (`.github`, config folders, …) to `ignorePatterns` in your config.
- Quartz is fetched into `$RUNNER_TEMP`, never into the workspace, so it can't end up in your content.
- The npm cache is keyed on the Quartz version. If you pin a branch rather than a tag or SHA, the cache may be stale, but `npm ci` still installs exactly what the lockfile says.
- `docs/` doubles as the test suite: every push and pull request builds it and fails if no `index.html` comes out.

## License

[MIT](LICENSE)
