[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/{APP-NAME}/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/{APP-NAME}/tree/main)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# Flux Operator Helm Chart

Giant Swarm offers a Flux Operator app which can be installed in workload clusters.
Here, we define the Flux Operator Helm Chart with its templates and default configuration.

**What is this app?**

The Flux Operator is a Kubernetes CRD controller that manages the lifecycle of CNCF Flux and the ControlPlane enterprise distribution.

**Why did we add it?**

The operator extends Flux with self-service capabilities, deployment windows and preview environments for GitLab and GitHub pull requests testing.

**Who can use it?**

Any user of the Giant Swarm platform.

## Installing

This app is a Helm Chart, hence any method capable of handling Helm Charts is good for installation, for example:

[App Platform](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/deploy-app/#creating-an-app-resource)
[Flux](https://fluxcd.io/flux/components/helm/helmreleases/)
[Helm binary](https://helm.sh/docs/helm/helm_install/)


## Limitations

Some apps have restrictions on how they can be deployed. Not following these limitations will most likely result in a broken deployment.

- only a single instance of this app is allowed in within a cluster.

## Credit

- https://github.com/controlplaneio-fluxcd/charts/tree/main/charts/flux-operator
