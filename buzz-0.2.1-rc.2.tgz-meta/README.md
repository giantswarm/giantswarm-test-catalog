[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/buzz/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/buzz/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/buzz/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/buzz)

# buzz chart

[Buzz](https://github.com/block/buzz) is a self-hostable workspace where people and AI agents share channels,
repositories, workflows and huddles. Under the hood it is a Nostr relay: one Rust binary serving WebSocket, REST
and the web UI, backed by PostgreSQL, Redis and S3-compatible object storage.

This repository packages upstream's Helm chart for the Giant Swarm app platform and publishes it to the
`giantswarm` catalog and `oci://gsoci.azurecr.io/charts/giantswarm/buzz`.

## How the chart is built

- `helm/buzz/templates` is upstream's `deploy/charts/buzz/templates`, vendored by
  [vendir](https://carvel.dev/vendir/) at the ref in `vendir.yml` with the upstream-ready patches in `patches/`
  applied on top (`make sync-chart` does both); the bundled Postgres and Redis subcharts (CloudPirates) are
  vendored into `helm/buzz/charts`.
- `helm/buzz/tests` holds the chart's helm-unittest suites; `make helm-test` runs them with `helm lint`, as the
  `chart-test` CI job does.
- `helm/buzz/values.yaml` is upstream's `values.yaml` with every image on `gsoci.azurecr.io`, where
  [retagger](https://github.com/giantswarm/retagger) mirrors them (`ghcr.io/block/buzz`, `ghcr.io/block/buzz-minio`,
  `postgres`, `redis`), and `@schema` annotations for the generated `values.schema.json`.

To move to a new upstream release: bump `ref` in `vendir.yml` (and the subchart versions if upstream's
`Chart.yaml` changed them), run `make sync-chart` (drop a patch upstream has taken), set `appVersion` in `helm/buzz/Chart.yaml` to the relay version,
carry upstream's `values.yaml` changes over, and run `devctl gen precommit --language generic --repo-name buzz
--flavors helmchart` to regenerate the schema. The relay tag has to be mirrored on gsoci first.

## Installing

Two profiles, as upstream documents them:

- **Production**: external PostgreSQL, Redis and S3 (`externalPostgresql`, `externalRedis`, `s3`), secrets in
  `secrets.existingSecret`. When only the S3 secret key lives in a Secret of its own, `s3.existingSecret`
  (`name`, `key`, default key `BUZZ_S3_SECRET_KEY`) points the relay at it and keeps it out of the
  chart-managed Secret.
- **Quickstart** (evaluation): `postgresql.enabled`, `redis.enabled` and `minio.enabled` bring the services up
  in-cluster and the chart generates the relay secrets. `helm/buzz/ci/quickstart-values.yaml` is that profile.
  The bundled MinIO image is `linux/amd64` only.

`relayUrl` (the public `wss://` URL) is always required, and `ownerPubkey` while
`relay.requireRelayMembership` is true. A Flux `HelmRelease`:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: OCIRepository
metadata:
  name: buzz
  namespace: buzz
spec:
  interval: 10m
  url: oci://gsoci.azurecr.io/charts/giantswarm/buzz
  ref:
    semver: ">=0.1.0 <1.0.0"
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: buzz
  namespace: buzz
spec:
  interval: 10m
  chartRef:
    kind: OCIRepository
    name: buzz
  values:
    relayUrl: wss://buzz.example.com
    ownerPubkey: "<64-char hex Nostr pubkey>"
```

The relay answers on port 3000 (WebSocket, REST, web UI) and its health endpoints on 8080
(`/_liveness`, `/_readiness`).

## Credit

- https://github.com/block/buzz (Apache-2.0)
