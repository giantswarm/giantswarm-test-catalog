[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/dicebear/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/dicebear/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/dicebear/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/dicebear)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# dicebear chart

This chart deploys a self-hosted [DiceBear](https://www.dicebear.com/) avatar HTTP API
(the upstream `dicebear/api` image, mirrored to `gsoci.azurecr.io/giantswarm/dicebear-api`).

**What is this app?**

DiceBear is an open-source avatar generator. This chart runs its HTTP API in-cluster so
the platform can render agent avatar icons — two initials on a deterministic, color-blind-safe
colored background, derived entirely from an agent's technical name.

**Why did we add it?**

Agents on the platform need a stable visual identity that looks the same across every
surface (Backstage, Slack, and any future chat surface). Self-hosting removes a
third-party runtime dependency and rate limits from a customer-facing feature, and lets us
raise the raster output-size cap. See [giantswarm/giantswarm#37211](https://github.com/giantswarm/giantswarm/issues/37211).

**Who can use it?**

The Agent Platform (bumblebee) deploys it wherever the platform bundle runs. It is not
intended for direct customer configuration.

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml

```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Compatibility

This app has been tested to work with the following workload cluster release versions:

- _add release version_

## Limitations

Some apps have restrictions on how they can be deployed.
Not following these limitations will most likely result in a broken deployment.

- _add limitation_

## Credit

- Upstream project: https://github.com/dicebear/dicebear
- Upstream image: https://hub.docker.com/r/dicebear/api
