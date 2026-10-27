[![CircleCI](https://circleci.com/gh/giantswarm/csi-external-snapshotter-app.svg?style=shield)](https://circleci.com/gh/giantswarm/csi-external-snapshotter-app)

# CSI External Snapshotter

To be able to create volume snapshots with any CSI driver, we need to install CRDs and a snapshot
controller. Backup tools such as [Velero](https://velero.io/docs/main/csi/) rely on both being
present in the cluster.

The chart is generated from the upstream
[kubernetes-csi/external-snapshotter](https://github.com/kubernetes-csi/external-snapshotter)
release manifests:

- `client/config/crd` → the `csi-external-snapshotter-crds` dependency chart
- `deploy/kubernetes/snapshot-controller` → the chart's top-level templates

## Layout

| Path | What it is |
| --- | --- |
| `vendir.yml` | The **only** place the upstream version is pinned. `vendir.lock.yml` records the resolved commit. |
| `sync/sync.sh` | Runs `vendir sync`, then each patch script in order. |
| `sync/kustomize/` | Overlay that turns the upstream snapshot-controller manifests into Helm templates — it injects `{{ .Release.Namespace }}` and the `{{ .Values.snapshotController.* }}` references, applies the Giant Swarm securityContext/affinity/resources patch, and splits the upstream files into one template per resource. |
| `sync/patches/*/` | One directory per concern (`chart`, `values`, `controller`, `templates`, `crds`). `manifests/` inside each holds the hand-written source of truth. |
| `helm/csi-external-snapshotter-app/` | **Generated.** Do not edit by hand — `make sync` overwrites it. |

Everything hand-written lives under `sync/patches/templates/manifests/`: `_helpers.tpl`,
`gs/defaults.yaml`, `gs/network-policies/` and `crd-adopt/`. Generated files sit at the top level of
`templates/`, which is why the hand-written ones are kept in subdirectories.

## CRDs

The CRDs ship as the `csi-external-snapshotter-crds` dependency chart, as ordinary templates rather
than in a `crds/` directory, so Helm renders, diffs and **upgrades** them like any other resource.
They carry `helm.sh/resource-policy: keep`, so `helm uninstall` leaves them (and therefore every
`VolumeSnapshot` in the cluster) in place. `crds.install: false` disables the dependency via its
`condition`.

Chart versions up to 0.3.x applied the CRDs with a `kubectl apply` hook job instead, which left them
without Helm ownership metadata. The `crd-adopt` pre-install/pre-upgrade hook hands that ownership
over on the first upgrade; it is a no-op on a fresh install, and can be dropped once every cluster
has been through a 0.4.0 or later release.

## Upgrading to a new upstream release

Requires `vendir`, `kustomize` and `yq` on `$PATH`.

1. Bump `ref` under `directories[0].contents[0].git` in [`vendir.yml`](vendir.yml).
2. Regenerate the chart:

   ```sh
   make sync
   ```

   This resolves and vendors upstream into `vendor/` (gitignored), then rewrites
   `helm/csi-external-snapshotter-app/`. `appVersion`, the
   `io.giantswarm.application.upstream-chart-version` annotation and
   `snapshotController.tag` are all derived from the `ref` — there is nothing else to bump. The CRD
   dependency chart's version is bumped to the upstream version only when the CRDs actually changed.
3. Review `git diff helm/` (the sync prints it) and add a `CHANGELOG.md` entry.

The chart's own `version` is left alone by the sync and is replaced at build time anyway
(`replace-chart-version-with-git` in [`.abs/main.yaml`](.abs/main.yaml)); the committed value only
needs to be valid semver so that `helm lint` and `helm template` work locally.
