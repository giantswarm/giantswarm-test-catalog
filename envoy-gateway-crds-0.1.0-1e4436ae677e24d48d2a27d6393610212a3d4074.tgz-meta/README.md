[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/envoy-gateway-crds-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/envoy-gateway-crds-app/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/envoy-gateway-crds-app/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/envoy-gateway-crds-app)

# envoy-gateway-crds chart

Giant Swarm offers an envoy-gateway-crds app that installs the Envoy Gateway CRDs in workload clusters.

**What is this app?**

This app installs the Custom Resource Definitions (CRDs) for [Envoy Gateway](https://gateway.envoyproxy.io/). These CRDs define resources such as `EnvoyProxy`, `BackendTrafficPolicy`, `ClientTrafficPolicy`, `SecurityPolicy`, and others that Envoy Gateway uses to configure the proxy.

**Why did we add it?**

The Envoy Gateway CRDs are distributed separately from the main `envoy-gateway-app` so they can be managed independently. This follows the same pattern as Gateway API CRDs, allowing CRDs to be installed and upgraded without touching the controller.

**Who can use it?**

It is intended for platform teams managing Envoy Gateway on Giant Swarm clusters. It is typically installed as part of the `gateway-api-bundle`.

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)
- By creating an [App resource](https://docs.giantswarm.io/reference/platform-api/crd/apps.application.giantswarm.io) using the platform API as explained in [Getting started with App Platform](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/).

## Configuring

This chart has no configurable values. It only installs CRDs.

## Compatibility

| envoy-gateway-crds-app | Envoy Gateway |
| --- | --- |
| 1.8.x | 1.8.x |

## Development

The CRDs are extracted from the upstream `gateway-helm` chart using vendir and a sync script:

```bash
./sync/sync.sh
```

This fetches the upstream chart into `vendor/` and extracts only the Envoy Gateway CRDs into `helm/envoy-gateway-crds/templates/crds/`.

## Credit

- https://github.com/envoyproxy/gateway
