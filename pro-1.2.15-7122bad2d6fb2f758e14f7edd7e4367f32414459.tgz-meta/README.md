# Giant Swarm Portfolio Roadmap Organizer (PRO)

An MCP server for managing Giant Swarm's project boards on GitHub Projects V2.

## Overview

PRO exposes Giant Swarm project boards to AI assistants via the [Model Context Protocol](https://modelcontextprotocol.io/). It supports multiple boards:

- **Roadmap Board** ([#273](https://github.com/orgs/giantswarm/projects/273)) -- Company quarterly goals (rocks), team/SIG/WG issues
- **Customer Board** ([#345](https://github.com/orgs/giantswarm/projects/345)) -- Customer-related activities and requests from private customer repositories

The server provides tools for listing, filtering, creating, and updating issues on either board, along with per-board schema resources that describe the available fields and their valid values. Field filtering is generic -- the LLM reads the board's schema resource to discover available fields, then passes arbitrary field name/value pairs as filters.

## Requirements

- Node.js 20+
- GitHub Personal Access Token with `project:write` and `repo:write` scopes

## Installation

```bash
git clone https://github.com/giantswarm/pro.git
cd pro
npm install
```

To make the `pro` command available globally (required for the MCP client config and self-update), install it into your user path -- no `sudo` needed:

**Option A: Using nvm or fnm (recommended)**

If you manage Node.js with [nvm](https://github.com/nvm-sh/nvm) or [fnm](https://github.com/Schniz/fnm), npm's global prefix is already in your home directory:

```bash
npm install -g .
```

**Option B: Configure npm's global prefix**

Without a version manager, npm's default global prefix is a system path. Redirect it to a user-local directory:

```bash
npm config set prefix ~/.local
```

Make sure `~/.local/bin` is in your `PATH` (add to `~/.bashrc` or `~/.zshrc`):

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Then install:

```bash
npm install -g .
```

**Option C: Use the full path in MCP config**

If you prefer not to install globally, point your MCP client config directly at the checkout:

```json
{
  "mcpServers": {
    "giantswarm-pro": {
      "command": "node",
      "args": ["/path/to/pro/bin/index.js"],
      "env": {
        "GITHUB_API_TOKEN": "your-github-token"
      }
    }
  }
}
```

Note: this skips the global binary, so `pro --self-update` won't be available.

## Configuration

### Environment Variables

- `GITHUB_API_TOKEN` (required): GitHub PAT with `project:write` and `repo:write` scopes.
- `HTTP_PORT` (optional): Port for HTTP transport (default: 8080).

## Transport Modes

The server supports two transport modes, selected via the `--transport` flag:

### stdio (default)

For local AI client integration (Claude Desktop, Cursor, Continue):

```bash
node bin/index.js
# or explicitly:
node bin/index.js --transport=stdio
```

### Streamable HTTP

For remote deployment (Kubernetes, Docker):

```bash
node bin/index.js --transport=streamable-http
```

This starts an HTTP server on port 8080 (configurable via `HTTP_PORT`) with:
- `POST/GET/DELETE /mcp` -- MCP streamable HTTP endpoint
- `GET /healthz` -- Liveness probe (always 200)
- `GET /readyz` -- Readiness probe (200 when MCP server is connected)

### MCP Client Configuration (stdio)

Add to your MCP client config (e.g., Claude Desktop, Cursor, Continue):

```json
{
  "mcpServers": {
    "giantswarm-pro": {
      "command": "pro",
      "args": [],
      "env": {
        "GITHUB_API_TOKEN": "your-github-token"
      }
    }
  }
}
```

See `mcp-config-example.json` for a template.

**Claude Desktop config locations:**
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

### Kubernetes Deployment

A Helm chart is available for deploying PRO in Kubernetes with the streamable HTTP transport.

**Prerequisites:**

1. Create a namespace:
   ```bash
   kubectl create namespace mcp-pro
   ```

2. Create a secret with your GitHub token (requires `project:write` and `repo:write` scopes):
   ```bash
   kubectl -n mcp-pro create secret generic pro-github-token \
     --from-literal=token=your-github-token
   ```

**Install:**

```bash
helm install pro ./helm/pro -n mcp-pro
```

**Verify:**

```bash
kubectl -n mcp-pro get pods
kubectl -n mcp-pro port-forward svc/pro 8080:8080
curl http://localhost:8080/healthz   # {"status":"ok"}
curl http://localhost:8080/readyz    # {"status":"ready"}
```

The Helm chart configures `--transport=streamable-http` automatically. The deployment includes liveness (`/healthz`) and readiness (`/readyz`) probes, runs as a non-root user, and enforces a read-only root filesystem.

**Key Helm values:**

| Value | Default | Description |
|-------|---------|-------------|
| `image.repository` | `gsoci.azurecr.io/giantswarm/pro` | Container image registry and name |
| `image.tag` | `v1.2.15` | Image tag |
| `service.port` | `8080` | Service port |
| `env` | GitHub token from secret | Environment variables for the container |
| `ingress.enabled` | `false` | Enable ingress (disabled by default) |

### Docker

```bash
docker run -d \
  -p 8080:8080 \
  -e GITHUB_API_TOKEN=your-github-token \
  gsoci.azurecr.io/giantswarm/pro:v1.2.15
```

The container defaults to `--transport=streamable-http`.

## Boards

| Board | Project | Description |
|-------|---------|-------------|
| `roadmap` | [#273](https://github.com/orgs/giantswarm/projects/273) | Company quarterly goals, team/SIG/WG issues |
| `customer` | [#345](https://github.com/orgs/giantswarm/projects/345) | Customer activities and requests from private repos |

The `board` parameter defaults to `"roadmap"` on all tools for backward compatibility.

## MCP Tools

### Read Tools

- **`list_issues`** -- List and filter issues from a project board using generic field filters. Specify `board` to choose the board (`"roadmap"` or `"customer"`). Use `filters` (a field name-to-value map) to filter by any single-select field. Use `emptyFields` to find items missing specific field values. Read the board's schema resource first to discover available fields and valid options.

- **`get_issue_details`** -- Get full details for a specific item: repository metadata, title, body text, comments with timestamps, assignees, labels, dates, and all field values. Board-independent (queries by item ID).

### Write Tools

- **`update_issue_field`** -- Update a single-select field value. Provide human-readable field and option names; the server resolves them to GitHub node IDs. Specify `board` to target the correct board.

- **`create_issue_in_project`** -- Create a new GitHub Issue in a repository and add it to a board. Optionally set initial status and assignees. Public issues in `giantswarm/roadmap` require `confirmPublicSafe=true`.

- **`add_existing_issue`** -- Add an existing GitHub issue to a board by URL or node ID.

- **`archive_item`** -- Archive a project item from the active board view.

## MCP Resources

Resources are provided per board. Read the schema resource before querying to understand the available fields and their valid option values.

| Resource | Description |
|----------|-------------|
| `roadmap://schema` | Field schema for the Roadmap Board |
| `roadmap://overview` | Stats and status distribution for the Roadmap Board |
| `customer://schema` | Field schema for the Customer Board |
| `customer://overview` | Stats and status distribution for the Customer Board |

## Repository Safety Guidance

- `giantswarm/roadmap` is public. Use it only for sanitized, public-safe issue content.
- `giantswarm/giantswarm` is internal/private. Use it for internal and customer-specific operational details.
- Customer board issues live in private customer repositories (e.g. `giantswarm/<customer>`). The board itself is a public GitHub project, but issue content is only visible to users with repository access.
- The server enforces an explicit confirmation (`confirmPublicSafe=true`) before creating issues in `giantswarm/roadmap`.

## Example Conversations

**Roadmap -- Product Owner Workflow:**
```
You: "Show me all roadmap issues for Team Honey Badger that are missing a function field"
AI:  [Reads roadmap://schema, then uses list_issues with board="roadmap", filters={"Team": "Honey Badger"}, emptyFields=["Function"]]

You: "Set the function for issue PVTI_xxx to 'Security'"
AI:  [Uses update_issue_field with board="roadmap"]
```

**Customer Board -- Account Engineer Workflow:**
```
You: "Show me all customer issues that are blocked"
AI:  [Reads customer://schema, then uses list_issues with board="customer", filters={"Status": "Blocked"}]

You: "What customer issues are assigned to Team Tenet?"
AI:  [Uses list_issues with board="customer", filters={"Team": "Tenet"}]
```

**Cross-board Workflow:**
```
You: "Create a new feature request in the customer repo and add it to the customer board"
AI:  [Uses create_issue_in_project with board="customer"]

You: "Add issue https://github.com/giantswarm/example/issues/42 to the roadmap board"
AI:  [Uses add_existing_issue with board="roadmap"]
```

## Development

```bash
# Start the MCP server locally (stdio)
node bin/index.js

# Start with HTTP transport
node bin/index.js --transport=streamable-http
```

## License

Copyright (c) 2025 Giant Swarm GmbH. All Rights Reserved.
