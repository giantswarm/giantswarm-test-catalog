[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/kueue/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/kueue/tree/main)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# kueue chart

Giant Swarm offers a kueue App which can be installed in workload clusters.
Here, we define the kueue chart with its templates and default configuration.

**What is this app?**

Kueue is a set of APIs and controllers for job queueing. It is a job-level manager that decides when a job should be admitted to start (as in pods can be created) and when it should stop (as in active pods should be deleted).

**Why did we add it?**

Kueue provides advanced job scheduling and resource management capabilities for Kubernetes workloads, enabling better resource utilization and fair sharing across different teams and applications.

**Who can use it?**

Any Giant Swarm customer who needs advanced job scheduling, resource quotas, and workload prioritization in their Kubernetes clusters.
## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)
- By creating an [App resource](https://docs.giantswarm.io/reference/platform-api/crd/apps.application.giantswarm.io) using the platform API as explained in [Getting started with App Platform](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/).

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

## Upstream Synchronization

This chart is based on the official [Kueue Helm chart](https://github.com/kubernetes-sigs/kueue) from the Kubernetes SIGs organization. We use [Carvel vendir](https://carvel.dev/vendir/) to synchronize with upstream changes.

### Syncing with Upstream

To pull the latest changes from the upstream Kueue chart:

```bash
# Sync upstream changes
./sync-upstream.sh

# Or use vendir directly
vendir sync
```

The upstream content will be available in:

- `upstream/kueue/` - The official Kueue Helm chart
- `upstream/docs/` - Documentation and examples from the upstream repository

### Comparing Changes

After syncing, you can compare the upstream chart with our customized version:

```bash
# Compare Chart.yaml files
diff -u upstream/kueue/Chart.yaml helm/kueue/Chart.yaml

# Compare values.yaml files
diff -u upstream/kueue/values.yaml helm/kueue/values.yaml

# Compare templates directory
diff -r upstream/kueue/templates helm/kueue/templates
```

### Configuration

The vendir configuration is defined in `vendir.yml` and automatically:

- Fetches the latest stable releases (>=0.14.0)
- Includes release candidates and beta versions
- Excludes unnecessary files like README.md and .helmignore
- Maintains a lock file (`vendir.lock.yml`) for reproducible builds

## Credit

- [Official Kueue Project](https://github.com/kubernetes-sigs/kueue)
- [Kueue Documentation](https://kueue.sigs.k8s.io/)
