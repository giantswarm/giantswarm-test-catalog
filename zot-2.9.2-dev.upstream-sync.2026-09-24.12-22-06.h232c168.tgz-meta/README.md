# Zot chart

This chart installs the [Zot](https://zotregistry.dev/) registry. It is based on the official upstream chart.

## Contributing

*Please note*: this repo is managed using the git-subtree module, all PRs have to be merged with merge commits,
not squashes!

## Memory management recommendations

### Generic requests and limits with VPA

This is the standard approach. VPA is enabled by default with room to increase Zot deployment CPU and memory limits.

### Fixed memory usage with Go GC configuration

In this use case, we recommend disabling the VPA and set a fixed memory request and limit (same value) for
the Zot deployment. Alongside that, configure the Go Garbage Collector with `.env` Helm values.

We recommend setting `GOMEMLIMIT` to about 80% of the pod memory limits. You should not set `GOGC` higher than `100`.

See: https://tip.golang.org/doc/gc-guide

```yaml
# With the fix memory usage, we recommend not using a VPA in this scenario as the GOMEMLIMIT will hard limit the memory usage
giantswarm:
  verticalPodAutoscaler:
    enabled: false

# Leave some room for the CPU usage as the GC will require it to clean it
resources:
  requests:
    cpu: 300m
    # Fixed request
    memory: 1024Mi
  limits:
    cpu: 500m
    # Fixed limit
    memory: 1024Mi

# Configure the Go Garbage Collector via environment variables
env:
  - name: "GOGC"
    # Keep it around 50, cannot be increased beyond 100
    value: "50"
  - name: "GOMEMLIMIT"
    # About 80% of the fixed memory limit
    value: "800MiB"
```

Setting a high enough priority class can also help keeping the deployment a little more stable.

### Notes on extension memory usage

In our experience, enabling the Zot `ui` and `search` (needed for `ui` extension) significantly increases the memory
requirements for Zot.
