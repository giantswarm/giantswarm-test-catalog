[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/kubescape-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/kubescape-app/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/kubescape-app/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/kubescape-app)

# kubescape

[Kubescape](https://github.com/kubescape/kubescape) is an open-source security platform for Kubernetes that scans clusters for vulnerabilities, misconfigurations and runtime anomalies, and stores the results as in-cluster report resources.
This repository packages the upstream [`kubescape-operator`](https://github.com/kubescape/helm-charts) Helm chart as a Giant Swarm app, with Giant Swarm image mirrors, Kyverno policy exceptions, PodLogs collection and the capability set we run by default.
Giant Swarm's [starboard-exporter](https://github.com/giantswarm/starboard-exporter) can expose Kubescape's report data as Prometheus metrics.

## Installing

The app is published in the `giantswarm` and `default` catalogs and can be deployed like any other Giant Swarm app with a [Flux HelmRelease](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/deploy-app-helmrelease/), by [creating an App resource](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/), or with [`kubectl gs deploy chart`](https://docs.giantswarm.io/reference/kubectl-gs/deploy-chart/).
It runs on management and workload clusters and needs no scanning backend of its own.

## Configuring

All upstream values are nested under `kubescape-operator`; see the [upstream chart documentation](https://kubescape.io/docs/) for the full list.
The example below turns off two of the heavier runtime capabilities and routes the node-agent to a non-default runc path.

```yaml
# values.yaml
kubescape-operator:
  capabilities:
    runtimeDetection: disable
    malwareDetection: disable

# Where the node-agent finds runc on the host. Defaults to /opt/bin/runc for Flatcar.
global:
  overrideRuntimePath: /usr/bin/runc

# Namespace where Kyverno policy exceptions for Kubescape components are created.
exceptions:
  enabled: true
  namespace: policy-exceptions
```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Known issues

- Flatcar images from Giant Swarm release v35 onwards ship runc only at `/opt/bin/runc`. The node-agent crash-loops when it cannot find runc, which is why `global.overrideRuntimePath` defaults to that path since app version 0.1.2. Override it on nodes with a different layout.

## Development

The upstream charts are vendored into `helm/kubescape/charts` with [vendir](https://carvel.dev/vendir/) (see [`vendir.yml`](vendir.yml)).
Run `make update-chart` to sync a new upstream version; Renovate opens the version bump and the sync workflow rebuilds the vendored chart from it.

## Credit

- [kubescape/kubescape](https://github.com/kubescape/kubescape) and the [kubescape/helm-charts](https://github.com/kubescape/helm-charts) `kubescape-operator` chart this app wraps.
