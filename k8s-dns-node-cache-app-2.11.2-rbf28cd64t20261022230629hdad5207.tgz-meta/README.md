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

## Known issues and limitations

- The upstream application only works with `kube-proxy`-based clusters. This app has been made compatible with `cilium` as the cluster network proxy following [this guide](https://docs.cilium.io/en/stable/network/kubernetes/local-redirect-policy/#node-local-dns-cache).
- This app only works with `kube-proxy` in `iptables` mode. The upstream application works in `IPVS` mode as well, but the Giant Swarm app does not support that use case.
- After removing the application previously installed in the cluster, it might take some time for the injected `iptables` rules to be deleted. 
  While that happens, DNS queries will fail for all pods running in that node will fail. We suggest rolling or rebooting all nodes after deleting this app.
- This application makes `net-exporter` <= v1.10.3 probes fail and thus makes clusters page.
- Unless running `cilium` as CNI, this application does NOT work and breaks networking on clusters running AWS CNI.
