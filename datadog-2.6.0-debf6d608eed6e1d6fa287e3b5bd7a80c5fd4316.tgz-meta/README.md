[![CircleCI](https://circleci.com/gh/giantswarm/datadog-app.svg?style=shield)](https://circleci.com/gh/giantswarm/datadog-app)

# Datadog Agent chart

Giant Swarm offers a Datadog App which can be installed in workload clusters.
Here we define the datadog chart with its templates and default configuration.

**What is this app?**

The Datadog Agent is software that runs on your hosts. It collects events and metrics from hosts and sends them to Datadog, where you can analyze your monitoring and performance data.

## Installing

There are 3 ways to install this app onto a workload cluster.

1. [Using our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app)
2. [Using our API](https://docs.giantswarm.io/api/#operation/createClusterAppV5)
3. Directly creating the [App custom resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) on the management cluster.

## Configuring

### datadog-user-values
**This is an example of a values file you could upload using our web interface.**
```
# datadog-user-values.yaml
datadog:
  datadog:
    clusterName: giantswarm-abc12
    site: datadoghq.eu
    clusterAgent:
      podSecurity:
        podSecurityPolicy:
          create: true
    agents:
      podSecurity:
        podSecurityPolicy:
          create: true
```

### datadog-user-secrets
**This is an example of a secret file you could upload using our web interface.**
```
# datadog-user-secrets.yaml
datadog:
  datadog:
    apiKey: XXX
```

### How to create App CR and ConfigMap for the management cluster

See our [full reference page on how to configure applications](https://docs.giantswarm.io/app-platform/app-configuration/).

## Credit

* [DataDog agent on Github](https://github.com/DataDog/datadog-agent)
* [DataDog agent documentation](https://docs.datadoghq.com/agent/)
* [DataDog helm charts](https://github.com/DataDog/helm-charts)
