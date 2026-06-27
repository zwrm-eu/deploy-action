# Deploy to ZWRM — GitHub Action

Build and deploy an application to a [ZWRM](https://zwrm.io) control plane straight from your
CI/CD pipeline. Wraps the `zwrm deploy` command.

The build happens **server-side**: the action uploads your build context to the control plane,
which builds the image and launches your VMs. That means the runner needs **no Docker, no
buildx, no KVM** — just this action and your `ZWRM_API_TOKEN`.

```yaml
- uses: zwrm-eu/deploy-action@v1
  with:
    api-token: ${{ secrets.ZWRM_API_TOKEN }}
    org-id: my-org
```

## Quick start

1. **Create an API token** for CI (do this once, from a machine where you're logged in):

   ```bash
   zwrm auth tokens create --name github-actions
   ```

   Copy the token — it's shown only once.

2. **Add it as a secret** in your repository: *Settings → Secrets and variables → Actions →
   New repository secret*, named `ZWRM_API_TOKEN`.

3. **Add a workflow** at `.github/workflows/deploy.yml`:

   ```yaml
   name: Deploy
   on:
     push:
       branches: [main]

   jobs:
     deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: zwrm-eu/deploy-action@v1
           with:
             api-token: ${{ secrets.ZWRM_API_TOKEN }}
             org-id: my-org
   ```

   `app` and `context` default to your repo's `zwrm.toml`, so that's the whole integration.

## Inputs

| Input               | Required | Default                            | Description                                                                                  |
| ------------------- | -------- | ---------------------------------- | -------------------------------------------------------------------------------------------- |
| `api-token`         | **yes**  | —                                  | ZWRM API token. Store as a secret. Maps to `ZWRM_API_TOKEN`.                                  |
| `org-id`            | no       | `""`                               | Organization slug or ID. Maps to `ZWRM_ORG_ID`. Optional if the token resolves to one org.   |
| `app`               | no       | from `zwrm.toml`                   | Application name. Overrides `app.name` in `zwrm.toml`.                                        |
| `control-plane-url` | no       | `https://zwrm.io`                  | Control plane API URL.                                                                        |
| `working-directory` | no       | `.`                                | Directory containing `zwrm.toml`. The deploy runs from here.                                  |
| `context`           | no       | `working-directory`                | Build context directory.                                                                      |
| `force-build`       | no       | `false`                            | Force a rebuild, bypassing the build cache.                                                   |
| `cli-version`       | no       | `latest`                           | zwrm CLI version: a pinned tag like `v0.11.13`, or `latest`.                                  |
| `release-base-url`  | no       | `https://releases.zwrm.eu/zwrmd`   | Release server base URL. Override for canary channels or self-hosted mirrors.                 |

There are no outputs yet (the CLI streams build logs to the step log and fails the step on a
failed deploy). Machine-readable outputs — deployment ID, app URLs — are planned; see
[Roadmap](#roadmap).

## Examples

### Pin the CLI version (recommended for reproducible CI)

`latest` is convenient but moves under you. Pin a version for deterministic builds:

```yaml
- uses: zwrm-eu/deploy-action@v1
  with:
    api-token: ${{ secrets.ZWRM_API_TOKEN }}
    org-id: my-org
    cli-version: v0.11.13
```

### Monorepo — deploy a subdirectory

```yaml
- uses: zwrm-eu/deploy-action@v1
  with:
    api-token: ${{ secrets.ZWRM_API_TOKEN }}
    org-id: my-org
    working-directory: services/api
```

### Override app name and force a clean build

```yaml
- uses: zwrm-eu/deploy-action@v1
  with:
    api-token: ${{ secrets.ZWRM_API_TOKEN }}
    org-id: my-org
    app: my-api-staging
    force-build: "true"
```

### Self-hosted / custom control plane

```yaml
- uses: zwrm-eu/deploy-action@v1
  with:
    api-token: ${{ secrets.ZWRM_API_TOKEN }}
    org-id: my-org
    control-plane-url: https://cp.example.com:8080
```

## How it works

1. **Install** — detects the runner OS/arch, downloads the matching `zwrm` binary from the
   release server, verifies it against `checksums.txt`, and puts it on `PATH`.
2. **Deploy** — runs `zwrm deploy` from `working-directory`, passing auth via `ZWRM_API_TOKEN`
   / `ZWRM_ORG_ID`. The CLI reads `zwrm.toml`, creates the app if it doesn't exist, uploads the
   build context, and streams build logs until the deployment completes. A failed or rolled-back
   deployment exits non-zero and fails the job.

Supported runners: `ubuntu-*` and `macos-*`, on X64 and ARM64.

## Versioning

This action follows the GitHub Marketplace convention of a moving major tag plus immutable
release tags:

- `zwrm-eu/deploy-action@v1` — tracks the latest `v1.x.y` (recommended).
- `zwrm-eu/deploy-action@v1.2.3` — pins an exact release.

Note that `cli-version` (the zwrm CLI) is versioned independently of the action tag.

## Security

- Pass the token only through the `api-token` input wired to a secret. Never hard-code it.
- Downloaded binaries are checksum-verified against the release server's `checksums.txt`.
- For maximum supply-chain hardening, pin the action to a commit SHA and set an explicit
  `cli-version`.

## Roadmap

- Step outputs: deployment ID, app URLs, health status.
- Optional wait/no-wait and timeout controls.

## License

[MIT](./LICENSE)
