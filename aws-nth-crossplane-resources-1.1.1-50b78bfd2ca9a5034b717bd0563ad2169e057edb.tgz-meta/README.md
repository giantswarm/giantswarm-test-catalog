[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/aws-nth-crossplane-resources/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/aws-nth-crossplane-resources/tree/main)

# AWS Node Termination Handler Crossplane Resources

This app contains all the resources needed for Node Termination Handler to work on any management/workload cluster.

Based on https://github.com/giantswarm/aws-node-termination-handler-upstream?tab=readme-ov-file#1-create-an-sqs-queue

Resources created:
- <clusterid>-nth IAM Role used by the deployment to connect to SQS
- <clusterid>-nth-notification IAM Role used in the ASG LifecycleHooks to talk to SQS
- <clusterid>-nth SQS Queue will contain the Termination messages
- <clusterid>-nth SQS Policy


## Why this name for the app and not the full name?

The full name would not work for clusters with 20 characters name.


