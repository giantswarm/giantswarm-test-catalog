[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/external-secrets/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/external-secrets/tree/main)

# external-secrets chart

Giant Swarm offers an `external-secrets` App which can be installed in workload clusters.
Here we define the `external-secrets` chart with its templates and default configuration.

## Installing

This app is a cluster singleton: install it only once per workload cluster.

The recommended way to install this app onto a workload cluster is a Flux `HelmRelease`:

- [Deploying an application via a Flux HelmRelease](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/deploy-app-helmrelease/)
- [Adding a HelmRelease via GitOps](https://docs.giantswarm.io/tutorials/continuous-deployment/helm-releases/add-helmrelease/)

As a fallback, the legacy App Platform ([deprecated](https://docs.giantswarm.io/overview/fleet-management/app-management/app-platform-deprecation/)) is still supported:

- By creating an [App resource](https://docs.giantswarm.io/reference/platform-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Deploying an app (legacy App CR)](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/deploy-app/).
- [Using GitOps to add an App CR](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)

## Configuring

### values.yaml

This is an example of a values file that disables the `ClusterGenerator` CRD and its controller.
Both settings are needed together. See [`values.yaml`](helm/external-secrets/values.yaml) for all options.

```yaml
# values.yaml
crds:
  createClusterGenerator: false
processClusterGenerator: false
```

### Deploying with kubectl-gs

You can use the [official Giant Swarm kubectl plug-in](https://github.com/giantswarm/kubectl-gs/) to create the
Flux `OCIRepository` and `HelmRelease` in the management cluster.

Here is an example that would install the app to workload cluster `abc123` of organization `example`:

```shell
kubectl gs deploy chart \
  --chart-name external-secrets \
  --version 2.11.0 \
  --organization example \
  --target-cluster abc123 \
  --target-namespace external-secrets \
  --values-file values.yaml
```

Add `--dry-run` to print the manifests without applying them.

See the [`kubectl gs deploy chart` reference](https://docs.giantswarm.io/reference/kubectl-gs/deploy-chart/) for all options.

## Credit

- https://github.com/external-secrets/external-secrets
