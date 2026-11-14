[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/coredns-app/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/coredns-app/tree/main)

# coredns-app

Helm Chart for CoreDNS in Workload Clusters.

* Installs the the DNS server [CoreDNS](https://github.com/coredns/coredns).

# Deployment

Managed by the Giant Swarm [App Platform](https://docs.giantswarm.io/app-platform/).

# Configuration Options

- All configuration options are documented in the [values.yaml](/helm/coredns-app/values.yaml) file.
- [Advanced CoreDNS configuration](https://docs.giantswarm.io/advanced/coredns/) docs.

# For developers

## Installing the Chart

To install the chart locally:

```bash
$ git clone https://github.com/giantswarm/coredns-app.git
$ cd coredns-app
$ helm install helm/coredns-app
```

Provide a custom `values.yaml`:

```bash
$ helm install coredns-app -f values.yaml
```
