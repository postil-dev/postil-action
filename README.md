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
      - uses: postil-dev/postil-action@00442e2340edaa4a681955dbb25e20650ca1514c
        with:
          cli-ref: fbbe0891bc302a7644c6fb2232ee5f4af587894b
          cli-release: v0.7.1
          api-key: ${{ secrets.OPENROUTER_API_KEY }}
          model: ${{ vars.POSTIL_REVIEW_MODEL }}
```

Set the `POSTIL_REVIEW_MODEL` repository variable to a model qualified for your review profile. Pin both repositories to immutable commit SHAs. `cli-release` selects a prebuilt binary only when the release resolves to `cli-ref` and its checksum and tag-bound Sigstore signature verify. Source fallback accepts only the Postil maintainer or GitHub web-flow signing fingerprints and requires the runner's GPG configuration to resolve that key.

## Inputs

| Input | Required | Purpose |
| --- | --- | --- |
| `cli-ref` | yes | Full 40-character `postil-cli` commit SHA |
| `api-key` | yes | Model provider credential |
| `cli-release` | no | Matching signed release tag for a prebuilt Linux or macOS binary |
| `api-base` | no | Model endpoint, defaulting to the CLI provider endpoint |
| `api-format` | no | `openai-compatible` or `anthropic` |
| `endpoint-auth-header`, `endpoint-auth-value` | no | Paired additional private-gateway authentication |
| `allow-private-api-base` | no | Permit a local or private-network endpoint |
| `model` | no | Primary model; required unless trusted config supplies one |
| `model-cascade` | no | Comma-separated qualified fallback models |
| `fail-on` | no | `info`, `warn`, `error`, or `never` |
| `config` | no | Explicit Postil configuration path |
| `pr` | no | Pull-request number when not inferred from the event |
| `since-sha`, `baseline` | no | Incremental review inputs |
| `soft-fail` | no | Report findings without failing the job |
| `sarif-path` | no | Write SARIF 2.1.0 for code-scanning upload |
| `github-token` | no | Forge token, defaulting to `github.token` |

Outputs are `envelope-path`, `gate-failing`, and `sarif-path`.

The action does not select an unverified fallback model. Set `model` to a model qualified for your review profile, or provide trusted configuration with `config`. If resolution produces no model, Postil fails before contacting a provider. `envelope-path` points to the complete CLI envelope, including `reviewCoverage` when the review uses bounded source selection.

For the native Anthropic Messages API, set `api-format: anthropic` and its API base. Private gateways can use `endpoint-auth-header` and `endpoint-auth-value` for one additional credential; set both or neither. Set `allow-private-api-base: true` only for an endpoint whose network boundary you trust.

## Gate setup

Postil publishes an advisory `postil/review` check and a blocking `postil/gate` check. Mark only `postil/gate` as required in branch protection. `soft-fail: true` keeps the workflow job green, but does not change the dedicated gate check.

## Security

Do not combine `pull_request_target`, a checkout of untrusted pull-request code, and repository secrets. Postil can read a remote diff through the GitHub API and does not need to execute pull-request code.

The default `github.token` is repository-scoped and expires with the job. The model credential is passed directly to the configured provider.

Configuration, output formats, and forge behavior are documented in the [Postil CLI repository](https://github.com/postil-dev/postil-cli).

## License

Apache-2.0.
