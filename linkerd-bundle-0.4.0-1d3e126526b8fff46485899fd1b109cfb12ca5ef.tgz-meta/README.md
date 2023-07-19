[![CircleCI](https://circleci.com/gh/giantswarm/linkerd-bundle.svg?style=shield)](https://circleci.com/gh/giantswarm/linkerd-bundle)

# linkerd-bundle chart

The linkerd bundle provides the necessary components to enable service mesh capabilities in a workload cluster.

**Apps**

- [linkerd2-cni-app](https://github.com/giantswarm/linkerd2-cni-app)
- [linkerd-control-plane-app](https://github.com/giantswarm/linkerd-control-plane-app)
- [linkerd-multicluster-app (optional)](https://github.com/giantswarm/linkerd-multicluster-app)
- [linkerd-multicluster-link-app (optional)](https://github.com/giantswarm/linkerd-multicluster-link-app)
- [linkerd-viz-app](https://github.com/giantswarm/linkerd-viz-app)

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/advanced/gitops/#installing-managed-apps)
- [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app).
- By creating an [App resource](https://docs.giantswarm.io/use-the-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/app-platform/getting-started/) and the [App Bundle reference](https://docs.giantswarm.io/getting-started/app-platform/app-bundle/). Also check out the [examples](https://github.com/giantswarm/linkerd-bundle/tree/main/examples).

## Configuring

Check the [examples](https://github.com/giantswarm/linkerd-bundle/tree/main/examples) for different configuration scenarios.

See our [full reference on how to configure apps](https://docs.giantswarm.io/app-platform/app-configuration/) for more details.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- _add release version_
