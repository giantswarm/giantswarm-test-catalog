# Opsgenie & PagerDuty Bidirectional Integration Proxy

A Python server that provides bidirectional synchronization between Opsgenie and PagerDuty platforms.

## Features

- HTTP server that listens for webhooks from both Opsgenie and PagerDuty
- Bidirectional synchronization of alerts and incidents
- Creates PagerDuty incidents from Opsgenie alerts
- Synchronizes notes from Opsgenie to PagerDuty
- Synchronizes acknowledgement status in both directions
- Verifies PagerDuty webhook signatures for security

## Requirements

- Python 3.7 or later
- Flask
- Requests
- python-dotenv
- opsgenie-sdk

## Installation

1. Clone the repository:

```bash
git clone https://github.com/yourusername/pagerduty-opsgenie-proxy.git
cd pagerduty-opsgenie-proxy
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Configure environment variables:

Create a `.env` file in the project root with the following variables:

```
# PagerDuty Configuration
PAGERDUTY_API_KEY=your_pagerduty_api_key_here
PAGERDUTY_SERVICE_ID=your_pagerduty_service_id_here
PAGERDUTY_WEBHOOK_SECRET=your_pagerduty_webhook_secret
PAGERDUTY_EMAIL=your_email@example.com
PAGERDUTY_USER=username #Avoids loops processing events

# Opsgenie Configuration
OPSGENIE_API_KEY=your_opsgenie_api_key_here
OPSGENIE_INTEGRATION_ID=ccb77fcc-3474-4ee3-8140-300926635ed1
```

## PagerDuty Setup

1. Go to your PagerDuty account and create an API key with appropriate permissions
2. Identify the service ID you want to create incidents for
3. Create a webhook subscription pointing to your server's `/pd-webhook` endpoint
4. Generate a webhook secret and use it to secure your webhook endpoint
5. Add all configuration details to your `.env` file

## Opsgenie Setup

1. Create an API key in Opsgenie with appropriate permissions
2. Set up a webhook integration in Opsgenie pointing to your server's `/ops-webhook` endpoint
3. Note the integration ID of the webhook
4. Add the API key and integration ID to your `.env` file

## Usage

1. Run the server:

```bash
python app.py
```

The server will start listening on port 5001 by default.

2. Configure webhook endpoints:

- Opsgenie webhook: `http://your-server-ip:5001/ops-webhook`
- PagerDuty webhook: `http://your-server-ip:5001/pd-webhook`

## How it Works

### Opsgenie → PagerDuty Direction

#### Alert Creation

1. Opsgenie sends an alert webhook to your server
2. The server checks if the alert is from the configured integration ID
3. If it matches, the server creates a PagerDuty incident with:
   - Title: Opsgenie alert message
   - Description: Opsgenie alert description
   - Urgency: Mapped from Opsgenie priority
   - Incident key: Opsgenie alert ID (for deduplication)
4. The server returns a success response with PagerDuty incident details

#### Note Synchronization

1. When a note is added in Opsgenie, it sends an "AddNote" action to the webhook
2. The server identifies the corresponding PagerDuty incident using the alert ID
3. The note is added to the PagerDuty incident with the username of the person who added it

#### Acknowledgement Tracking

1. When an alert is acknowledged in Opsgenie, it sends an "Acknowledge" action to the webhook
2. The server identifies the corresponding PagerDuty incident using the alert ID
3. A note is added to the PagerDuty incident indicating who acknowledged the alert in Opsgenie
4. Similarly, unacknowledgements are tracked and added as notes

### PagerDuty → Opsgenie Direction

#### Acknowledgement Sync

1. When an incident is acknowledged in PagerDuty, it sends a webhook to the server
2. The server extracts the incident key, which contains the Opsgenie alert ID
3. The server calls the Opsgenie API to acknowledge the corresponding alert
4. The acknowledgement includes the name of the PagerDuty user who performed the action
5. Similarly, unacknowledgements in PagerDuty are synced back to Opsgenie

#### Note Synchronization

1. When a note is added to an incident in PagerDuty, it sends an `incident.annotated` webhook event
2. The server extracts the incident key (Opsgenie alert ID) and the note content
3. The server calls the Opsgenie API to add the note to the corresponding alert
4. The note includes the name of the PagerDuty user who added it

## Supported Actions

### Opsgenie Webhook Actions

- `Create`: Creates a new incident in PagerDuty
- `AddNote`: Adds a note from Opsgenie to the corresponding PagerDuty incident
- `Acknowledge`: Adds a note to the PagerDuty incident indicating who acknowledged the alert
- `UnAcknowledge`: Adds a note to the PagerDuty incident indicating who unacknowledged the alert

### PagerDuty Webhook Events

- `incident.acknowledged`: Acknowledges the corresponding alert in Opsgenie
- `incident.unacknowledged`: Unacknowledges the corresponding alert in Opsgenie
- `incident.annotated`: Adds notes from PagerDuty to the corresponding Opsgenie alert
- `incident.triggered`: No action (alerts are created by Opsgenie to PagerDuty flow)
- `incident.resolved`: Currently not implemented

