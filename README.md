# Postil Review Action

Postil is a low-noise pull request review gate. This action installs the
Postil reviewer CLI from `postil-dev/postil-reviewer` at a pinned commit SHA
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
| `reviewer-ref`       | no       | `d51705920a8fca2717a7b9c1bb224e7cdede661a` | Full `postil-dev/postil-reviewer` commit SHA to install.           |
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
| `diff-limit`         | no       |                                            | Maximum diff bytes passed to the reviewer.                         |
| `check-name`         | no       |                                            | GitHub check name override.                                        |
| `output-json`        | no       |                                            | Optional path for the structured review envelope.                  |

## Reviewer Pin

`reviewer-ref` must be a full 40-character commit SHA. This keeps the action
stable and prevents accidental drift to a moving branch or tag. Update the
default only after the reviewer commit has passed its own tests.

## Security Notes

For `pull_request_target` workflows:

- Check out the trusted base SHA, not the pull request head.
- Do not run dependency install, build, or scripts from the pull request.
- Let the Postil reviewer fetch the pull request diff and config through the
  GitHub API.
- Pass secrets only through action inputs or environment variables.
