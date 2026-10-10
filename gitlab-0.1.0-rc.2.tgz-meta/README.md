# gitlab chart

GitLab CE in one container: the Omnibus image `gitlab/gitlab-ce` (Puma, Sidekiq, Gitaly, gitlab-shell, PostgreSQL,
Redis and nginx) on one volume, behind a StatefulSet. A small, self-contained GitLab for testing integrations
against, such as a git provider for workspaces: OAuth 2.0 applications with PKCE, the REST API, project and group
access tokens, git over HTTPS and SSH.

It is not a production GitLab. The official [GitLab chart](https://docs.gitlab.com/charts/) is that path: since
its 10.0 release it bundles no PostgreSQL, Redis or object storage, so its smallest install is ten images and three
external services. This chart is one multi-arch image of about 1.4 GiB and one volume.

The image is mirrored from Docker Hub to `gsoci.azurecr.io/giantswarm/gitlab-ce` by
[retagger](https://github.com/giantswarm/retagger); the chart's `appVersion` is the image tag and Renovate follows
the `-ce.0` tags.

## Installing

The chart is published to the `giantswarm` app catalog and to `oci://gsoci.azurecr.io/giantswarm/gitlab`.
Before installing, create the Secret with the initial root password in the release's namespace:

```sh
kubectl -n gitlab create secret generic gitlab-root-password --from-literal=password='<a password of 12 or more characters>'
```

GitLab reads it only on its first start. The first start sets the instance up and takes several minutes; the
startup probe allows 15 minutes (`probes.startupSeconds`).

A Flux HelmRelease behind a Gateway API Gateway with a cert-manager certificate:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: OCIRepository
metadata:
  name: gitlab
  namespace: gitlab
spec:
  interval: 1h
  url: oci://gsoci.azurecr.io/giantswarm/gitlab
  ref:
    semver: ">=0.1.0"
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: gitlab
  namespace: gitlab
spec:
  interval: 10m
  chartRef:
    kind: OCIRepository
    name: gitlab
  values:
    externalUrl: https://gitlab.example.com
    httpRoute:
      enabled: true
      parentRefs:
        - name: giantswarm-default
          namespace: envoy-gateway-system
          sectionName: https
    ssh:
      enabled: true
      externalPort: 2222
      service:
        type: ClusterIP
    tcpRoute:
      enabled: true
      parentRefs:
        - name: giantswarm-default
          namespace: envoy-gateway-system
          sectionName: gitlab-ssh
```

With an Ingress instead: `ingress.enabled: true`, its class in `ingress.className` and the certificate's annotation
in `ingress.annotations` (for example `cert-manager.io/cluster-issuer: letsencrypt-giantswarm`).

## Configuring

The values and their defaults are in [helm/gitlab/README.md](helm/gitlab/README.md). What the chart sets up:

- **`GITLAB_OMNIBUS_CONFIG`** is rendered from the values: `external_url`, TLS terminated in front of the pod (nginx
  serves plain HTTP on `http.port` and sends `X-Forwarded-Proto`), Let's Encrypt, the container registry, KAS, Pages,
  the bundled Prometheus and the usage ping off, two Puma workers and a Sidekiq concurrency of 10.
  `omnibus.extraConfig` appends further `gitlab.rb` lines.
- **One volume** (`persistence.size`, 30 GiB by default) holds `/var/opt/gitlab` (repositories, PostgreSQL, Redis,
  uploads) and `/etc/gitlab` (the generated secrets and SSH host keys). Logs go to an `emptyDir`, `/dev/shm` is a
  256 MiB memory volume.
- **Resources**: 2 CPU and 6 GiB requested, an 8 GiB memory limit.
- **Probes** run `curl` inside the container against `/-/liveness` and `/-/readiness`: GitLab answers those only to
  its monitoring allowlist, which is localhost.
- **SSH** is off by default; with `ssh.enabled` a second Service exposes port 22 of the pod as a NodePort,
  LoadBalancer or ClusterIP behind a TCPRoute. `ssh.externalPort` is the port clone URLs show.
- **A NetworkPolicy** admits traffic to the HTTP and SSH ports only, from the peers in `networkPolicy.from`
  (anywhere by default).

## Pod Security Standards: the baseline profile

The chart meets the Pod Security Standards **baseline** profile, not **restricted**. The Omnibus image cannot run
as a non-root user: its entrypoint starts as root, runs `gitlab-ctl reconfigure` (which owns the data directories
to the service users `git`, `gitlab-psql`, `gitlab-redis` and `gitlab-www`) and starts every service through
runit, which switches to those users itself. Restricted forbids exactly that (`runAsNonRoot`, and no capability but
`NET_BIND_SERVICE`).

What the chart does set: `allowPrivilegeEscalation: false`, `privileged: false`, the `RuntimeDefault` seccomp
profile, every capability dropped except the ones a root start that switches users needs (`CHOWN`,
`DAC_OVERRIDE`, `FOWNER`, `SETUID`, `SETGID`, `KILL`, `NET_BIND_SERVICE`, all in the baseline set), no host
namespaces, ports or paths, no service account token. The root filesystem stays writable: Omnibus writes its
runtime state below `/opt/gitlab` and `/run` on start. On a cluster that enforces restricted, install it into a
namespace with a policy exception for the `require-run-as-nonroot`, `require-run-as-non-root-user` and
`disallow-capabilities-strict` rules.

## Limitations

- One replica. Omnibus is a single instance; `replicas: 0` stops it and keeps the volume.
- No container registry, GitLab Pages or KAS.
- No backups: a test instance.

## Credit

- [GitLab Omnibus](https://gitlab.com/gitlab-org/omnibus-gitlab), MIT licensed
