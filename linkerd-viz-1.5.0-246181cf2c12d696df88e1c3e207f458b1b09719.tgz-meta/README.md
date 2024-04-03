[![CircleCI](https://circleci.com/gh/giantswarm/{APP-NAME}-app.svg?style=shield)](https://circleci.com/gh/giantswarm/{APP-NAME}-app)

# linkerd-viz chart

This chart is an extension for [Linkerd2 service mesh app](https://github.com/giantswarm/linkerd2-app) provided by Giant Swarm.
It adds a visualization stack of prometheus and metrics components to the cluster, pre-configured for the linkerd-control-plane-app.

## Installing

There are 3 ways to install this app onto a workload cluster.

1. [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app)
2. [Using our API](https://docs.giantswarm.io/api/#operation/createClusterAppV5)
3. Directly creating the [App custom resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) on the management cluster.

## Configuring

The user configuration should only contain values that are additional to or diverge from the default values provided by the chart. Check our [user configuration docs](https://docs.giantswarm.io/app-platform/app-configuration/) to see how configurations are merged.


### Sample App CR and ConfigMap for the management cluster
If you have access to the Kubernetes API on the management cluster, you could create
the App CR and ConfigMap directly.

Here is an example that would install the app to
workload cluster `<cluster-id>`:

```
# appCR.yaml
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  name: linkerd-viz
  namespace: <cluster-id>
spec:
  catalog: giantswarm
  kubeConfig:
    inCluster: false
  name: linkerd-viz
  namespace: linkerd-viz
  namespaceConfig:
    labels:
      kubernetes.io/metadata.name: linkerd-viz
      linkerd.io/extension: viz
  userConfig:
    configMap:
      name: linkerd-viz-userconfig-<your-cluster-id>
      namespace: <your-cluster-id>
    secret:
      name: linkerd-viz-userconfig-<your-cluster-id>
      namespace: <your-cluster-id>
  version: 0.8.0

```

See our [full reference page on how to configure applications](https://docs.giantswarm.io/app-platform/app-configuration/) for more details.

## Credit

* https://github.com/linkerd/linkerd2/tree/main/viz/charts/linkerd-viz
