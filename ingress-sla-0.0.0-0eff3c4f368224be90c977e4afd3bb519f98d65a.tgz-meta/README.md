[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/ingress-sla-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/ingress-sla-app/tree/main)

# ingress-sla-app chart

This app is required to enable SLA monitoring for Ingress in Giant Swarm cluster.

> **NOTE:** It requires [prometheus-blackbox-exporter-app](https://github.com/giantswarm/prometheus-blackbox-exporter-app) to be installed in the cluster.

## Installing

> **NOTE:** It requires [prometheus-blackbox-exporter-app](https://github.com/giantswarm/prometheus-blackbox-exporter-app) to be installed in the cluster.

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/advanced/gitops/apps/)
- [Using our web interface](https://docs.giantswarm.io/platform-overview/web-interface/app-platform/#installing-an-app).
- By creating an [App resource](https://docs.giantswarm.io/use-the-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/getting-started/app-platform/).

## Configuring

Configuration is explained in the comments of [`values.yaml`](helm/ingress-sla-app/values.yaml) file.

A minimal working configuration can look like this:

```yaml
global:
  checks:
    - ingressClass: nginx
```

It assumes that:

- Giant Swarm provided base domain is used
- `letsencrypt-giantswarm` `ClusterIssuer` is used for certificates
- [prometheus-blackbox-exporter-app](https://github.com/giantswarm/prometheus-blackbox-exporter-app) is installed with default selector labels

All of the settings above can be overwritten on the check or global level.
