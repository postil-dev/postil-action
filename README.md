# Postil Review Action

Postil is a low-noise pull request review gate. This action installs the
Postil CLI from `postil-dev/postil-cli` at a pinned commit SHA
and runs it against a pull request.

The action does not build from the caller repository and does not rely on a
website package script. For `pull_request_target`, keep the checkout on trusted
base code and let Postil read the pull request diff through the GitHub API.

## Usage

```yaml
name: Postil review

on:
  pull_request_target:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  postil:
    runs-on: ubuntu-latest
    if: github.event.pull_request.draft == false
    steps:
      - name: Check out trusted base
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.base.sha }}
          persist-credentials: false

      - name: Review pull request
        uses: postil-dev/postil-action@v1
        with:
          api-key: ${{ secrets.OPENROUTER_API_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

When a workflow cannot rely on the standard pull request event payload, pass the
target explicitly:

```yaml
- uses: postil-dev/postil-action@v1
  with:
    api-key: ${{ secrets.OPENROUTER_API_KEY }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repo: ${{ github.repository }}
    pr: ${{ github.event.pull_request.number }}
    sha: ${{ github.event.pull_request.head.sha }}
```

## Inputs

| Input                | Required | Default                                    | Description                                                        |
| -------------------- | -------- | ------------------------------------------ | ------------------------------------------------------------------ |
| `api-key`            | yes      |                                            | OpenRouter API key.                                                |
| `github-token`       | no       | `${{ github.token }}`                      | Token for GitHub API reads and review output.                      |
| `cli-ref`            | no       | `d0c81b91b6082c7788b2dec78eaf5f5bacc348c2` | Full `postil-dev/postil-cli` commit SHA to install.                |
| `reviewer-ref`       | no       |                                            | Deprecated. Use `cli-ref` instead.                                 |
| `model`              | no       | `deepseek/deepseek-v4-pro`                 | Primary OpenRouter model.                                          |
| `model-cascade`      | no       |                                            | Optional comma-separated fallback models.                          |
| `fail-on`            | no       | `error`                                    | Exit 1 for findings at `info`, `warn`, or `error`.                 |
| `no-inline`          | no       | `false`                                    | Set to `true` to skip inline review comments.                      |
| `config-path`        | no       |                                            | Optional local config path. Empty uses repo config through GitHub. |
| `repo`               | no       |                                            | Repository in `owner/name` form.                                   |
| `pr`                 | no       |                                            | Pull request number.                                               |
| `sha`                | no       |                                            | Pull request head SHA.                                             |
| `github-api-url`     | no       |                                            | GitHub API URL override.                                           |
| `openrouter-api-url` | no       |                                            | OpenRouter API URL override.                                       |
| `diff-limit`         | no       |                                            | Maximum diff bytes passed to the CLI.                              |
| `check-name`         | no       |                                            | GitHub check name override.                                        |
| `output-json`        | no       |                                            | Optional path for the structured review envelope.                  |

## CLI Pin

`cli-ref` must be a full 40-character commit SHA. This keeps the action
stable and prevents accidental drift to a moving branch or tag. Update the
default only after the CLI commit has passed its own tests.

`reviewer-ref` is still recognized only to fail fast with a clear migration
message. Use `cli-ref` with a full `postil-dev/postil-cli` commit SHA.

## Security Notes

For `pull_request_target` workflows:

- Check out the trusted base SHA, not the pull request head.
- Do not run dependency install, build, or scripts from the pull request.
- Let the Postil CLI fetch the pull request diff and config through the
  GitHub API.
- Pass secrets only through action inputs or environment variables.
