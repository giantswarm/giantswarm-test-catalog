[![CircleCI](https://circleci.com/gh/giantswarm/kyverno-app.svg?style=shield)](https://circleci.com/gh/giantswarm/kyverno-app)

# kyverno

Kyverno is an admission controller offering policy enforcement as a validating or mutating webhook.
It audits or enforces policies for cluster resources, and produces reports about the compliance of the cluster.

Giant Swarm uses it to enforce [Pod Security Standards (PSS)][pss-policies] as a replacement for Pod Security Policies (PSPs), as well as other community-supported policies. For more information on the switch from PSP to PSS, see [our blog post][pss-blog].

This repository packages the upstream [Kyverno][kyverno-chart] and [Policy Reporter][policy-reporter] Helm charts as one Giant Swarm app, with Giant Swarm image mirrors, Cilium network policies, VPA resources, report cleanup policies and a set of built-in policy exceptions for platform components.
The Kyverno CRDs are not installed by this chart (`crds.install: false`); they come from the separate [kyverno-crds][kyverno-crds] app, and the PSS policies from [kyverno-policies][kyverno-policies].
All three are normally deployed together through the [security-bundle][security-bundle].

## Installing

The app is published in the `giantswarm` and `default` catalogs and is a cluster singleton.
Install the [kyverno-crds][kyverno-crds] app first, then deploy this app like any other Giant Swarm app with a [Flux HelmRelease](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/deploy-app-helmrelease/), by [creating an App resource](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/), or with [`kubectl gs deploy chart`](https://docs.giantswarm.io/reference/kubectl-gs/deploy-chart/).
Most clusters get it through the [security-bundle][security-bundle] instead.

## Configuring

Upstream values are nested under `kyverno` and `policy-reporter`; see the [Kyverno docs][kyverno-docs] and the [configuration reference in this chart](helm/kyverno/README.md#values) for all values.
The example below scales the admission controller and allows `PolicyException` resources in an additional namespace.

```yaml
# values.yaml
kyverno:
  admissionController:
    replicas: 3

# PolicyExceptions are only accepted in these namespaces.
policyExceptions:
  allowedPolexNamespaces:
    - policy-exceptions
    - my-team-exceptions
```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Development

The upstream charts are vendored into `helm/kyverno/charts/kyverno` and `helm/kyverno/charts/policy-reporter` with [vendir](https://carvel.dev/vendir/) (see [`vendir.yml`](vendir.yml)).

### Steps to update the upstream charts

Renovate opens a pull request bumping the version in `vendir.yml`, and the [Sync from upstream](.github/workflows/zz_generated.sync_from_upstream.yaml) workflow runs on that branch to re-vendor the charts, regenerate the values schema and docs, and push the result.
To do it by hand, bump the `ref` in `vendir.yml`, run `make update-chart`, then `make sync-values` and `make sync-schema`, and review the diff.
`make check-appversion` verifies that the chart's `appVersion` matches the vendored Kyverno subchart.

## Credit

- [kyverno/kyverno](https://github.com/kyverno/kyverno)
- [kyverno/policy-reporter][policy-reporter]

[kyverno-chart]: https://github.com/kyverno/kyverno/tree/main/charts/kyverno
[kyverno-crds]: https://github.com/giantswarm/kyverno-crds
[kyverno-docs]: https://kyverno.io/docs/
[kyverno-policies]: https://github.com/giantswarm/kyverno-policies
[policy-reporter]: https://github.com/kyverno/policy-reporter
[pss-blog]: https://www.giantswarm.io/blog/giant-swarms-farewell-to-psp
[pss-policies]: https://kyverno.io/policies/?policytypes=Pod%2520Security%2520Standards%2520%28Baseline%29%2BPod%2520Security%2520Standards%2520%28Restricted%29
[security-bundle]: https://github.com/giantswarm/security-bundle
