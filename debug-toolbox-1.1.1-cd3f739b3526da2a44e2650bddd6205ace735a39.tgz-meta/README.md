# debug-toolbox-app

This app is a debug toolbox for Giant Swarm clusters.

## Installation

Running the kubectl gs template with the following command:

```sh
export ORG=your-org
export NAMESPACE=your-namespace
export CLUSTER=your-cluster
kubectl gs template app --catalog giantswarm --name debug-toolbox --namespace org-$ORG --target-namespace $NAMESPACE --version 1.0.0 --cluster-name $CLUSTER
```

It will create a `App` resource in the management cluster.

When you want to run with especial PSS support, you can run the following command:

```sh
export ORG=your-org
export NAMESPACE=your-namespace
export CLUSTER=your-cluster
kubectl gs template app --catalog giantswarm --name debug-toolbox \
  --namespace org-$ORG --target-namespace $NAMESPACE \
  --version 1.0.0 --cluster-name $CLUSTER \
  --user-configmap=helm/debug-toolbox/values_pss_example.yaml
```

And tune the values example to your needs.
