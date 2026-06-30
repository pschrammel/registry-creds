# AGENTS.md — registry-creds

Compact guidance for OpenCode sessions working in this repo.

## What this is

Single Go binary that runs inside a Kubernetes cluster and refreshes registry
`ImagePullSecrets` across **all namespaces** (default excludes `kube-system`).
Supports AWS ECR, GCR, Docker Private Registry, and Azure Container Registry.

## Build & test

- **Build**: `make build` — cross-compiles a static linux/amd64 binary (`registry-creds`).
- **Test**: `make test` — runs `go test -v`.
- **Docker image**: `make container` (depends on `make build`).
- **Clean**: `make clean` removes the compiled binary.

CI (`.travis.yml`) runs `make test` then `make build` on Go 1.8.

## Dependency management

- Managed via Go modules (`go.mod` / `go.sum`).
- `manifest.json` and `lock.json` are historical artifacts from the pre-Go-modules era.

## Running locally

```bash
go run ./main.go --kubecfg-file=<pathToKubecfgFile>
```

If `--kubecfg-file` is omitted the app tries in-cluster config.

## Project layout

| Path | Purpose |
|------|---------|
| `main.go` | Entrypoint, controller logic, token generation for 4 providers |
| `main_test.go` | Unit tests with hand-rolled fakes for k8s + cloud clients |
| `k8sutil/k8sutil.go` | Kubernetes client wrapper and namespace watcher |
| `k8s/` | Deployment and Secret manifests |

## Code quirks

- **Global flag variables**: CLI args are stored in package-level `flag` variables (e.g. `argAWSRegion`, `argGCRURL`). Tests mutate these globals.
- **Global retry config**: `RetryCfg` and backoff timers are globals; tests call `disableRetries()` / `enableShortRetries()` to avoid long waits.
- **Test fakes are extensive**: `fakeKubeClient`, `fakeEcrClient`, `fakeGcrClient`, `fakeDprClient`, `fakeACRClient` are all defined in `main_test.go`.
- Uses **k8s.io/client-go v0.36.2** and **AWS SDK for Go v2**.

## Constraints

- `--skip-kube-system` defaults to `true`; the app will not attach secrets to the `kube-system` namespace.
- AWS account IDs come from the `awsaccount` env var (comma-separated) or the `--aws-account` flag.
