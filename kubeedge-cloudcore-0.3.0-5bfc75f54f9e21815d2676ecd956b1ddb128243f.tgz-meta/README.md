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

## Updating

> [!WARNING]
> DO NOT merge Renovate PRs without following the instructions below.
>
> Failure to do so will not update the chart itself.

### Renovate updates

1. Check out the PR created by Renovate.
2. Run `sync/sync.sh` which will retrieve the upstream chart, customize it for Giant Swarm, and update the chart files.

### Manual updates

1. Update the `tag` field in [`vendir.yml`](vendir.yml) to the desired upstream version.
2. Run `sync/sync.sh` which will retrieve the upstream chart, customize it for Giant Swarm, and update the chart files.

## Credit

- https://github.com/kubeedge/kubeedge
