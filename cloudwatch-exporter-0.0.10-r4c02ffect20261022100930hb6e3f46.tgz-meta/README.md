[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/cloudwatch-exporter-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/cloudwatch-exporter-app/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/cloudwatch-exporter-app/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/cloudwatch-exporter-app)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# cloudwatch-exporter-app chart

Giant Swarm offers a cloudwatch-exporter App which can be installed in workload clusters.
Here, we define the cloudwatch-exporter chart with its templates and default configuration.

**What is this app?**

[Yet Another CloudWatch Exporter (yace)](https://github.com/prometheus-community/yet-another-cloudwatch-exporter) is a Prometheus exporter for AWS CloudWatch metrics. This chart is a Giant Swarm wrapper around the upstream `prometheus-yet-another-cloudwatch-exporter` chart.

**Why did we add it?**

To expose AWS CloudWatch metrics to Prometheus on Giant Swarm clusters through the app platform.

**Who can use it?**

Anyone running on AWS who needs CloudWatch metrics scraped into their monitoring stack.

## Upstream sync

This chart is not hand-written; it is generated from the upstream chart with `vendir` and customized with a values patch. To update or re-sync:

```bash
# edit the chart version in vendir.yml, then:
./sync/sync.sh
```

`sync/sync.sh` runs `vendir sync` to fetch the upstream chart into `vendor/` and `helm/cloudwatch-exporter/`, applies the patch in `sync/patches/values/`, and records the resulting divergences from upstream under `diffs/`. Customizations live only in `sync/patches/` and `helm/cloudwatch-exporter/Chart.yaml`; never edit the generated chart files directly.

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

## Credit

- [prometheus-community/helm-charts](https://github.com/prometheus-community/helm-charts)
