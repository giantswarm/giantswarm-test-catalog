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

<details>
<summary>Legacy: sample App CR and ConfigMap for the management cluster</summary>

If you still use the App Platform, you could create the App CR and ConfigMap directly in the
management cluster. Here is an example that would install the app to workload cluster `abc123`
of organization `example`:

```yaml
# app.yaml
---
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  labels:
    giantswarm.io/cluster: abc123
  name: external-secrets
  namespace: org-example
spec:
  catalog: giantswarm
  kubeConfig:
    inCluster: false
  name: external-secrets
  namespace: external-secrets
  userConfig:
    configMap:
      name: external-secrets-userconfig-abc123
      namespace: org-example
  version: 2.11.0
```

```yaml
# user-values-configmap.yaml
---
apiVersion: v1
data:
  values: |
    crds:
      createClusterGenerator: false
    processClusterGenerator: false
kind: ConfigMap
metadata:
  name: external-secrets-userconfig-abc123
  namespace: org-example
```

</details>

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Credit

- https://github.com/external-secrets/external-secrets
