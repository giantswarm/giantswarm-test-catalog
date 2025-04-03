[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/kong-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/kong-app/tree/main)

# kong-app chart

[*Kong for Kubernetes*](https://github.com/Kong/kubernetes-ingress-controller) is
an open-source Ingress Controller for Kubernetes that offers API management capabilities
with a plugin architecture.

Giant Swarm offers a Kong Managed App which can be installed in workload clusters.

## Available Versions

| Giant Swarm Chart Release | Upstream Chart Release | Kong Version | Kong IC Version | Kong-Gateway Enterprise container tag |
| --- | --- | --- | --- | --- |
| Unreleased | [2.48.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2480) | [3.8.0](https://github.com/Kong/kong/blob/3.8.0/changelog/3.8.0/3.8.0.md) | [3.4.3](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#343) | [3.8.1.0-debian](https://docs.konghq.com/gateway/changelog/#3810) |
| [v4.5.1](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#451---2025-02-18) | [2.46.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2460) | [3.8.0](https://github.com/Kong/kong/blob/3.8.0/changelog/3.8.0/3.8.0.md) | [3.4.1](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#341) | [3.8.1.0-debian](https://docs.konghq.com/gateway/changelog/#3810) |
| [v4.5.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#450---2025-01-28) | [2.46.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2460) | [3.8.0](https://github.com/Kong/kong/blob/3.8.0/changelog/3.8.0/3.8.0.md) | [3.4.1](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#341) | [3.8.1.0-debian](https://docs.konghq.com/gateway/changelog/#3810) |
| [v4.4.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#440---2024-08-19) | [2.40.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2400) | [3.7.1](https://github.com/Kong/kong/blob/3.7.1/changelog/3.7.1/3.7.1.md) | [3.2.3](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#323) | [3.7.1.2-debian](https://docs.konghq.com/gateway/changelog/#3712) |
| [v4.3.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#430---2024-07-30) | [2.39.3](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2393) | [3.7.1](https://github.com/Kong/kong/blob/3.7.1/changelog/3.7.1/3.7.1.md) | [3.2.3](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#323) | [3.7.1.2-debian](https://docs.konghq.com/gateway/changelog/#3712) |
| [v4.2.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#420---2024-07-23) | [07ddc3d](https://github.com/Kong/charts/compare/kong-2.35.1...07ddc3d) | [3.6.1](https://github.com/Kong/kong/blob/3.6.1/changelog/3.6.1/3.6.1.md) | [3.1.6](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#316) | 3.6.1.7-debian |
| [v4.1.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#410---2024-04-29) | [07ddc3d](https://github.com/Kong/charts/compare/kong-2.35.1...07ddc3d) | [3.6.1](https://github.com/Kong/kong/blob/3.6.1/changelog/3.6.1/3.6.1.md) | [3.1.4](https://github.com/Kong/kubernetes-ingress-controller/blob/v3.1.4/CHANGELOG.md#314) | 3.6.1.3-debian |
| [v4.0.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#400---2024-04-22) | [07ddc3d](https://github.com/Kong/charts/compare/kong-2.35.1...07ddc3d) | [3.6.1](https://github.com/Kong/kong/blob/3.6.1/changelog/3.6.1/3.6.1.md) | [3.1.3](https://github.com/Kong/kubernetes-ingress-controller/blob/v3.1.3/CHANGELOG.md#313) | 3.6.1.3-debian |
| [v3.8.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#380---2024-03-27) | [2.35.1](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2351) | [3.5.0](https://github.com/Kong/kong/blob/3.5.0/changelog/3.5.0/3.5.0.md) | [3.0.2](https://github.com/Kong/kubernetes-ingress-controller/blob/v3.0.2/CHANGELOG.md#302) | 3.5.0.3-debian |
| [v3.7.1](https://github.com/giantswarm/kong-app/blob/release-v3.7.x/CHANGELOG.md#371---2024-07-08) | [2.33.3](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2333) | [3.5.0](https://github.com/Kong/kong/blob/3.5.0/changelog/3.5.0/3.5.0.md) | [3.0.2](https://github.com/Kong/kubernetes-ingress-controller/blob/v3.0.2/CHANGELOG.md#302) | 3.5.0.2-debian |
| [v3.7.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#370---2024-01-16) | [2.33.3](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2333) | [3.5.0](https://github.com/Kong/kong/blob/3.5.0/changelog/3.5.0/3.5.0.md) | [3.0.2](https://github.com/Kong/kubernetes-ingress-controller/blob/v3.0.2/CHANGELOG.md#302) | 3.5.0.2-debian |
| [v3.6.2](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#362---2023-12-13) | [2.33.1](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2331) | [3.5.0](https://github.com/Kong/kong/blob/3.5.0/changelog/3.5.0/3.5.0.md) | [3.0.1](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#301) | 3.5.0.1-debian |
| [v3.6.1](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#360---2023-12-11) | [2.33.1](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2331) | [3.5.0](https://github.com/Kong/kong/blob/3.5.0/changelog/3.5.0/3.5.0.md) | [3.0.1](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#301) | 3.5.0.1-debian |
| [v3.6.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#360---2023-12-07) | [2.33.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2330) | [3.5.0](https://github.com/Kong/kong/blob/3.5.0/changelog/3.5.0/3.5.0.md) | [3.0.1](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#301) | 3.5.0.1-debian |
| [v3.5.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#350---2023-10-16) | [2.29.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2290) | [3.4.2](https://github.com/Kong/kong/blob/3.4.2/CHANGELOG.md#342) | [2.12.0](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#2111) | 3.4.1.1-debian |
| [v3.4.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#340---2023-08-22) | [2.23.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2230) | [3.3.1](https://github.com/Kong/kong/blob/3.3.1/CHANGELOG.md#331) | [2.10.4](https://github.com/Kong/kubernetes-ingress-controller/blob/v2.10.4/CHANGELOG.md#2103) | 3.3.1.0-debian |
| [v3.3.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#330---2023-05-17) | [2.21.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2210) | [3.2.2](https://github.com/Kong/kong/blob/3.2.2/CHANGELOG.md#322) | [2.9.3](https://github.com/Kong/kubernetes-ingress-controller/blob/v2.9.3/CHANGELOG.md#293) | 3.2.2.1-debian |
| [v3.2.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#320---2023-05-04) | [2.20.1](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2201) | [3.2.2](https://github.com/Kong/kong/blob/3.2.2/CHANGELOG.md#322) | [2.9.3](https://github.com/Kong/kubernetes-ingress-controller/blob/v2.9.3/CHANGELOG.md#293) | 3.2.2.1-debian |
| [v3.1.1](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#311---2023-04-24) | [2.19.1](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2191) | [3.2.2](https://github.com/Kong/kong/blob/3.2.2/CHANGELOG.md#322) | [2.9.3](https://github.com/Kong/kubernetes-ingress-controller/blob/v2.9.3/CHANGELOG.md#293) | 3.2.2.1-debian |
| [v3.1.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#310---2023-04-20) | [2.19.0](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2190) | [3.2.2](https://github.com/Kong/kong/blob/3.2.2/CHANGELOG.md#322) | [2.9.3](https://github.com/Kong/kubernetes-ingress-controller/blob/v2.9.3/CHANGELOG.md#293) | 3.2.2.1-debian |
| [v3.0.1](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#301---2023-02-14) | [2.16.4](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2164) | [3.1.1](https://github.com/Kong/kong/blob/3.1.1/CHANGELOG.md#311) | [2.8.1](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#281) | 3.1.1.3-debian |
| [v3.0.0](https://github.com/giantswarm/kong-app/blob/main/CHANGELOG.md#300---2023-02-09) | [2.16.2](https://github.com/Kong/charts/blob/main/charts/kong/CHANGELOG.md#2162) | [3.1.1](https://github.com/Kong/kong/blob/3.1.1/CHANGELOG.md#311) | [2.8.1](https://github.com/Kong/kubernetes-ingress-controller/blob/main/CHANGELOG.md#281) | 3.1.1.3-debian |

For older versions, please refer to the [changelog](https://github.com/giantswarm/kong-app/blob/master/CHANGELOG.md)

## Configuration

This Chart is configured to deploy *Kong OSS*. For instructions on how to deploy Kong Enterprise please read [Kong Enterprise](#kong-enterprise).

When supplying a set of custom configuration options, only include configuation you want to change. **DO NOT** copy the whole `values.yaml` file.

For detailed explanations of configuration options, please refer to the [list of configuration options](https://github.com/giantswarm/kong-app/blob/master/helm/kong-app/README.md#configuration) and
also the [`values.yaml` file](https://github.com/giantswarm/kong-app/blob/master/helm/kong-app/values.yaml)

Any key value put under the `env` section translates to environment variables
used to control Kong's configuration. Every key is prefixed with KONG_ and
upper-cased before setting the environment variable.

This app does not by default provide a database and if a database is required,
then you will need to BYOD (Bring Your Own Database). For testing purposes, it
is possible launch postgres alongside this App (described below).

The default installation of the App will use Kong Ingress Controller.
The recommended way to configure plugins, consumers and services when using *Kong for Kubernetes* is by utilizing [Kong annotations](https://docs.konghq.com/kubernetes-ingress-controller/latest/references/annotations/) and [Kong custom resources](https://docs.konghq.com/kubernetes-ingress-controller/latest/concepts/custom-resources/#main).

### Container image registry

You can change the container image registry by setting the following values in your `values.yaml` file

```yaml
image:
  registry: &registry docker.io
ingressController:
  image:
    registry: *registry

# In case you've enabled the postgresql sub-chart
# postgresql:
#   image:
#     registry: *registry

# In case you've changed the default waitImage
# waitImage:
#   registry: *registry
```

### Kong Enterprise

Kong enterprise requires a valid enterprise license Secret to be present in the target namespace for kong.

Save the license key to a plain text file named `kong-enterprise-license.json`.
Then create the Secret with name `kong-enterprise-license` in namespace `kong-app` by running the following command:

```bash
kubectl create secret generic kong-enterprise-license \
  --namespace kong-app \
  --from-file=license=./kong-enterprise-license.json
```

Make sure to set the following values in your `values.yaml` file:

```yaml
image:
  repository: giantswarm/kong-gateway
  tag: 3.8.1.0-debian # check the Release matrix above for possible versions
enterprise:
  enabled: true
  licenseSecret: kong-enterprise-license
```

### Using your own Database

Kong supports PostgreSQL version 9.5 up to 15.

Example database configuration:

```YAML
env:
  database: "postgres" # can be "off" or "postgres"
  pg_host: 127.0.0.1
  pg_port: 12345
  pg_user: postgres
  pg_password: # This can also be a string value, but not recommended
    valueFrom:
      secretKeyRef:
        name: postgres
        key: kong
```

_note_: If `pg_port` is not set then it will default to `5432`

#### Installing a database alongside the App

For testing purposes, it is possible to install a PostgreSQL server alongside
the App. To do this, you'll need to specify the following app configuration:

```YAML
postgresql:
  enabled: true
```
(There is no need to add PostgreSQL configuration data to `env`, a default user
and password will be used.)

Please note: This configuration should only be used for testing and is not officially
supported by Giant Swarm.

### Using Kong Ingress Controller with a Database

When using Kong Ingress Controller, a database is not required.
However in some cases (for example, plugin support, Kong developer portal) a database is
required. The Kong Ingress Controller does not communicate with the database directly.
Instead, it uses the Admin API of the proxy container which then persists configuration either
in memory or in the configured database.

Like in the case of using a database, Giant Swarm does not support the database and
can only provide best efforts support with this configuration.

To configure, please see ['Using your own Database'](#using-your-own-database).
But ensure that that `ingressController.enabled` is set to `true`.

Note:
> Is it possible to create consumers using the Admin API?
>
> From version 0.5.0 onwards, Kong Ingress Controller tags each entity that it
> manages inside Kong's database and only manages the entities that it creates.
> This means that if consumers and credentials are created dynamically,
> they won't be deleted by the Ingress Controller.
(Taken from https://docs.konghq.com/kubernetes-ingress-controller/latest/faq/#is-it-possible-to-create-consumers-using-the-admin-api)

### Kong Ingress Controller CRDs

When installed through the Giant Swarm App platform, CRD installation is taken care of
automatically.

To disable CRD installation, set the following value in your `values.yaml` file:

```yaml
kubectlApplyJob:
  enabled: false
```

## Development

Information about chart and version development can be found in [sync/README.md](https://github.com/giantswarm/kong-app/blob/main/sync/README.md).

## Automatic and manual testing

The helm chart in this repository undergoes [a series of automated tests](https://github.com/giantswarm/kong-app/blob/master/tests/ats/test_basic_cluster.py) running on a [kind](https://kind.sigs.k8s.io/) cluster ([kind cluster config](https://github.com/giantswarm/kong-app/blob/master/tests/kind_config.yaml)) executed by [app-test-suite](https://github.com/giantswarm/app-test-suite). ([chart values used for tests](https://github.com/giantswarm/kong-app/blob/master/tests/test-values.yaml))

Testing includes creation of `Deployment`, `Service`, and `Ingress` resources to check if reconciliation works as intended.

To execute tests on your machine, obtain a copy of the [dats.sh](https://github.com/giantswarm/app-test-suite/releases/download/v0.2.4/dats.sh) helper script and a chart archive (`helm pull https://giantswarm.github.io/giantswarm-catalog/kong-app-2.5.0.tgz` or build an archive with your changes using [`app-build-suite`](http://github.com/giantswarm/app-build-suite)), then execute

```
./dats.sh -c kong-app-2.5.0.tgz
```

Please check out file [`.ats/main.yaml`](.ats/main.yaml) for some additional hints configuring app-test-suite.

### Manual testing

The repository includes resources to test this chart and chart upgrades in directory [`tests/manual`](tests/manual).

## Security Policy

### Reporting a Vulnerability

Please visit https://www.giantswarm.io/responsible-disclosure for information on
reporting security issues.

## Credit

* https://github.com/Kong/charts/
