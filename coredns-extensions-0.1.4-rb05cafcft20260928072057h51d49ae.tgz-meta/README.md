[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/coredns-extensions-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/coredns-extensions-app/tree/main)

# coredns-extensions chart

Giant Swarm offers a coredns-extensions App which can be installed in workload clusters.
Here, we define the coredns-extensions chart with its templates and default configuration.

**What is this app?**

The app provides complementary resources to the [coredns-app](https://github.com/giantswarm/coredns-app) such as VerticalPodAutoscaler CR.

## Installing

There are several ways to install this app onto a workload cluster.

- [Using GitOps to instantiate the App](https://docs.giantswarm.io/tutorials/continuous-deployment/apps/add-appcr/)
- By creating an [App resource](https://docs.giantswarm.io/reference/platform-api/crd/apps.application.giantswarm.io) using the platform API as explained in [Getting started with App Platform](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/).

## Configuring

### values.yaml

**This is an example of a values file you could upload using our web interface.**

```yaml
# values.yaml

vpa:
  maxAllowed:
    Cpu: 2
    Memory: 1Gb

```

### Sample App CR for the management cluster

If you have access to the Kubernetes API on the management cluster, you could create the App CR directly.

Here is an example that would install the app to workload cluster `abc12`:

```yaml
# appCR.yaml

apiVersion: application.giantswarm.io/v1alpha1
kind: App
metadata:
  labels:
    giantswarm.io/cluster: <cluster_id>
  name: <cluster_id>-coredns-extensions
  namespace: <org_namespace>
spec:
  catalog: giantswarm-playground
  kubeConfig:
    inCluster: false
  name: coredns-extensions
  namespace: kube-system
  version: "0.1.0"
```

See our [full reference on how to configure apps](https://docs.giantswarm.io/tutorials/fleet-management/app-platform/app-configuration/) for more details.

## Limitations

This app requires disabling the HorizontalPodAutoscaler memory target of CoreDNS. Add this block to your cluster userconfig before installing it:

```yaml
    global:
      apps:
        coreDns:
          values:
            hpa:
              targetMemoryUtilizationPercentage: 0
```

