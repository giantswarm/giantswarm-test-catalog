[![CircleCI](https://circleci.com/gh/giantswarm/linkerd-multicluster-link.svg?style=shield)](https://circleci.com/gh/giantswarm/linkerd-multicluster-link)

[Read me after cloning this template (GS staff only)](https://intranet.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# linkerd-multicluster-link chart

Giant Swarm offers a linkerd-multicluster-link App which can be installed in workload clusters.
Here we define the linkerd-multicluster-link chart with its templates and default configuration.

**What is this app?**

**Why did we add it?**

**Who can use it?**

## Installing

There are several ways to install this app onto a workload cluster.

- [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app).
- By creating an [App resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/app-platform/getting-started/).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml
targetClusterName: "<cluster-id>"
target:
  enabled: true
  api: "https://api.<cluster-id>.gigantic.io:443"
  CA: "<cluster-ca-pem>"
  token: "<my-token>"
  gateway:
    ip: "<gateway-ip-of-target-cluster>"
    port: "4143"
    portProbe: "4191"
```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create
the App CR and ConfigMap directly.

Here is an example that would install the app to
workload cluster `abc12`:

```yaml
# appCR.yaml
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  name: linkerd-multicluster-link
  namespace: <org_namespace>
spec:
  catalog: giantswarm
  kubeConfig:
    inCluster: false
  name: linkerd-multicluster-link
  namespace: linkerd-multicluster
  namespaceConfig:
    labels:
      linkerd.io/extension: multicluster
  userConfig:
    configMap:
      name: linkerd-multicluster-link-userconfig-<your-cluster-id>
      namespace: <your-cluster-id>
  version: 0.9.0
```

```yaml
# user-values-configmap.yaml
apiVersion: v1
data:
  values: |
    targetClusterName: "<cluster-id>"
    target:
      enabled: true
      api: "https://api.<cluster-id>.gigantic.io:443"
      CA: "<cluster-ca-pem>"
      token: "<my-token>"
      gateway:
        ip: "<gateway-ip-of-target-cluster>"
        port: "4143"
        portProbe: "4191"
kind: ConfigMap
metadata:
  name: linkerd-multicluster-link-userconfig-<your-cluster-id>
  namespace: <your-cluster-id>
```

See our [full reference on how to configure apps](https://docs.giantswarm.io/app-platform/app-configuration/) for more details.

## Breaking changes

- In version 0.9.0 and above the value `target.name` has been deprecated and replaced with `targetClusterName` to follow the upstream chart behavior.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- _add release version_

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- _add limitation_

## Credit

- linkerd-multicluster-link
