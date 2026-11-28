# beekeeper

beekeeper is being rebuilt as a new major version. The previous implementation has been
removed from this repository; its releases remain on
[GitHub Releases](https://github.com/giantswarm/beekeeper/releases).

Until the new code lands, the repository holds a skeleton:

- `main.go`, a placeholder binary that has no commands yet.
- `api/v1alpha1`, Beekeeper's resources: runners, agent tasks, agent sessions and
  subscriptions ([docs/api.md](docs/api.md)), with their generated CustomResourceDefinitions in
  `config/crd`.
- `helm/beekeeper`, a chart that installs no resources.
- The generated build, test and release pipeline (CircleCI and GitHub Actions).

## Development

```bash
make build   # the binary for this machine
make test    # Go tests and the chart's unit tests
make generate  # deepcopy functions and CustomResourceDefinitions from api/
pre-commit run --all-files
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for commit and pull request conventions.
