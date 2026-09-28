[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/kubeedge-cloudcore-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/kubeedge-cloudcore-app/tree/main)

[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# kubeedge-cloudcore chart

## Configuring

The following applies when `.cni.cilium.enabled: true` is set in `values.yaml`. This is the default value.

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

## Connecting an edge node

The edge node must have a container runtime (for example containerd) and [`keadm`](https://kubeedge.io/docs/setup/install-with-keadm) with the same version as cloudcore.

### Get the cloudcore address

The chart creates the `cloudcore` Service of type `LoadBalancer` in the release namespace. The edge node must be able to reach its external IP on ports `10000` (cloudhub) and `10002` (certificates).

```sh
kubectl -n <release-namespace> get svc cloudcore -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

### Get the token

Cloudcore creates the join token and keeps it in the Secret `tokensecret` in the `kubeedge` namespace. This namespace does not change with the release namespace. The token has the CA hash and a signed token. Cloudcore replaces the token every 12 hours (`cloudHub.tokenRefreshDuration`), so get a new token immediately before you join a node. The token is necessary only for the join. After the join, the node uses its certificate.

```sh
kubectl -n kubeedge get secret tokensecret -o jsonpath='{.data.tokendata}' | base64 -d
```

Or, with `keadm` and a kubeconfig for the workload cluster:

```sh
keadm gettoken --kube-config <path-to-kubeconfig>
```

### Join the node

Run this on the edge node:

```sh
keadm join \
  --cloudcore-ipport=<cloudcore-ip>:10000 \
  --token=<token> \
  --kubeedge-version=v1.23.0
```

To make sure that the node is connected, do `kubectl get nodes`. The node has the label `node-role.kubernetes.io/edge`.

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
