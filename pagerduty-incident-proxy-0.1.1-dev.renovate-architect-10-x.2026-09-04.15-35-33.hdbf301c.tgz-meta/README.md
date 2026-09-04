# PagerDuty Incident Proxy

Tiny FastAPI service that receives PagerDuty webhooks and forwards `incident.*` events to downstream management cluster endpoints based on declarative routing rules.

## Quickstart (local)

```bash
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
cp config.example.yaml config.yaml   # adjust routes/targets
CONFIG_PATH=./config.yaml uvicorn src.main:app --reload --port 8080
```

Send a sample webhook:

```bash
curl -X POST http://localhost:8080/webhook/pagerduty \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"event":"incident.trigger","incident":{"service":{"summary":"cluster-a"}}}]}'
```

## Configuration

Configuration is YAML based (`config.yaml` by default, override with `CONFIG_PATH`):

- `allowed_events`: list of PagerDuty event names to accept.
- `webhook_signing_secret`: enable HMAC-SHA256 verification of `X-PagerDuty-Signature` using your webhook signing key.
- `routes`: evaluated top → bottom, first match wins. Each route has `match` rules that inspect fields using dot-notation (e.g. `incident.service.summary`) with `equals`, `any_of`, or `contains` comparators, and a list of `targets` to forward to.
- `default_targets`: fallback when no route matches.

See `config.example.yaml` for a concrete example.

## Container image

Build locally:

```bash
docker build -t pagerduty-incident-proxy:local .
docker run -p 8080:8080 -v $PWD/config.example.yaml:/etc/pagerduty-incident-proxy/config.yaml pagerduty-incident-proxy:local
```

## Helm chart

The chart in `helm/pagerduty-incident-proxy` deploys the proxy. Key values:

- `image.repository` / `image.tag`: container image to run.
- `service.port`: port exposed inside the cluster.
- `proxy.config`: rendered into `/etc/pagerduty-incident-proxy/config.yaml` inside the pod.

Example snippet:

```yaml
proxy:
  config:
    allowed_events: [incident.trigger, incident.create]
    routes:
      - name: cluster-a
        match:
          - field: incident.service.summary
            contains: cluster-a
        targets:
          - url: https://cluster-a.example.com/hooks/pagerduty
            headers:
              Authorization: "Bearer <token>"
    default_targets: []
```
