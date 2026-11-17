# mcp-grafana-acme

Argo CD-manifested deployment of the [mcp-grafana](https://github.com/srclosson/mcp-grafana)
MCP server for the `stephgen-medussa.observe.devopsnow.cloud` Grafana instance,
running in the `mcp-grafana-acme` namespace of the `fe-stackgen-run` cluster.

## Layout

- `deploy/` — the manifests Argo CD syncs (source path of the Application)
  - `mcp-grafana-acme.yaml` — the MCP server Deployment + Service
  - `mcp-grafana-acme-auth.yaml` — the nginx bearer-token gate for callers
    (credential check only, it proxies nothing), + Service + ConfigMap
  - `sealed-secret.yaml` — the `mcp-grafana-acme-secrets` secret
    (`GRAFANA_SERVICE_ACCOUNT_TOKEN`, `MCP_BEARER_TOKEN`), sealed with
    [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) for this
    namespace's key. To rotate a value: edit it locally, re-run
    `kubeseal < new-secret.yaml > deploy/sealed-secret.yaml`, commit.
- `argocd/application.yaml` — the Application CR; applied by hand once into
  the `argocd` namespace, then Argo CD manages everything under `deploy/`.

## Notes

- The image is the personal build `docker.io/srclosson/mcp-grafana:latest`
  (imagePullPolicy Always, so every rollout pulls whatever was pushed last).
- The TLS secret `mcp-grafana-acme-civo-tls` is managed by cert-manager and
  deliberately lives outside this repo.

## Roadmap

- [ ] Switch authentication to a shared user across Grafana orgs
      (`GRAFANA_USERNAME`/`GRAFANA_PASSWORD`) so `--dynamic-multi-org` can
      target orgs per call; drop `--disable-proxied` so datasource discovery
      spans every org the user can access.
