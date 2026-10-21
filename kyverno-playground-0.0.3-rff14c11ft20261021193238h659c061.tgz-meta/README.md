[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/kyverno-playground/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/kyverno-playground/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/kyverno-playground/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/kyverno-playground)

# kyverno-playground

This repository packages the upstream [Kyverno Playground](https://github.com/kyverno/playground) Helm chart as a Giant Swarm app.
The Playground is a web application for writing and testing Kyverno policies against sample or live cluster resources without applying anything to the cluster, and it optionally ships an MCP server so AI agents can validate policies the same way.
The app is published for everyone in the Giant Swarm catalog but is not a managed app: Giant Swarm does not monitor it and support is best effort.

## Installing

Deploy it like any other Giant Swarm app with a [Flux HelmRelease](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/deploy-app-helmrelease/), by [creating an App resource](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/), or with [`kubectl gs deploy chart`](https://docs.giantswarm.io/reference/kubectl-gs/deploy-chart/).
The Playground works against any cluster; connected cluster mode additionally needs the RBAC granted by the `clusterRoles` and `roles` values.

## Configuring

All upstream values are nested under `kyverno-playground`; see the [chart README](helm/kyverno-playground/README.md) for the full list.
This example exposes the UI through an Ingress and enables connected cluster mode so the Playground can load existing resources and policies from the cluster it runs in.

```yaml
# values.yaml
kyverno-playground:
  ingress:
    enabled: true
    className: nginx
    hosts:
      - host: kyverno-playground.example.com
        paths:
          - path: /
            pathType: Prefix
  config:
    cluster:
      enabled: true
  mcp:
    enabled: false
```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Development

The upstream chart is vendored into `helm/kyverno-playground/charts/kyverno-playground` with [vendir](https://carvel.dev/vendir/) (see [`vendir.yml`](vendir.yml)).
Run [`sync/patches/sync.sh`](sync/patches/sync.sh) to pull a new upstream version and update the chart dependencies.

## Credit

- [kyverno/playground](https://github.com/kyverno/playground), whose chart and application this app wraps.
