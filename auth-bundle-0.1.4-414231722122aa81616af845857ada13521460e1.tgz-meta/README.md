[![CircleCI](https://circleci.com/gh/giantswarm/auth-bundle.svg?style=shield)](https://circleci.com/gh/giantswarm/auth-bundle)

# auth-bundle chart

The auth bundle provides all necessary components to enable auth capabilities in a giantswarm cluster.

## Apps

* [dex-app](https://github.com/giantswarm/dex-app)
* [athena](https://github.com/giantswarm/athena)
* [rbac-bootstrap-app](https://github.com/giantswarm/rbac-bootstrap-app)
* [ingress-nginx](https://github.com/giantswarm/ingress-nginx-app)

## Installing

### Quick Start

The auth-bundle can be installed using Giant Swarm's web interface or via direct Kubernetes resource deployment. For a quick installation, follow these steps:

1. **Web Interface Installation**:
   - [Navigate to our web interface](https://docs.giantswarm.io/ui-api/web/app-platform/#installing-an-app).
   - Select your organization, then your cluster and choose the option to install a new app.
   - Search for `auth-bundle` from the Giant Swarm Catalog and follow the on-screen instructions.

2. **Direct Kubernetes Resource Deployment**:
   - Ensure you have the `app` CRD installed in your management cluster.
   - Create an [App resource](https://docs.giantswarm.io/ui-api/management-api/crd/apps.application.giantswarm.io/) in the management cluster as explained in [Getting started with App Platform](https://docs.giantswarm.io/app-platform/getting-started/).

## Configuring

Each app within the `auth-bundle` can be configured to meet your specific needs. For each app you can use `userConfig` to supply values or `extraConfigs` as secret or configmap
### Example configuration
```yaml
apps:
  athena:
    userConfig:
      configMap:
        values: |
          managementCluster:
            name: op2df
  dex-app:
    userConfig:
      configMap:
        values: |
          isWorkloadCluster: true
          deployDexK8SAuthenticator: false
      secret: true
  ingress-nginx:
    enabled: false
```
### [Dex-app](https://github.com/giantswarm/dex-app) Configuration

- **Enable access to your cluster via dex**: ensure that the needed [oidc settings are enabled on the cluster resource.](https://docs.giantswarm.io/advanced/access-management/configure-dex-in-your-cluster/#configure-the-oidc-values-on-the-cluster-resource)
- **Deploying Dex K8s Authenticator**: Optional based on requirements. Can be enabled with `deployDexK8SAuthenticator: true`.
- **Copying OIDC Configuration**: It's possible to copy the OIDC part from the MC to the WC, ensuring seamless authentication across clusters.

### [Athena](https://github.com/giantswarm/athena) Configuration

- **Setup Guide**: Detailed instructions on setting up Athena for authentication management, highlighting its integration with Dex in Workload Clusters.

### Troubleshooting Common Issues

- **Dex-app Authentication Failure**: Verify the Secret contains the correct OIDC provider details. Ensure the `dex-app` configuration matches your provider's requirements.
- **RBAC Permissions Issues**: Check the `rbac-bootstrap-app` has been configured with correct policies. Ensure roles and role bindings are correctly applied.
- **Ingress and External-DNS**: Ingress and External DNS configuration challenges, including ingress class settings and external DNS behavior when nginx is not in the `kube-system` namespace, or when the ingress is not pubilcally accessible.
