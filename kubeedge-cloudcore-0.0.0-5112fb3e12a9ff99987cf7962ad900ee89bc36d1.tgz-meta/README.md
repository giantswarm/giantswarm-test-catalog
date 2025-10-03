[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/kubeedge-cloudcore-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/kubeedge-cloudcore-app/tree/main)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# kubeedge-cloudcore chart

## Configuring

In order to schedule Cilium agent pods correctly, we need to stop them from running on KubeEdge nodes. A separate DaemonSet will be created automatically for any edge nodes. To do this, add the following to the Cilium chart's values:

### values.yaml

Note that the `podAntiAffinity` is taken from the [cilium-app](https://github.com/giantswarm/cilium-app/blob/main/helm/cilium/values.yaml) values - this is to ensure it doesn't get overwritten.

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - topologyKey: kubernetes.io/hostname
        labelSelector:
          matchLabels:
            k8s-app: cilium
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
          - key: node-role.kubernetes.io/edge
            operator: DoesNotExist

```

## Credit

- https://github.com/kubeedge/kubeedge
