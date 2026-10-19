# AWS Cost Exporter

Prometheus exporter that exposes AWS resource costs for Kubernetes clusters.

## Features
- Discovers cluster nodes, EBS volumes, LoadBalancers, and NAT gateways
- Calculates hourly costs for AWS resources
- Supports both on-demand and spot instance pricing
- Runs inside Kubernetes clusters

## Usage
The AWS Cost Exporter runs as a pod inside your Kubernetes cluster and exposes cost metrics via a Prometheus endpoint. It discovers cluster resources and calculates their associated AWS costs.

### Prerequisites
- Kubernetes cluster running on AWS
- AWS credentials with permissions to:
  - Describe EC2 instances
  - Access pricing information
  - List EBS volumes
  - Describe load balancers
  - List VPC resources

### Installation
[Coming soon: Helm chart and deployment instructions]

### Configuration
[Coming soon: Configuration options and environment variables]

## Development
### Requirements
- Go 1.21+
- AWS SDK
- Kubernetes client-go
- Prometheus client

### Local Development
1. Clone the repository
```bash
git clone https://github.com/giantswarm/aws-cost-exporter.git
cd aws-cost-exporter
```

2. Install dependencies
```bash
go mod download
```

3. Run tests
```bash
go test ./...
```

### Project Structure
```
aws-cost-exporter/
├── cmd/
│   └── aws-cost-exporter/     # Main application entry point
├── pkg/
│   ├── collector/             # Prometheus collectors
│   ├── aws/                   # AWS client and pricing
│   └── metrics/               # Metrics definitions
```

## Testing
The project uses localstack with testcontainer for AWS service mocking in tests. This ensures reliable and reproducible testing of AWS interactions without requiring real AWS credentials.

### Running Tests
```bash
# Unit tests
go test ./...
```

### Building
```bash
# Unit tests
go build ./cmd/aws-cost-exporter/
```