## Testing

You can test the webhook receiver using curl:

### Testing Opsgenie to PagerDuty:

```bash
# Test alert creation
curl -X POST -H "Content-Type: application/json" -d '{"action":"Create","alert":{"alertId":"test-123","message":"Test alert"},"integrationId":"ccb77fcc-3474-4ee3-8140-300926635ed1"}' http://localhost:5001/ops-webhook

# Test note addition
curl -X POST -H "Content-Type: application/json" -d '{"action":"AddNote","alert":{"alertId":"test-123","username":"test-user"},"note":"This is a test note","integrationId":"ccb77fcc-3474-4ee3-8140-300926635ed1"}' http://localhost:5001/ops-webhook

# Test acknowledgement
curl -X POST -H "Content-Type: application/json" -d '{"action":"Acknowledge","alert":{"alertId":"test-123","username":"test-user"},"integrationId":"ccb77fcc-3474-4ee3-8140-300926635ed1"}' http://localhost:5001/ops-webhook

# Test unacknowledgement
curl -X POST -H "Content-Type: application/json" -d '{"action":"UnAcknowledge","alert":{"alertId":"test-123","username":"test-user"},"integrationId":"ccb77fcc-3474-4ee3-8140-300926635ed1"}' http://localhost:5001/ops-webhook
```

### Testing PagerDuty to Opsgenie:

```bash
# Test acknowledge from PagerDuty
curl -X POST -H "Content-Type: application/json" -d '{
  "event": {
    "id": "01G3PQ97QLWWR8SGW2AT6K18HE",
    "event_type": "incident.acknowledged",
    "resource_type": "incident",
    "occurred_at": "2025-10-27T15:26:48.576Z",
    "agent": {
      "id": "PAGENT123",
      "type": "user_reference",
      "self": "https://api.pagerduty.com/users/PAGENT123",
      "html_url": "https://example.pagerduty.com/users/PAGENT123",
      "summary": "Test User"
    },
    "client": null,
    "data": {
      "id": "PDI123",
      "type": "incident",
      "status": "acknowledged",
      "incident_key": "test-123",
      "title": "Test incident",
      "service": {
        "id": "PSERVICE123",
        "type": "service_reference",
        "summary": "Test Service"
      }
    }
  }
}' http://localhost:5001/pd-webhook

# Test unacknowledge from PagerDuty
curl -X POST -H "Content-Type: application/json" -d '{
  "event": {
    "id": "01G3PQ97QLWWR8SGW2AT6K18HF",
    "event_type": "incident.unacknowledged",
    "resource_type": "incident",
    "occurred_at": "2025-10-27T15:30:48.576Z",
    "agent": {
      "id": "PAGENT123",
      "type": "user_reference",
      "self": "https://api.pagerduty.com/users/PAGENT123",
      "html_url": "https://example.pagerduty.com/users/PAGENT123",
      "summary": "Test User"
    },
    "client": null,
    "data": {
      "id": "PDI123",
      "type": "incident",
      "status": "triggered",
      "incident_key": "test-123",
      "title": "Test incident",
      "service": {
        "id": "PSERVICE123",
        "type": "service_reference",
        "summary": "Test Service"
      }
    }
  }
}' http://localhost:5001/pd-webhook

# Test note/annotation from PagerDuty
curl -X POST -H "Content-Type: application/json" -d '{
  "event": {
    "id": "01G3PQ97QLWWR8SGW2AT6K18HG",
    "event_type": "incident.annotated",
    "resource_type": "incident",
    "occurred_at": "2025-10-27T15:40:48.576Z",
    "agent": {
      "id": "PAGENT123",
      "type": "user_reference",
      "self": "https://api.pagerduty.com/users/PAGENT123",
      "html_url": "https://example.pagerduty.com/users/PAGENT123",
      "summary": "Test User"
    },
    "client": null,
    "data": {
      "id": "PDI123",
      "type": "incident",
      "status": "acknowledged",
      "incident_key": "test-123",
      "title": "Test incident",
      "service": {
        "id": "PSERVICE123",
        "type": "service_reference",
        "summary": "Test Service"
      }
    },
    "note": {
      "id": "NOTE123",
      "content": "This is a test note from PagerDuty that should be synced to Opsgenie",
      "created_at": "2025-10-27T15:40:48.576Z",
      "user": {
        "id": "PAGENT123",
        "type": "user_reference",
        "summary": "Test User"
      }
    }
  }
}' http://localhost:5001/pd-webhook
```

## Exposing the Server

For webhooks to reach your server, you'll need to either:

- Deploy the application to a server with a public IP
- Use a service like ngrok for local testing: `ngrok http 5001`

## Security Considerations

This implementation includes some basic security features:

- PagerDuty webhook signature verification
- Environment variables for API keys and secrets

For production use, consider additional measures:

- Adding authentication to the Opsgenie webhook endpoint
- Using HTTPS
- Implementing IP whitelisting
- Securing all API keys and secrets
- Implementing rate limiting

## License

[Apache 2.0 License](LICENSE)
