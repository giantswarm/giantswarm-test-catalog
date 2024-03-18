[![CircleCI](https://circleci.com/gh/giantswarm/linkerd2-cni-app.svg?style=shield)](https://circleci.com/gh/giantswarm/linkerd2-cni-app)

# linkerd2-cni chart

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

Please note, the only supported namespace name is `linkerd-cni`.

If you're installing using the Web UI, you'll need to add the required labels and annotation to the `linkerd-cni` namespace by hand.

### Example AppCR

```
apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  name: linkerd2-cni
  namespace: by7t2
spec:
  catalog: giantswarm
  kubeConfig:
    inCluster: false
  name: linkerd2-cni
  namespace: linkerd-cni
  namespaceConfig:
    annotations:
      linkerd.io/inject: disabled
    labels:
      linkerd.io/cni-resource: "true"
      config.linkerd.io/admission-webhooks: disabled
  version: 0.8.0
```

## Usage with `linkerd` cli

You can use the `linkerd` cli as usual, just don't forget to specify the cni plugins namespace with `--cni-namespace` flag.

## Maintainer info

This chart is maintained using the vendir method. You need [vendir](https://github.com/vmware-tanzu/carvel-vendir) and [yq](https://github.com/mikefarah/yq) binaries for it to work.

Run `make upgrade-chart` to pull the last stable version of the chart from 
<https://github.com/giantswarm/linkerd2-upstream>.

## Credit

* [linkerd/linkerd2](https://github.com/linkerd/linkerd2/tree/main/charts/linkerd2-cni)
