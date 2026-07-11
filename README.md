# postil-action

GitHub Action for [Postil](https://postil.dev): a low-noise AI review gate that stays
silent on clean PRs and fails a dedicated `postil/gate` check on real risk.

This is a thin composite action. All review logic lives in the
[`postil` CLI](https://github.com/postil-dev/postil-cli), installed at the commit SHA
you pin — the action contains no JavaScript and no Docker image.

## Usage

```yaml
name: review
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  postil:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: postil-dev/postil-action@v1
        with:
          cli-ref: 87f4bf08b63712d3600030a7c458f0b790cfc0d5   # postil-cli v0.1.1
          cli-release: v0.1.1          # optional: prebuilt binary (must match cli-ref)
          api-key: ${{ secrets.OPENROUTER_API_KEY }}
```

Set `timeout-minutes` on the job: a hung model endpoint or a stuck source build
should not tie up your runner queue indefinitely.

When `cli-release` is set, Linux runners fetch a verified prebuilt CLI when one
matches the runner. The action supports glibc and Alpine/musl runners with
`bash`, `curl`, `jq`, `tar`, and checksum tools available on `x86_64` and
`aarch64`/`arm64`; unsupported platforms fall back to building the CLI from the
pinned `cli-ref`.

> **Note:** there is no `@v1` tag yet — this action has not had a tagged
> release. Until one is published, pin the action to a reviewed commit SHA
> (`postil-dev/postil-action@<40-hex sha>`) instead of `@v1`. Pinning to a SHA
> is the recommended practice for third-party actions regardless; switch to
> `@v1` once the first tag ships.

The job fails when the gate fails (severity `error` findings by default). To require the
gate without failing this job, set `soft-fail: true` and mark the `postil/gate` check as
required in branch protection instead — that is the recommended setup: advisory comments
never block, the gate check does.

## Inputs

| Input | Required | Description |
|---|---|---|
| `cli-ref` | yes | Full 40-hex commit SHA of `postil-dev/postil-cli`. Tags/branches are rejected: they are mutable, and this binary posts to your PRs. |
| `cli-release` | no | Release tag for a prebuilt binary. Verified to point at `cli-ref` and checksum-checked; any mismatch falls back to building from source. |
| `api-key` | yes | Key for your model endpoint. Postil never proxies or marks up inference. |
| `api-base` | no | Any OpenAI-compatible endpoint (OpenRouter default; Ollama, vLLM, Azure OpenAI). |
| `model`, `model-cascade` | no | Model override and comma-separated fallbacks. |
| `fail-on` | no | `info`/`warn`/`error`/`never` — overrides `gate.failOn` from repo config. |
| `config` | no | Explicit config path; otherwise `.postil.yaml` > `.coderabbit.yaml`. |
| `pr` | no | PR number; defaults to the triggering `pull_request` event. |
| `since-sha`, `baseline` | no | Incremental re-review (see CLI docs). |
| `soft-fail` | no | Report findings without failing the job. |
| `sarif-path` | no | Write SARIF 2.1.0 here, for upload via `github/codeql-action/upload-sarif`. |
| `github-token` | no | Defaults to `github.token`. Needs `pull-requests: write` and `checks: write`. |

## Outputs

- `envelope-path`: path to the review envelope JSON.
- `gate-failing`: `true` when gate-level findings exist.
- `sarif-path`: path to the SARIF file, set only when `sarif-path` was requested and the file was written.

## Security notes

- **Never use `pull_request_target` with a checkout of the PR head.** Postil does not
  need the PR code checked out at all — it reads the diff via the API. If your workflow
  must use `pull_request_target` (e.g. to access secrets on forked PRs), do not add a
  `actions/checkout` of `github.event.pull_request.head.sha`, and never build or execute
  PR code in the same job that holds your API key.
- Pinning `cli-ref` to a full SHA means the reviewed binary cannot change underneath you.
- The default `github.token` is scoped to the repository and expires with the job.

## License

Apache-2.0.
