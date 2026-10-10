# gitlab chart

GitLab CE in one container: the Omnibus image `gitlab/gitlab-ce` (Puma, Sidekiq, Gitaly, gitlab-shell, Workhorse
and nginx) on one volume, behind a StatefulSet, on the installation's own PostgreSQL, Redis and object storage. A small
GitLab for testing integrations against, such as a git provider for workspaces: OAuth 2.0 applications with PKCE, the
REST API, project and group access tokens, git over HTTPS and SSH.

The chart bundles no database, cache or object store: Giant Swarm installations already run the PostgreSQL operator
(CloudNativePG), Valkey and an S3 object store, so GitLab uses them instead of starting its own copies.

It is not a production GitLab. The official [GitLab chart](https://docs.gitlab.com/charts/) is that path. With
PostgreSQL, Redis and MinIO disabled it still runs about ten images (webservice, Workhorse, Sidekiq, Gitaly,
gitlab-shell, toolbox, KAS, the exporter, the migrations and certificates jobs), each mirrored to gsoci, and it reads
object storage credentials from a Secret holding a whole connection document, which the installer would have to
assemble from the object store's own Secret. The Omnibus image is one multi-arch image of about 1.4 GiB and its
`GITLAB_OMNIBUS_CONFIG` takes the three services and the existing Secrets directly, so it is the smaller of the two
for one small instance.

The image is mirrored from Docker Hub to `gsoci.azurecr.io/giantswarm/gitlab-ce` by
[retagger](https://github.com/giantswarm/retagger); the chart's `appVersion` is the image tag and Renovate follows
the `-ce.0` tags.

## Installing

The chart is published to the `giantswarm` app catalog and to `oci://gsoci.azurecr.io/giantswarm/gitlab`.
Before installing, provide these in the release's namespace; the chart creates none of them, and the pod waits until
every Secret exists:

- **PostgreSQL 16 or newer**: a database owned by GitLab's role, e.g. a CloudNativePG `Cluster` whose bootstrap
  creates database `gitlabhq_production` owned by `gitlab`. Its `<cluster>-rw` Service is `postgresql.host`, its
  `<cluster>-app` Secret (key `password`) is `postgresql.existingSecret`. The owner creates the `pg_trgm` and
  `btree_gist` extensions on first start (both trusted extensions).
- **Valkey or Redis**: its Service is `redis.host`, the Secret with the password of its default user is
  `redis.existingSecret`.
- **S3 buckets**: one per object type in `objectStorage.buckets`, by default `gitlab-artifacts`,
  `gitlab-external-diffs`, `gitlab-lfs`, `gitlab-uploads`, `gitlab-packages`, `gitlab-dependency-proxy`,
  `gitlab-terraform-state`, `gitlab-ci-secure-files` and `gitlab-ci-catalog-bundles`, on the endpoint
  `objectStorage.endpoint`, with one key: its id in `objectStorage.accessKeyId`, its secret in the Secret `objectStorage.existingSecret`. GitLab proxies
  every download, so the endpoint can stay inside the cluster.
- **The initial root password**:

  ```sh
  kubectl -n gitlab create secret generic gitlab-root-password --from-literal=password='<a password of 12 or more characters>'
  ```

  GitLab reads it only on its first start.

The first start migrates the database and sets the instance up; it takes several minutes; the
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
    postgresql:
      host: gitlab-db-rw.gitlab.svc
    redis:
      host: gitlab-valkey.gitlab.svc
    objectStorage:
      endpoint: http://garage.garage.svc:3900
      accessKeyId: GK0123456789abcdef01234567
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
- **External services**: the bundled PostgreSQL and Redis are off (`postgresql['enable']`, `redis['enable']`);
  `gitlab_rails['db_*']` and `gitlab_rails['redis_*']` point at `postgresql.*` and `redis.*`. Object storage is the
  consolidated `gitlab_rails['object_store']` form: one AWS-style connection (`endpoint`, `region`, `path_style`)
  and a bucket per type for artifacts, external diffs, LFS, uploads, packages, the dependency proxy, Terraform state,
  CI secure files and CI/CD catalog bundles, with downloads proxied through GitLab. The passwords and the S3 secret key reach the container
  as environment variables from their Secrets, never in the rendered configuration. A changed password in a Secret
  takes effect when the pod restarts.
- **One volume** (`persistence.size`, 30 GiB by default) holds `/var/opt/gitlab` (the Git repositories) and
  `/etc/gitlab` (the generated secrets, which decrypt the database's encrypted columns, and the SSH host keys). Logs
  go to an `emptyDir`, `/dev/shm` is a 256 MiB memory volume.
- **Resources**: 2 CPU and 4 GiB requested, a 6 GiB memory limit.
- **Probes** run `curl` inside the container against `/-/liveness` and `/-/readiness`: GitLab answers those only to
  its monitoring allowlist, which is localhost.
- **SSH** is off by default; with `ssh.enabled` a second Service exposes port 22 of the pod as a NodePort,
  LoadBalancer or ClusterIP behind a TCPRoute. `ssh.externalPort` is the port clone URLs show.
- **A NetworkPolicy** admits traffic to the HTTP and SSH ports only, from the peers in `networkPolicy.from`
  (anywhere by default).

## Pod Security Standards: the baseline profile

The chart meets the Pod Security Standards **baseline** profile, not **restricted**. The Omnibus image cannot run
as a non-root user: its entrypoint starts as root, runs `gitlab-ctl reconfigure` (which owns the data directories
to the service users `git` and `gitlab-www`) and starts every service through
runit, which switches to those users itself. Restricted forbids exactly that (`runAsNonRoot`, and no capability but
`NET_BIND_SERVICE`).

What the chart does set: `allowPrivilegeEscalation: false`, `privileged: false`, the `RuntimeDefault` seccomp
profile, every capability dropped except the ones a root start that switches users needs (`CHOWN`,
`DAC_OVERRIDE`, `FOWNER`, `SETUID`, `SETGID`, `KILL`, `NET_BIND_SERVICE`) and the ones sshd's privilege
separation needs to chroot its pre-authentication child into `/run/sshd` and log through the audit subsystem
(`SYS_CHROOT`, `AUDIT_WRITE`), all in the baseline set; no host namespaces, ports or paths, no service account
token. The root filesystem stays writable: Omnibus writes its
runtime state below `/opt/gitlab` and `/run` on start. On a cluster that enforces restricted, install it into a
namespace with a policy exception for the `require-run-as-nonroot`, `require-run-as-non-root-user` and
`disallow-capabilities-strict` rules.

## Limitations

- One replica. Omnibus is a single instance; `replicas: 0` stops it and keeps the volume.
- No container registry, GitLab Pages or KAS.
- No backups of the repositories: a test instance. The database and the buckets are backed up, if at all, by the
  services that hold them.

## Credit

- [GitLab Omnibus](https://gitlab.com/gitlab-org/omnibus-gitlab), MIT licensed
