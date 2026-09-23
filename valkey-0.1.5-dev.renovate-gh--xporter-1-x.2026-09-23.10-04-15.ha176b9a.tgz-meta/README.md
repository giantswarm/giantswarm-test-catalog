[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/valkey-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/valkey-app/tree/main)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# valkey chart

Giant Swarm offers a valkey App which can be installed in workload clusters.
Here, we define the valkey chart with its templates and default configuration.

**What is this app?**

[Valkey](https://valkey.io/) is an open-source, high-performance key-value datastore that serves as a drop-in replacement for Redis. It supports a wide variety of data structures such as strings, hashes, lists, sets, sorted sets, and more.

**Why did we add it?**

Valkey is a community-driven fork of Redis, maintained by the Linux Foundation. It provides an alternative to Redis following Redis Labs' license changes.

**Who can use it?**

Anyone needing an in-memory data store for caching, message queuing, or session management.

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)
- By creating an [App resource](https://docs.giantswarm.io/reference/platform-api/crd/apps.application.giantswarm.io) using the platform API as explained in [Getting started with App Platform](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml
valkey:
  replicaCount: 3
  dataStorage:
    enabled: true
    requestedSize: 10Gi
```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create the App CR and ConfigMap directly.

Here is an example that would install the app to workload cluster `abc12`:

```yaml
# appCR.yaml
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  name: valkey
  namespace: abc12
spec:
  catalog: giantswarm
  kubeConfig:
    inCluster: false
  name: valkey
  namespace: valkey
  version: 0.1.0
```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- CAPI workload clusters

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- This chart deploys a single Valkey instance by default. For high availability, increase `replicaCount`.

## Credit

- [Valkey Helm Chart](https://github.com/valkey-io/valkey-helm)
