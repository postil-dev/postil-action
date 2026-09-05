# Postil Action

Run [Postil](https://postil.dev) as a pull-request review gate in GitHub Actions. Clean changes receive no review comment. Gate-level findings fail the job and the `postil/gate` check.

## Quick start

Pin the action and CLI to immutable commits. Supply the model credential through GitHub Actions secrets.

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
          cli-ref: dcf4e34c4804d4bf64705ab6367c883cea23b33a
          cli-release: v0.7.2
          api-key: ${{ secrets.OPENROUTER_API_KEY }}
          model: ${{ vars.POSTIL_REVIEW_MODEL }}
```

`cli-ref` is required and must be a full 40-character commit SHA. `cli-release` is optional: the action uses its binary only when the release resolves to `cli-ref` and its checksum and tag-bound Sigstore signature verify. Otherwise it builds the pinned CLI source.

Set `model` to a model qualified for the repository's review profile, or provide trusted Postil configuration. If neither supplies a model, the action fails before contacting a provider.

## Configuration and safety

The action accepts model endpoint, model cascade, severity, incremental-review, SARIF, and configuration-path inputs. The complete input and output contract is in [action.yml](action.yml); CLI configuration and provider setup are documented in the [Postil CLI repository](https://github.com/postil-dev/postil-cli).

Postil publishes advisory `postil/review` and blocking `postil/gate` checks. Require only `postil/gate` in branch protection. `soft-fail: true` keeps the workflow job green but does not change the dedicated gate check.

Do not combine `pull_request_target`, a checkout of untrusted pull-request code, and repository secrets. Postil reads the pull-request diff through the GitHub API and does not need to execute pull-request code.

## License

Apache-2.0.
