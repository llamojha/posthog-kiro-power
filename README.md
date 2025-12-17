# PostHog Kiro Power

A [Kiro Power](https://kiro.dev/docs/powers/) for interacting with [PostHog](https://posthog.com) analytics directly from your IDE.

## Features

- **Insights & Analytics** - Run trends, funnels, and HogQL queries
- **Feature Flags** - Create, update, and manage feature flags
- **Experiments** - Set up and analyze A/B tests
- **Surveys** - Create surveys and view response statistics
- **Error Tracking** - List and investigate application errors
- **Dashboards** - Create and manage analytics dashboards
- **LLM Analytics** - Track AI/LLM costs across models

## Installation

### Prerequisites

1. A PostHog account at [posthog.com](https://posthog.com)
2. A Personal API Key with MCP Server preset:
   - Go to https://app.posthog.com/settings/user-api-keys?preset=mcp_server
   - Create a new key and copy it

### Install in Kiro

1. Open Kiro
2. Open the Powers panel (Command Palette → "Open Powers")
3. Click "Add Custom Power"
4. Choose one of:
   - **Local Directory**: Clone this repo and provide the path to `powers/posthog`
   - **Git Repository**: Use this repo URL (once published)
5. Update the API key in the power's `mcp.json`:
   ```json
   "POSTHOG_AUTH_HEADER": "Bearer phx_your_api_key_here"
   ```

## Usage

Once installed, Kiro will automatically activate the PostHog power when you ask about:

- Analytics, funnels, or user behavior
- Feature flags or experiments
- Surveys or user feedback
- Error tracking
- PostHog-related queries

### Example Prompts

- "Show me the signup funnel conversion for the last 7 days"
- "Create a feature flag for the new checkout flow at 10% rollout"
- "What errors are users seeing in production?"
- "Set up an A/B test for the pricing page"
- "How many daily active users do we have?"

## Configuration

The power uses PostHog's remote MCP server. Configuration is in `powers/posthog/mcp.json`:

```json
{
  "mcpServers": {
    "posthog": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote@latest",
        "https://mcp.posthog.com/mcp",
        "--header",
        "Authorization:${POSTHOG_AUTH_HEADER}"
      ],
      "env": {
        "POSTHOG_AUTH_HEADER": "Bearer YOUR_PERSONAL_API_KEY_HERE"
      }
    }
  }
}
```

If your client doesn't support Streamable HTTP, use the SSE endpoint instead:

```
https://mcp.posthog.com/sse
```

## Documentation

- [POWER.md](powers/posthog/POWER.md) - Main power documentation
- [steering/steering.md](powers/posthog/steering/steering.md) - Advanced HogQL queries and workflows
- [PostHog MCP Docs](https://posthog.com/docs/model-context-protocol) - Official PostHog MCP documentation

## Contributing

Contributions welcome! Please open an issue or PR.
