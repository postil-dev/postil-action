# Postil Review Action

Run the [Postil](https://postil.dev) low-noise PR review gate as a GitHub
Action. The action installs the [`postil-cli`](https://github.com/postil-dev/postil-cli)
binary at a pinned commit SHA (or crates.io version) and invokes
`postil review` against the pull request.

The action does no review logic of its own. It is a thin wrapper around the
CLI, so the same engine runs locally, in CI, and in the hosted Postil worker.

## Usage

Standard `pull_request` workflow:

```yaml
name: Postil Review
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  review:
    if: github.event.pull_request.draft == false
    runs-on: ubuntu-latest
    steps:
      - uses: postil-dev/postil-action@v1
        with:
          api-key: ${{ secrets.OPENROUTER_API_KEY }}
```

`pull_request_target` workflow (safer for forks; never builds untrusted code):

```yaml
name: Postil Review
on:
  pull_request_target:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  review:
    if: github.event.pull_request.draft == false
    runs-on: ubuntu-latest
    steps:
      - name: Checkout trusted base
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.base.sha }}
          persist-credentials: false

      - uses: postil-dev/postil-action@v1
        with:
          api-key: ${{ secrets.OPENROUTER_API_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          repo: ${{ github.repository }}
          pr: ${{ github.event.pull_request.number }}
          sha: ${{ github.event.pull_request.head.sha }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | yes | | OpenRouter API key. |
| `github-token` | no | `github.token` | Token for GitHub API reads, inline review comments, and check-run output. |
| `cli-ref` | no | | Full 40-character commit SHA in `postil-dev/postil-cli` to install. |
| `cli-version` | no | | Crates.io version of `postil-cli`. Mutually exclusive with `cli-ref`. |
| `model` | no | `deepseek/deepseek-v4-pro` | Primary OpenRouter model. |
| `model-cascade` | no | | Comma-separated fallback model list. |
| `fail-on` | no | `error` | Severity threshold for non-zero exit. |
| `no-inline` | no | `false` | When `true`, skip inline review comments. |
| `config-path` | no | | Path to a runtime config file (YAML or JSON). |
| `repo` | no | | Override for `owner/name`. |
| `pr` | no | | Override for PR number. |
| `sha` | no | | Override for PR head SHA. |
| `github-api-url` | no | | GitHub Enterprise base URL. |
| `openrouter-api-url` | no | | OpenRouter base URL. |
| `diff-limit` | no | | Maximum diff bytes passed to the model. |
| `check-name` | no | | Check-run name override (default `postil/review`). |
| `output-json` | no | | Path to write the structured envelope. |

## Outputs

| Output | Description |
|---|---|
| `conclusion` | `success` / `neutral` / `failure` — the resolved check conclusion. |
| `findings-count` | Number of findings posted to the PR. |

## Required branch protection (recommended)

For Postil to act as a hard merge gate, mark `postil/review` as a required
status check in branch protection. The Action also writes a job summary to
`GITHUB_STEP_SUMMARY` so the result is visible in the workflow run page.

## License

Apache-2.0.
