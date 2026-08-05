[![CircleCI](https://circleci.com/gh/giantswarm/csi-external-snapshotter-app.svg?style=shield)](https://circleci.com/gh/giantswarm/csi-external-snapshotter-app)

# CSI External Snapshotter

To be able to create volume snapshots with any CSI driver, we need to install CRDs and a snapshot
controller. Backup tools such as [Velero](https://velero.io/docs/main/csi/) rely on both being
present in the cluster.

The chart is generated from the upstream
[kubernetes-csi/external-snapshotter](https://github.com/kubernetes-csi/external-snapshotter)
release manifests:

- https://github.com/kubernetes-csi/external-snapshotter/tree/v8.6.0/client/config/crd
- https://github.com/kubernetes-csi/external-snapshotter/tree/v8.6.0/deploy/kubernetes/snapshot-controller

## Upgrading to a new upstream release

1. Bump `EXTERNAL_SNAPSHOTTER_VERSION` in [`Makefile.custom.mk`](Makefile.custom.mk).
2. Regenerate the chart:

   ```sh
   make all APPLICATION=csi-external-snapshotter-app
   ```

   This clones the upstream repo at that tag into `config/kustomize/input/`, runs the
   Giant Swarm `kustomize` overlay in `config/kustomize/`, and writes the result into
   `helm/csi-external-snapshotter-app/` — CRDs go to `files/` (applied by the `crd-install`
   hook job), everything else to `templates/`.
3. Bump `snapshotController.tag` in `values.yaml` and `appVersion` in `Chart.yaml` to the
   same version, review the diff, and add a `CHANGELOG.md` entry.

Anything hand-written lives in `templates/crd-install/`, `templates/defaults.yaml` and
`templates/_helpers.tpl`; those files are not touched by the generator.
