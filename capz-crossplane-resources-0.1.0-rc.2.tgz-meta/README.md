[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/capz-crossplane-resources/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/capz-crossplane-resources/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/capz-crossplane-resources/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/capz-crossplane-resources)

# capz-crossplane-resources

A Helm chart of Crossplane resources that every CAPZ management cluster needs. It is deployed by the CAPZ
collection of [management-cluster-bases](https://github.com/giantswarm/management-cluster-bases).

## Resources

### The CAPZ identity

The Crossplane `UserAssignedIdentity` `capz` observes the management cluster's CAPZ user-assigned managed identity
(`<managementCluster>-capz` in the resource group `<managementCluster>`). It is observe-only
(`managementPolicies: [Observe]`): Crossplane never changes or deletes the identity.

Its `status.atProvider.principalId` is the identity's object ID, which nothing else on the management cluster
records. Workload cluster charts read it from there, for example [cluster-aks](https://github.com/giantswarm/cluster-aks)
to grant the identity Azure RBAC roles on an AKS cluster.

## Configuring

The chart's values are documented in [helm/capz-crossplane-resources/README.md](helm/capz-crossplane-resources/README.md).
On a management cluster:

```yaml
managementCluster: <codename>
```

The subscription that holds the identity defaults to `spec.subscriptionID` of the management cluster's own
`AzureCluster` `org-giantswarm/<managementCluster>`. Set `capzIdentity.subscriptionId` where that does not apply.

## Requirements

- Crossplane with the Upbound Azure provider family's `provider-azure-managedidentity`.
- A Crossplane Azure `ProviderConfig` named after the management cluster (or set `providerConfigName`) whose
  identity may read the CAPZ identity.
