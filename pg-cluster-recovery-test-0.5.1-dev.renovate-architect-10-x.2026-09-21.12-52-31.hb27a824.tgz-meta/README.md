[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/pg-cluster-recovery-test/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/pg-cluster-recovery-test/tree/main)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# pg-cluster-recovery-test chart

Giant Swarm offers a pg-cluster-recovery-test App which can be installed in workload clusters.
Here, we define the pg-cluster-recovery-test chart with its templates and default configuration.

**What is this app?**

This app tests [CloudNativePG](https://github.com/giantswarm/cloudnative-pg-app/) database backups.

It relies on a `CronJob` that runs a [test-script](https://github.com/giantswarm/pg-cluster-recovery-test/blob/main/helm/pg-cluster-recovery-test/templates/pg-cluster-configmaps.yaml).

This test script will create a new CNPG cluster, bootstrap it from some backups, and check that the creation was successful.

This helps validate that backups are working.

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)
- By creating an [App resource](https://docs.giantswarm.io/reference/platform-api/crd/apps.application.giantswarm.io) using the platform API as explained in [Getting started with App Platform](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/).
- Or as a dependency of another helm chart, like we did in [grafana-app's helm chart](https://github.com/giantswarm/grafana-app/blob/main/helm/grafana/Chart.yaml).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml

```

### Sample App CR and ConfigMap for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create the App CR and ConfigMap directly.

Here is an example that would install the app to workload cluster `abc12`:

```yaml
# appCR.yaml

```

```yaml
# user-values-configmap.yaml

```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- _add release version_

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- _add limitation_

## Credit

- {APP HELM REPOSITORY}
