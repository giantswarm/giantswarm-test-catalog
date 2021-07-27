[![CircleCI](https://circleci.com/gh/giantswarm/linkerd2-cni-app.svg?style=shield)](https://circleci.com/gh/giantswarm/linkerd2-cni-app)

# linkerd2-cni-app chart

Giant Swarm offers a Linkerd2 CNI Managed App which can be installed in tenant clusters.
Here we define the Linkerd2 CNI chart with its templates and default configuration.

## Installation

The target namespace should have the following annotations and labels (use App CR's
[namespaceConfig](https://docs.giantswarm.io/app-platform/namespace-configuration/#configuring-labels--annotations) to deliver them):

* labels
  * `config.linkerd.io/admission-webhooks=disabled`
  * `linkerd.io/cni-resource="true"`
* annotation 
  * `linkerd.io/inject=disabled`

### Example AppCR

```
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  name: linkerd2-cni-app
  namespace: by7t2
spec:
  catalog: giantswarm-test
  kubeConfig:
    inCluster: false
  name: linkerd2-cni-app
  namespace: linkerd2-cni-app
  namespaceConfig:
    annotations:
      linkerd.io/inject: disabled
    labels:
      linkerd.io/cni-resource: "true"
      config.linkerd.io/admission-webhooks: disabled
  version: 0.4.0
```

## Usage with `linkerd` cli

You can use the `linkerd` cli as usual, just don't forget to specify the cni plugins namespace with `--cni-namespace` flag.

## Maintainer info

This chart is maintained using the git subtree method from
<https://github.com/giantswarm/linkerd2-upstream>.
Pay attention that in order to sync the chart, you need to synchronize two
directories `helm/linkerd2-cni-app` and `helm/partials`.

## Credit

* [linkerd/linkerd2](https://github.com/linkerd/linkerd2/tree/main/charts/linkerd2-cni)
