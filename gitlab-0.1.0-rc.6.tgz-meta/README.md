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

This repository is private, so the chart is private too: each release is pushed to
`oci://gsociprivate.azurecr.io/charts/giantswarm/gitlab`. Pulling it needs credentials for `gsociprivate.azurecr.io`:
a `kubernetes.io/dockerconfigjson` Secret in the namespace of the `OCIRepository` (below, `gitlab-pull-secret`),
created as described in the
[image pull secret runbook](https://intranet.giantswarm.io/docs/support-and-ops/runbooks/create-image-pull-secret/),
and referenced by `spec.secretRef`. The image itself comes from the public `gsoci.azurecr.io` and needs no secret.

Before installing, provide these in the release's namespace; the chart creates none of them but the buckets, and
the pod waits until every Secret exists:

- **PostgreSQL 16 or newer**: a database owned by GitLab's role, e.g. a CloudNativePG `Cluster` whose bootstrap
  creates database `gitlabhq_production` owned by `gitlab`. Its `<cluster>-rw` Service is `postgresql.host`, its
  `<cluster>-app` Secret (key `password`) is `postgresql.existingSecret`. The owner creates the `pg_trgm` and
  `btree_gist` extensions on first start (both trusted extensions).
- **Valkey or Redis**: its Service is `redis.host`, the Secret with the password of its default user is
  `redis.existingSecret`.
- **An S3 key**: on the endpoint `objectStorage.endpoint`, its id in `objectStorage.accessKeyId`, its secret in the
  Secret `objectStorage.existingSecret`, allowed to create buckets (Garage: `garage key allow --create-bucket <key>`).
  The chart creates the buckets itself (see [Object storage](#object-storage)). GitLab proxies every download, so the
  endpoint can stay inside the cluster.
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
  url: oci://gsociprivate.azurecr.io/charts/giantswarm/gitlab
  provider: generic
  secretRef:
    name: gitlab-pull-secret
  ref:
    # Only release candidates exist so far; ">=0.1.0-0" includes them.
    semver: ">=0.1.0-0"
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

## Object storage

The chart creates the buckets it configures; an installer creates none by hand. Every bucket of
`objectStorage.buckets` (by default `gitlab-artifacts`, `gitlab-external-diffs`, `gitlab-lfs`, `gitlab-uploads`,
`gitlab-packages`, `gitlab-dependency-proxy`, `gitlab-terraform-state`, `gitlab-ci-secure-files` and
`gitlab-ci-catalog-bundles`) is created by a Helm pre-install and pre-upgrade hook Job, `<release>-create-buckets`,
before the StatefulSet is applied. It uses the same endpoint, region, path style and key as GitLab, creates a bucket
that is missing and leaves one that exists as it is, so it runs on every install and upgrade and a second run changes
nothing. On Garage, a bucket the Job creates is an alias local to the key, which is the key GitLab reads and writes
it with. A chart release that adds a bucket therefore rolls on a floating version range without the bucket existing
beforehand: the upgrade creates it before the pod that needs it starts.

The Job runs the GitLab image, which the StatefulSet pulls anyway, with Omnibus' own Ruby and AWS SDK, as user 65534
on a read-only root filesystem with every capability dropped: the Pod Security Standards **restricted** profile. It
carries the pod's `nodeSelector`, `tolerations` and `affinity`. A succeeded Job is deleted; a failed one keeps its logs
for a day or until the next install or upgrade replaces it, and the release fails rather than start GitLab without its
buckets. Like the pod, it waits for the Secret `objectStorage.existingSecret`, within the release's Helm timeout.
`objectStorage.createBuckets.enabled: false` switches the Job off, for an object store whose buckets another system
manages.

## Pod Security Standards: the baseline profile

The GitLab pod meets the Pod Security Standards **baseline** profile, not **restricted** (the bucket Job meets
restricted). The Omnibus image cannot run
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
