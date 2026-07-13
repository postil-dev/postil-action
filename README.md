# Postil Action

Run [Postil](https://postil.dev) as a quiet pull-request review gate in GitHub Actions. Clean changes receive no review comment. Gate-level findings fail the job and the separate `postil/gate` check.

This composite action installs a pinned, signed [`postil` CLI](https://github.com/postil-dev/postil-cli) build. The action contains no separate review engine.

## Usage

```yaml
name: postil

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  review:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: postil-dev/postil-action@9c8cf2c2f650f5946774c6d01626da507836b418
        with:
          cli-ref: dd1381381d4791475a277333837394f2f5032d27
          cli-release: v0.6.0
          api-key: ${{ secrets.OPENROUTER_API_KEY }}
```

Pin both repositories to immutable commit SHAs. `cli-release` selects a prebuilt binary only when the release resolves to `cli-ref` and its checksum and Sigstore signature verify. Other cases build the signed CLI commit from source.

## Inputs

| Input | Required | Purpose |
| --- | --- | --- |
| `cli-ref` | yes | Full 40-character `postil-cli` commit SHA |
| `api-key` | yes | Model provider credential |
| `cli-release` | no | Matching signed release tag for a prebuilt Linux binary |
| `api-base` | no | OpenAI-compatible model endpoint |
| `model` | no | Primary model override |
| `model-cascade` | no | Comma-separated fallback models |
| `fail-on` | no | `info`, `warn`, `error`, or `never` |
| `config` | no | Explicit Postil configuration path |
| `pr` | no | Pull-request number when not inferred from the event |
| `since-sha`, `baseline` | no | Incremental review inputs |
| `soft-fail` | no | Report findings without failing the job |
| `sarif-path` | no | Write SARIF 2.1.0 for code-scanning upload |
| `github-token` | no | Forge token, defaulting to `github.token` |

Outputs are `envelope-path`, `gate-failing`, and `sarif-path`.

## Gate setup

Postil publishes an advisory `postil/review` check and a blocking `postil/gate` check. Mark only `postil/gate` as required in branch protection. `soft-fail: true` keeps the workflow job green, but does not change the dedicated gate check.

## Security

Do not combine `pull_request_target`, a checkout of untrusted pull-request code, and repository secrets. Postil can read a remote diff through the GitHub API and does not need to execute pull-request code.

The default `github.token` is repository-scoped and expires with the job. The model credential is passed directly to the configured provider.

Configuration, output formats, and forge behavior are documented in the [Postil CLI repository](https://github.com/postil-dev/postil-cli).

## License

Apache-2.0.
