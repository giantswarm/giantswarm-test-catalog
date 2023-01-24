[![CircleCI](https://circleci.com/gh/giantswarm/external-secrets.svg?style=shield)](https://circleci.com/gh/giantswarm/external-secrets)

# external-secrets chart

Giant Swarm offers an `external-secrets` App which can be installed in workload clusters.
Here we define the `external-secrets` chart with its templates and default configuration.

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/advanced/gitops/#installing-managed-apps)
- [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app).
- By creating an [App resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/app-platform/getting-started/).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml
crds:
  createClusterExternalSecret: true
  createClusterSecretStore: true
```

### Templating

You can use the [official Giant Swarm kubectl plug-in](https://github.com/giantswarm/kubectl-gs/) to template the
App CR and related resources.

```shell
kubectl gs template app \
  --catalog giantswarm-catalog \
  --name external-secrets \
  --version 0.2.1 \
  --target-namespace org-example \
  --cluster-name abc123 \
  --user-configmap values.yaml
```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create
the App CR and ConfigMap directly.

Here is an example that would install the app to workload cluster `abc12`:

```yaml
# app.yaml
---
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  name: external-secrets
  namespace: abc123
spec:
  catalog: giantswarm-catalog
  kubeConfig:
    inCluster: false
  name: external-secrets
  namespace: org-example
  userConfig:
    configMap:
      name: external-secrets-userconfig-abc123
      namespace: abc123
  version: 0.2.1
```

```yaml
# user-values-configmap.yaml
---
apiVersion: v1
data:
  values: |+
    crds:
      createClusterExternalSecret: true
      createClusterSecretStore: true
kind: ConfigMap
metadata:
  name: external-secrets-userconfig-abc123
  namespace: abc123
```

See our [full reference on how to configure apps](https://docs.giantswarm.io/app-platform/app-configuration/) for more details.

## Credit

- https://github.com/external-secrets/external-secrets
