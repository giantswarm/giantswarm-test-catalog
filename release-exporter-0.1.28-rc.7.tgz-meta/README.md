# release-exporter

`release-exporter` fetches github data and serve it as metrics for our Grafana Cloud.

## Features

- Expose the release information for all our apps.

## Development

The following environment variables need to be set:

- `GITHUB_KEY` - a GitHub token with `repo` scope
