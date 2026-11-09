[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/sveltos-dashboard/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/sveltos-dashboard/tree/main)

# sveltos-dashboard chart

Giant Swarm copy of the upstream [Sveltos](https://projectsveltos.github.io/sveltos/) dashboard Helm chart
([`charts/dashboard`](https://github.com/projectsveltos/helm-charts/tree/main/charts/dashboard)).
The dashboard is a web UI for the Sveltos resources and the clusters Sveltos manages.

Differences from upstream:

- the chart carries the Giant Swarm metadata (team, icon, annotations),
- the images come from `gsoci.azurecr.io/giantswarm/projectsveltos/` (`global.registry`),
- every resource has the `application.giantswarm.io/team` label.

The dashboard needs Sveltos in the same cluster: install the [giantswarm/sveltos](https://github.com/giantswarm/sveltos) chart first.

## Installing

The recommended way to install this app is a Flux `HelmRelease`:

- [Deploying an application via a Flux HelmRelease](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/deploy-app-helmrelease/)
- [Adding a HelmRelease via GitOps](https://docs.giantswarm.io/tutorials/continuous-deployment/helm-releases/add-helmrelease/)

## Configuring

See [`values.yaml`](helm/sveltos-dashboard/values.yaml) for all options.

## Upgrading from upstream

`helm/sveltos-dashboard` is a git subtree of the upstream chart, managed with
[git-subtree-utils](https://github.com/giantswarm/git-subtree-utils) (see [`subtree-cfg.ini`](subtree-cfg.ini)).
The [`auto-upgrade`](.github/workflows/auto-upgrade.yaml) workflow runs daily and opens an `upstream-sync`
pull request for a new upstream `sveltos-dashboard-X.Y.Z` tag. [`post-hook.sh`](post-hook.sh) keeps our
`Chart.yaml` and copies the upstream versions into it.

Merge the subtree pull requests with a merge commit, never squash: the next sync finds the last synced
tag from the git note on the merge commit, and `git subtree` needs the squashed upstream commit in the history.

`helm/sveltos-dashboard/values.schema.json` and `helm/sveltos-dashboard/README.md` come from upstream, keep them unchanged.
If you add a value, add it to the schema too.

## Credit

- https://github.com/projectsveltos/helm-charts
