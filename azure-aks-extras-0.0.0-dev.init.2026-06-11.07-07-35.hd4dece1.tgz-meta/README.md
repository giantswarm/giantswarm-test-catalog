[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/azure-aks-extras/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/azure-aks-extras/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/azure-aks-extras/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/azure-aks-extras)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# azure-aks-extras chart

Giant Swarm offers a azure-aks-extras App which can be installed in workload clusters.
Here, we define the azure-aks-extras chart with its templates and default configuration.

**What is this app?**

This app contains certain resources required on Azure AKS clusters, deployed with our [`cluster-aks` chart](https://github.com/giantswarm/cluster-aks).

**Why did we add it?**

AKS has built-in Coredns, Cilium and metrics-server, so we are not deploying our own apps for those components. That means that we are missing some of the auxiliary resources that come with those charts. This chart contains those resources tailored for our AKS deployment.

**Who can use it?**

This app is deployed out-of-the-box from our [`cluster-aks` chart](https://github.com/giantswarm/cluster-aks), so there's no need for users to deploy it or manage it themselves.

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)
- By creating an [App resource](https://docs.giantswarm.io/reference/platform-api/crd/apps.application.giantswarm.io) using the platform API as explained in [Getting started with App Platform](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/).

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- ...

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- This app is only compatible with [`cluster-aks`](https://github.com/giantswarm/cluster-aks).

## Credit

- https://github.com/giantswarm/cluster-aks
- https://github.com/giantswarm/coredns-app
- https://github.com/giantswarm/metrics-server-app
