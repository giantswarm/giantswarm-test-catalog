[![CircleCI](https://circleci.com/gh/giantswarm/auth-bundle.svg?style=shield)](https://circleci.com/gh/giantswarm/auth-bundle)

# auth-bundle chart

The auth bundle provides all necessary components to enable auth capabilities in a giantswarm cluster.

## Apps

* [dex-app](https://github.com/giantswarm/dex-app)
* [athena](https://github.com/giantswarm/athena)
* [rbac-bootstrap-app](https://github.com/giantswarm/rbac-bootstrap-app)
* [ingress-nginx](https://github.com/giantswarm/ingress-nginx-app)

## Installing

There are several ways to install this app onto a workload cluster.

- [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app).
- By creating an [App resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/app-platform/getting-started/).

## Configuring

### values.yaml

for each app you can use `userConfig` to supply values
or `extraConfigs` as secret or configmap

### cluster

To enable access to your cluster via dex, ensure that the needed [oidc settings are enabled on the cluster resource.](https://docs.giantswarm.io/advanced/access-management/configure-dex-in-your-cluster/#configure-the-oidc-values-on-the-cluster-resource)
