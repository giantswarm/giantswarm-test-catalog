[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/gpu-operator-app.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/gpu-operator-app)

# gpu-operator-app

Giant Swarm offers a GPU Operator app which can be installed in workload clusters when using GPU based instance types.

Here, we define the GPU Operator upstream version with additional configuration and network policies.

# Installation

Installation can be done via Happa or by running `helm install --namespace kube-system gpu-operator helm/gpu-operator`.

# Configuration

`Toolkit` and `Driver` **must kept disabled** in the `values.yaml` file. Flatcar already deals with the driver and the toolkit. 

# Credit

This app is based on the images and manifests provided by NVIDIA in their [GPU Operator](https://github.com/NVIDIA/gpu-operator) project.
