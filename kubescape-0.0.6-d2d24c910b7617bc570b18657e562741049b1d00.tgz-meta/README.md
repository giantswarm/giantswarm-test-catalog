[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/openssf-scorecard-exporter/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/kubescape-app/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/openssf-scorecard-exporter/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/kubescape-app)
[Guide about how to manage an app on Giant Swarm](https://handbook.giantswarm.io/docs/dev-and-releng/app-developer-processes/adding_app_to_appcatalog/)

# kubescape-app

[Kubescape](https://github.com/kubescape/kubescape) is an open-source security scanning tool for Kubernetes and the devops ecosystem. This repository contains a Helm chart for Kubescape, packaged as used by Giant Swarm.

Kubescape scans Kubernetes clusters for vulnerabilities, dangerous misconfigurations, and other risks, and generates reports and remediation recommendations.

Giant Swarm's [starboard-exporter](https://github.com/giantswarm/starboard-exporter) can be used to expose additional metrics for Kubescape's native report data.

## Credit

- https://github.com/kubescape/kubescape
