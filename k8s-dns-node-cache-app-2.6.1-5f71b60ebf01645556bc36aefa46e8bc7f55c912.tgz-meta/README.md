[![CircleCI](https://circleci.com/gh/giantswarm/k8s-dns-node-cache-app/tree/main.svg?style=svg)](https://circleci.com/gh/giantswarm/k8s-dns-node-cache-app/tree/main) [![Docker Repository on Quay](https://quay.io/repository/giantswarm/k8s-dns-node-cache/status "Docker Repository on Quay")](https://quay.io/repository/giantswarm/k8s-dns-node-cache)

# k8s-dns-node-cache-app

This repository contains a Giant Swarm managed app that deploys the [Node Local DNS strategy](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/) in a kubernetes cluster.

NodeLocal DNSCache improves Cluster DNS performance by running a dns caching agent on cluster nodes as a DaemonSet.
In today's architecture, Pods in ClusterFirst DNS mode reach out to a kube-dns serviceIP for DNS queries.
This is translated to a kube-dns/CoreDNS endpoint via iptables rules added by kube-proxy.
With this new architecture, Pods will reach out to the dns caching agent running on the same node, thereby avoiding iptables DNAT rules and connection tracking.
The local caching agent will query kube-dns service for cache misses of cluster hostnames(cluster.local suffix by default).

## Release compatibility

Release v1.x works in all CNIs using kubeproxy/iptables.
Release v2.x works in Cilium CNI in EBPf mode.

## Installation and configuration

This App is meant to be a drop-in add-on that can be installed with default configuration in any Giant Swarm cluster.
The app will be running as a daemonset alongside the already existing `coreDNS` app, dramatically improving DNS resolution
performances with a very small footprint in terms of system resources.

## Network Policies

When installing `k8s-dns-node-cache-app` you need to ensure any other workload running in the cluster has access to the new DNS service.
This is an example network policy you might need to create:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-node-local-dns-for-my-app
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Egress
  egress:
  - to:
    - ipBlock:
        # this is the coredns service's Cluster IP.
        cidr: 172.31.0.10/32
    ports:
    - protocol: UDP
      port: 53
  - to:
    - podSelector:
        matchLabels:
          k8s-app: coredns
    ports:
    - protocol: UDP
      port: 53
    - protocol: UDP
      port: 1053
```

## Known issues and limitations

- The upstream application only works with `kube-proxy`-based clusters. It will NOT work when using other `Kubernetes Service` implementations such as `Cilium` for example.
- This app only works with `kube-proxy` in `iptables` mode. The upstream application works in `IPVS` mode as well, but the Giant Swarm app does not support that use case.
- After removing the application previously installed in the cluster, it might take some time for the injected `iptables` rules to be deleted. 
  While that happens, DNS queries will fail for all pods running in that node will fail. We suggest rolling or rebooting all nodes after deleting this app.
- This application makes `net-exporter` <= v1.10.3 probes fail and thus makes clusters page.
- This application does NOT work and breaks networking on AWS clusters (suspect conflict with AWS CNI).
