[![CircleCI](https://dl.circleci.com/status-badge/img/gh/giantswarm/pagerduty-analyzer/tree/main.svg?style=svg)](https://dl.circleci.com/status-badge/redirect/gh/giantswarm/pagerduty-analyzer/tree/main)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/giantswarm/pagerduty-analyzer/badge)](https://securityscorecards.dev/viewer/?uri=github.com/giantswarm/pagerduty-analyzer)

# PagerDuty Analyzer

Automated incident analysis service for Giant Swarm clusters. Receives PagerDuty webhooks, extracts cluster information, and triggers diagnostic analysis using the [shoot](https://github.com/giantswarm/shoot) service.

## What is this app?

PagerDuty Analyzer is a FastAPI service that:
- Receives PagerDuty webhook notifications for `incident.triggered` and `incident.created` events
- Extracts workload cluster IDs from incident summaries
- Retrieves incident details from custom fields (`_description`)
- Automatically triggers [shoot](https://github.com/giantswarm/shoot) diagnostic analysis
- Returns structured analysis results

## Why did we add it?

To automate the initial investigation of cluster incidents reported through PagerDuty, reducing response time and providing engineering teams with immediate diagnostic insights.

## Architecture

```
PagerDuty → pagerduty-incident-proxy → pagerduty-analyzer → shoot → Kubernetes clusters
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

5. **Test with a sample webhook**:
```bash
curl -X POST http://localhost:8080/webhook/pagerduty \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{
      "event": "incident.triggered",
      "incident": {
        "id": "test123",
        "summary": "Cluster abc123 has pod failures",
        "custom_fields": {
          "_description": "Multiple pods in kube-system namespace are not ready"
        }
      }
    }]
  }'
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

### Optional Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `HOST` | Server bind address | `0.0.0.0` |
| `PORT` | Server port | `8080` |
| `LOG_LEVEL` | Logging level (DEBUG, INFO, WARNING, ERROR) | `INFO` |
| `SHOOT_TIMEOUT` | Timeout for shoot requests (seconds) | `300` |
| `SHOOT_MAX_TURNS` | Max agent turns for shoot analysis | `15` |
| `WEBHOOK_SECRET` | PagerDuty webhook signing secret (for verification) | None |
| `CLUSTER_ID_PATTERN` | Regex to extract cluster ID from summary | `(?:cluster[:\s-]*)?([a-z0-9]{5,})` |

### Cluster ID Extraction

The service uses regex patterns to extract cluster IDs from incident summaries. The default pattern matches:
- `cluster: abc123`
- `cluster-abc123`
- `Cluster abc123`
- `abc123` (standalone)

Customize the pattern via `CLUSTER_ID_PATTERN` environment variable.

## API Endpoints

### `POST /webhook/pagerduty`

Receives PagerDuty webhooks and processes incident events.

**Request Headers**:
- `Content-Type: application/json`
- `X-PagerDuty-Signature: v1=<hmac>` (optional, for signature verification)

**Request Body**:
```json
{
  "messages": [{
    "event": "incident.triggered",
    "incident": {
      "id": "incident123",
      "summary": "Cluster abc123 has issues",
      "custom_fields": {
        "_description": "Detailed problem description"
      }
    }
  }]
}
```

**Response** (202 Accepted):
```json
{
  "processed": 1,
  "skipped": 0,
  "results": [{
    "incident_id": "incident123",
    "cluster_id": "abc123",
    "event_type": "incident.triggered",
    "summary": "Cluster abc123 has issues",
    "description": "Detailed problem description",
    "shoot_triggered": true,
    "shoot_response": {
      "result": "Diagnostic report...",
      "request_id": "uuid-here",
      "metrics": {...}
    }
  }]
}
```

### `POST /analyze`

Manually trigger analysis for a specific incident.

**Query Parameters**:
- `incident_id`: PagerDuty incident ID
- `summary`: Incident summary (must contain cluster ID)
- `description`: Optional incident description

**Example**:
```bash
curl -X POST "http://localhost:8080/analyze?incident_id=test123&summary=Cluster%20abc123%20down&description=Pods%20failing"
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

## Integration with PagerDuty

### Webhook Configuration

1. In PagerDuty, go to **Integrations** → **Generic Webhooks (v3)**
2. Add webhook URL: `https://your-analyzer.example.com/webhook/pagerduty`
3. Configure webhook to send:
   - `incident.triggered`
   - `incident.created`
4. (Optional) Enable webhook signing and set `WEBHOOK_SECRET` in your environment

### Integration with pagerduty-incident-proxy

Configure [pagerduty-incident-proxy](https://github.com/giantswarm/pagerduty-incident-proxy) to forward events:

```yaml
# pagerduty-incident-proxy config.yaml
routes:
  - name: analyzer
    match:
      - field: event
        any_of: [incident.triggered, incident.created]
    targets:
      - url: http://pagerduty-analyzer:8080/webhook/pagerduty
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
- `WEBHOOK_SECRET`: Set for security in production
- `LOG_LEVEL`: Set to `INFO` or `WARNING` in production

## Monitoring

The service exposes:
- `/health`: Liveness probe
- `/ready`: Readiness probe

Configure your monitoring system to check these endpoints.

## Troubleshooting

### Cluster ID not extracted

Check the `CLUSTER_ID_PATTERN` regex against your incident summaries. Enable `LOG_LEVEL=DEBUG` to see extraction attempts.

### Shoot analysis fails

Verify:
- `SHOOT_ENDPOINT` is accessible from the analyzer
- Shoot service has proper Kubernetes credentials
- Network connectivity between services

### Webhook signature verification fails

Ensure `WEBHOOK_SECRET` matches the secret configured in PagerDuty.

## License

Apache-2.0

## Links

- [shoot service](https://github.com/giantswarm/shoot)
- [pagerduty-incident-proxy](https://github.com/giantswarm/pagerduty-incident-proxy)
- [Giant Swarm Documentation](https://docs.giantswarm.io)
