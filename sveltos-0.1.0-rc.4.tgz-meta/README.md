[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/sveltos/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/sveltos/tree/main)

# sveltos chart

Giant Swarm copy of the upstream [Sveltos](https://projectsveltos.github.io/sveltos/) Helm chart
([`charts/projectsveltos`](https://github.com/projectsveltos/helm-charts/tree/main/charts/projectsveltos)).
Sveltos manages Kubernetes add-ons and configuration across a fleet of clusters.

Differences from upstream:

- the chart is named `sveltos` and carries the Giant Swarm metadata (team, icon, annotations),
- all images, including the agents the controllers deploy at runtime, come from `gsoci.azurecr.io/giantswarm/projectsveltos/`
  (`global.registry`),
- telemetry is disabled (`telemetry.disabled: true`),
- every resource has the `application.giantswarm.io/team` label.

The dashboard chart is in [giantswarm/sveltos-dashboard](https://github.com/giantswarm/sveltos-dashboard).

## Installing

This app is a cluster singleton: install it only once per cluster, usually on the management cluster.

The recommended way to install this app is a Flux `HelmRelease`:

- [Deploying an application via a Flux HelmRelease](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/deploy-app-helmrelease/)
- [Adding a HelmRelease via GitOps](https://docs.giantswarm.io/tutorials/continuous-deployment/helm-releases/add-helmrelease/)

## Configuring

See [`values.yaml`](helm/sveltos/values.yaml) for all options.

## Upgrading from upstream

`helm/sveltos` is a git subtree of the upstream chart, managed with
[git-subtree-utils](https://github.com/giantswarm/git-subtree-utils) (see [`subtree-cfg.ini`](subtree-cfg.ini)).
The [`auto-upgrade`](.github/workflows/auto-upgrade.yaml) workflow runs daily and opens an `upstream-sync`
pull request for a new upstream `projectsveltos-X.Y.Z` tag. [`post-hook.sh`](post-hook.sh) keeps our
`Chart.yaml` and copies the upstream versions into it.

Merge the subtree pull requests with a merge commit, never squash: the next sync finds the last synced
tag from the git note on the merge commit, and `git subtree` needs the squashed upstream commit in the history.

`helm/sveltos/values.schema.json` and `helm/sveltos/README.md` come from upstream, keep them unchanged.
If you add a value, add it to the schema too.

## Credit

- https://github.com/projectsveltos/helm-charts
