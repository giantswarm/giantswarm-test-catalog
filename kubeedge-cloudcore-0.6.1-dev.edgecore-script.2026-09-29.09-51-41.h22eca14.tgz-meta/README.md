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

## Connecting an edge node

Note that `keadm` can change across versions and cilium support _matures_ over time. So it is best to check this procedure against upstream release notes when deploying a new version.

The edge node must reach the external IP of the `cloudcore` Service on ports `10000` (cloudhub) and `10002` (certificates). It does not need access to the Kubernetes API.

### 1. Install the prerequisites

Run this on the edge node. It installs containerd, runc, the CNI plugins and crictl.

```sh
curl -sfL https://raw.githubusercontent.com/giantswarm/kubeedge-cloudcore-app/main/edgecore/edgecore_prerequisites.sh | sudo bash
```

### 2. Get the join command

Run this with the workload cluster kubeconfig. It prints the command for step 3. Set `NAMESPACE` if cloudcore does not run in `kubeedge`.

```sh
./edgecore/get_join_command.sh
```

Cloudcore replaces the join token every 12 hours, so do step 3 immediately. After the join, the node uses its certificate.

### 3. Join the node

Run the printed command on the edge node. It installs `keadm` with the cloudcore version and joins the node. The join also enables the edgecore metaServer and edgeStream and sets `clusterDNS`, so that Cilium can reach the API, `kubectl logs`/`exec` work, and pods on the edge node get cluster DNS.

To make sure that the node is connected, do `kubectl get nodes`. The node has the label `node-role.kubernetes.io/edge`.

## Credit

- https://github.com/kubeedge/kubeedge
