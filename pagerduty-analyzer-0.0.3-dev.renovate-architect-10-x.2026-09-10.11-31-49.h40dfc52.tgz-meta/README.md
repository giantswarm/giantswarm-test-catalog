[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/pagerduty-analyzer/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/pagerduty-analyzer/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/pagerduty-analyzer/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/pagerduty-analyzer)

# PagerDuty Analyzer

Automated incident analysis service for Giant Swarm clusters. Accepts analysis requests, extracts cluster information, and triggers diagnostic analysis using the [shoot](https://github.com/giantswarm/shoot) service.

## What is this app?

PagerDuty Analyzer is a FastAPI service that:
- Accepts analysis requests with incident details
- Extracts workload cluster IDs from incident summaries
- Automatically triggers [shoot](https://github.com/giantswarm/shoot) diagnostic analysis
- Returns structured analysis results

## Why did we add it?

To automate the initial investigation of cluster incidents reported through PagerDuty, reducing response time and providing engineering teams with immediate diagnostic insights.

## Architecture

```
Incident tooling → pagerduty-analyzer → shoot → Kubernetes clusters
```

The service acts as a bridge between PagerDuty incident notifications and automated cluster diagnostics.

## Quick Start

### Local Development

1. **Clone the repository**:
```bash
git clone https://github.com/giantswarm/pagerduty-analyzer.git
cd pagerduty-analyzer
```

2. **Create virtual environment**:
```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

3. **Configure environment**:
```bash
cp .env.example .env
# Edit .env with your settings
```

4. **Run the service**:
```bash
uvicorn src.pagerduty_analyzer.main:app --reload --port 8080
```

5. **Test with a sample request**:
```bash
curl -X POST "http://localhost:8080/analyze?incident_id=test123"
```

### Docker

1. **Build and run**:
```bash
docker build -t pagerduty-analyzer:local .
docker run -p 8080:8080 \
  -e SHOOT_ENDPOINT=http://host.docker.internal:8000 \
  pagerduty-analyzer:local
```

2. **Using docker-compose**:
```bash
docker-compose up
```

## Configuration

Configuration is managed through environment variables. See `.env.example` for all options.

### Required Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `SHOOT_ENDPOINT` | URL of the shoot analysis service | `http://localhost:8000` |
| `PAGERDUTY_API_TOKEN` | PagerDuty API token used for incident lookup | (required) |

### Optional Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `HOST` | Server bind address | `0.0.0.0` |
| `PORT` | Server port | `8080` |
| `LOG_LEVEL` | Logging level (DEBUG, INFO, WARNING, ERROR) | `INFO` |
| `SHOOT_TIMEOUT` | Timeout for shoot requests (seconds) | `300` |
| `SHOOT_MAX_TURNS` | Max agent turns for shoot analysis | `15` |
| `PAGERDUTY_API_URL` | Base URL for PagerDuty REST API | `https://api.pagerduty.com` |
| `PAGERDUTY_API_TIMEOUT` | Timeout for PagerDuty API requests (seconds) | `20` |

### Cluster ID Extraction

The service uses built-in regex patterns to extract installation, cluster IDs, and alert names from incident titles. The workload pattern matches:
- `grizzly-5e4cr`
- `goggle-abc123`

Management-cluster titles like `grizzly - ClusterCrossplaneResourcesNotReady` use the installation name as the cluster ID.

## API Endpoints

### `POST /analyze`

Manually trigger analysis for a specific incident.

**Query Parameters**:
- `incident_id`: PagerDuty incident ID

**Example**:
```bash
curl -X POST "http://localhost:8080/analyze?incident_id=test123"
```

### `GET /health`

Health check endpoint.

**Response**:
```json
{"status": "ok"}
```

### `GET /ready`

Readiness check endpoint.

**Response**:
```json
{"status": "ready"}
```

## Development

### Running Tests

```bash
# Install dev dependencies
pip install -r requirements-dev.txt

# Run tests with coverage
pytest

# Run linting
ruff check .

# Run type checking
mypy src/
```

### Code Style

This project uses:
- **ruff** for linting and formatting
- **mypy** for type checking
- **pytest** for testing

## Deployment

### Kubernetes/Helm

See the `helm/pagerduty-analyzer` directory for Helm chart deployment.

### Environment Variables in Production

Ensure the following are configured:
- `SHOOT_ENDPOINT`: Points to your shoot service
- `LOG_LEVEL`: Set to `INFO` or `WARNING` in production

## Monitoring

The service exposes:
- `/health`: Liveness probe
- `/ready`: Readiness probe

Configure your monitoring system to check these endpoints.

## Troubleshooting

### Cluster ID not extracted

Enable `LOG_LEVEL=DEBUG` to see extraction attempts and verify title formats against the built-in patterns.

### Shoot analysis fails

Verify:
- `SHOOT_ENDPOINT` is accessible from the analyzer
- Shoot service has proper Kubernetes credentials
- Network connectivity between services

## License

Apache-2.0

## Links

- [shoot service](https://github.com/giantswarm/shoot)
- [Giant Swarm Documentation](https://docs.giantswarm.io)
