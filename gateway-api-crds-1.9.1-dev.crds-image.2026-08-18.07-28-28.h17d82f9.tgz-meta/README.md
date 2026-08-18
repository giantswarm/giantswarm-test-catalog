[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/gateway-api-crds-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/gateway-api-crds-app/tree/main)

# gateway-api-crds chart

This App installs the [Gateway API](https://gateway-api.sigs.k8s.io/) custom resource definitions

See our [full reference on how to configure apps](https://docs.giantswarm.io/getting-started/app-platform/app-configuration/) for more details.

Which CRDs get installed, and from which channel, is controlled by the `install` values. Each
key is a CRD's plural name and its value is `standard`, `experimental`, or `""` to skip it.

## How the CRDs are installed

The CRDs are too large to fit in Helm's release Secret, so the chart does not render them.
They are baked into the `giantswarm/gateway-api-crds` image under `/crds/<channel>/<plural>.yaml`,
and a `pre-install`/`pre-upgrade` hook Job applies the selected ones with
`kubectl apply --server-side`. The image tag defaults to the chart version and can be
overridden with `crds.image.tag`.

Consequences worth knowing:

- The CRDs are not Helm-managed, so `helm uninstall` leaves them, and the custom resources
  they hold, in place. Removing them is a deliberate manual step.
- Nothing is ever pruned. Setting a key to `""` stops that CRD from being applied but does
  not remove it from the cluster.
- `helm rollback` does not revert CRDs.

To regenerate `crds/` and the chart templates after changing `vendir.yml`, run `./sync/sync.sh`.

## Credit

- https://github.com/kubernetes-sigs/gateway-api
