[![CircleCI](https://circleci.com/gh/giantswarm/linkerd2-cni-app.svg?style=shield)](https://circleci.com/gh/giantswarm/linkerd2-cni-app)

# linkerd2-cni-app chart

Giant Swarm offers a Linkerd2 CNI Managed App which can be installed in tenant clusters.
Here we define the Linkerd2 CNI chart with its templates and default configuration.

## Installation

The target namespace (by default `linkerd-cni`) should have the following
annotations (use App CR's
[namespaceConfig](https://docs.giantswarm.io/app-platform/namespace-configuration/#configuring-labels--annotations) to deliver them):

* `config.linkerd.io/admission-webhooks=disabled`
* `linkerd.io/cni-resource="true"`
* `linkerd.io/inject=disabled`

## Maintainer info

This chart is maintained using the git subtree method from
<https://github.com/giantswarm/linkerd2-upstream>.
Pay attention that in order to sync the chart, you need to synchronize two
directories `helm/linkerd2-cni-app` and `helm/partials`.

## Credit

* [linkerd/linkerd2](https://github.com/linkerd/linkerd2/tree/main/charts/linkerd2-cni)
