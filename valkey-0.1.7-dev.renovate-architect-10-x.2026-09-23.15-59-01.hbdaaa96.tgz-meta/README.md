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

### Rolling on credential rotation

Valkey reads its users' passwords once, when the pod starts. The subchart's pod template carries a checksum over the credentials, so a rotation restarts the pod: `checksum/auth-secret`, the SHA-256 of the chart-rendered `<fullname>-auth` Secret for inline passwords, and `checksum/users-secret`, the value of `valkey.auth.usersExistingSecretChecksum` verbatim for a Secret the chart does not render. Whoever rotates `valkey.auth.usersExistingSecret` changes that value in the same change (a hash over the new data, a counter, a date); a Flux `HelmRelease` can feed it from a Secret or ConfigMap key through `valuesFrom` with `targetPath`. See the subchart's [README](helm/valkey/charts/valkey/README.md#rolling-on-credential-rotation).

```yaml
valkey:
  auth:
    enabled: true
    usersExistingSecret: "my-valkey-users"
    usersExistingSecretChecksum: "2026-09-23-1"
    aclUsers:
      default:
        permissions: "~* &* +@all"
```

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- CAPI workload clusters

## The vendored chart

`helm/valkey/charts/valkey` is upstream [valkey-io/valkey-helm](https://github.com/valkey-io/valkey-helm)'s `valkey` chart at the version `vendir.yml` pins, synced with `make update-chart` (`vendir sync`), which overwrites the directory. These changes are carried on top of it and have to be reapplied after a sync until upstream carries them:

- The metrics exporter authenticates against Valkey when `auth.enabled` is set (`REDIS_PASSWORD` from `usersExistingSecret`/`passwordKey` or the chart's `<fullname>-auth` Secret), backported from a later upstream version.
- The pod template's `checksum/auth-secret` and `checksum/users-secret` annotations and the `auth.usersExistingSecretChecksum` value (upstream: [valkey-io/valkey-helm#128](https://github.com/valkey-io/valkey-helm/pull/128) covers the rendered Secret's half), and the pod annotations rendered as annotations also without `podAnnotations` (upstream `main` has this).

`make helm-test` lints the chart and runs its unit tests (`helm/valkey/charts/valkey/tests/`); the `chart-test` CircleCI job runs it on every branch and tag.

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- This chart deploys a single Valkey instance by default. For high availability, increase `replicaCount`.

## Credit

- [Valkey Helm Chart](https://github.com/valkey-io/valkey-helm)
