---
name: "posthog"
displayName: "PostHog"
description: "Query and manage PostHog analytics, feature flags, experiments, surveys, and error tracking. Debug user behavior, analyze funnels, and run A/B tests directly from your IDE."
keywords:
  [
    "posthog",
    "analytics",
    "product analytics",
    "feature flags",
    "experiments",
    "ab testing",
    "surveys",
    "error tracking",
    "funnel",
    "insights",
  ]
author: "Alvaro Llamojha"
---

# PostHog

## Overview

The PostHog Power provides comprehensive access to your PostHog analytics platform directly from Kiro. Query user behavior, manage feature flags, run experiments, analyze funnels, and track errors without leaving your IDE.

**Key capabilities:**

- **Insights & Analytics**: Query trends, funnels, and run HogQL queries on your data
- **Feature Flags**: Create, update, and manage feature flags for gradual rollouts
- **Experiments**: Set up and analyze A/B tests with statistical results
- **Surveys**: Create and analyze user surveys with response statistics
- **Error Tracking**: List and investigate errors in your application
- **Dashboards**: Create and manage analytics dashboards
- **Events & Properties**: Explore event definitions and properties in your project
- **LLM Analytics**: Track AI/LLM costs and usage across models
- **Documentation**: Search PostHog docs for setup and best practices

**Authentication**: Requires a PostHog Personal API Key.

## Available Steering Files

This power has the following steering file:

- **steering** - Comprehensive query syntax guide with HogQL examples, workflows, and troubleshooting

## Available MCP Servers

### posthog

**Package:** `mcp-remote` + `https://mcp.posthog.com/mcp`
**Connection:** Remote MCP server via npx with Bearer token authentication

**Tools by Category:**

#### Dashboards

| Tool                       | Purpose                                            |
| -------------------------- | -------------------------------------------------- |
| `dashboard-create`         | Create a new dashboard in the project              |
| `dashboard-get`            | Get a specific dashboard by ID, including insights |
| `dashboard-update`         | Update an existing dashboard by ID                 |
| `dashboard-delete`         | Delete a dashboard by ID                           |
| `dashboards-get-all`       | Get all dashboards with optional filtering         |
| `add-insight-to-dashboard` | Add an existing insight to a dashboard             |

#### Insights & Analytics

| Tool                                 | Purpose                                       |
| ------------------------------------ | --------------------------------------------- |
| `query-run`                          | Run a trend, funnel, or HogQL query           |
| `query-generate-hogql-from-question` | Generate HogQL from natural language question |
| `insight-create-from-query`          | Save a query as an insight                    |
| `insight-get`                        | Get a specific insight by ID                  |
| `insight-query`                      | Execute a query on an existing insight        |
| `insight-update`                     | Update an existing insight by ID              |
| `insight-delete`                     | Delete an insight by ID                       |
| `insights-get-all`                   | Get all insights with optional filtering      |

#### Feature Flags

| Tool                          | Purpose                              |
| ----------------------------- | ------------------------------------ |
| `create-feature-flag`         | Create a new feature flag            |
| `feature-flag-get-all`        | Get all feature flags in the project |
| `feature-flag-get-definition` | Get the definition of a feature flag |
| `update-feature-flag`         | Update a feature flag                |
| `delete-feature-flag`         | Delete a feature flag                |

#### Experiments

| Tool                     | Purpose                                                  |
| ------------------------ | -------------------------------------------------------- |
| `experiment-create`      | Create A/B test with guided metric and flag setup        |
| `experiment-get`         | Get details of a specific experiment                     |
| `experiment-get-all`     | Get all experiments in the project                       |
| `experiment-results-get` | Get comprehensive results including metrics and exposure |
| `experiment-update`      | Update experiment with lifecycle management              |
| `experiment-delete`      | Delete an experiment by ID                               |

#### Surveys

| Tool                   | Purpose                                       |
| ---------------------- | --------------------------------------------- |
| `survey-create`        | Create a new survey                           |
| `survey-get`           | Get a specific survey by ID                   |
| `survey-update`        | Update an existing survey                     |
| `survey-delete`        | Delete a survey by ID                         |
| `surveys-get-all`      | Get all surveys with optional filtering       |
| `survey-stats`         | Get response statistics for a specific survey |
| `surveys-global-stats` | Get aggregated stats across all surveys       |

#### Error Tracking

| Tool            | Purpose                         |
| --------------- | ------------------------------- |
| `list-errors`   | List errors in the project      |
| `error-details` | Get details of a specific error |

#### Events & Properties

| Tool                     | Purpose                                            |
| ------------------------ | -------------------------------------------------- |
| `event-definitions-list` | List all event definitions with optional filtering |
| `properties-list`        | Get properties for events or persons               |
| `property-definitions`   | Get event and property definitions for the project |

#### Organization & Project Management

| Tool                       | Purpose                                  |
| -------------------------- | ---------------------------------------- |
| `organizations-get`        | Get organizations the user has access to |
| `organization-details-get` | Get details of the active organization   |
| `projects-get`             | Get projects in the current organization |
| `switch-organization`      | Change the active organization           |
| `switch-project`           | Change the active project                |

#### LLM Analytics

| Tool                              | Purpose                                |
| --------------------------------- | -------------------------------------- |
| `get-llm-total-costs-for-project` | Get daily LLM costs by model over time |

#### Documentation

| Tool          | Purpose                      |
| ------------- | ---------------------------- |
| `docs-search` | Search PostHog documentation |

## Tool Usage Examples

### Running Queries

**Run a trend query:**

```javascript
usePower("posthog", "posthog", "query-run", {
  query_type: "trend",
  event: "pageview",
  date_from: "-7d",
});
// Returns: Pageview trend over the last 7 days
```

**Run a funnel query:**

```javascript
usePower("posthog", "posthog", "query-run", {
  query_type: "funnel",
  events: ["signup_started", "signup_completed", "first_purchase"],
  date_from: "-30d",
});
// Returns: Conversion rates through the funnel
```

**Generate HogQL from natural language:**

```javascript
usePower("posthog", "posthog", "query-generate-hogql-from-question", {
  question: "How many unique users signed up last week?",
});
// Returns: HogQL query and results
```

### Managing Feature Flags

**Create a feature flag:**

```javascript
usePower("posthog", "posthog", "create-feature-flag", {
  key: "new-checkout-flow",
  name: "New Checkout Flow",
  rollout_percentage: 10,
});
// Returns: Created feature flag details
```

**Get all feature flags:**

```javascript
usePower("posthog", "posthog", "feature-flag-get-all", {});
// Returns: List of all feature flags in the project
```

**Update a feature flag rollout:**

```javascript
usePower("posthog", "posthog", "update-feature-flag", {
  id: "new-checkout-flow",
  rollout_percentage: 50,
});
// Returns: Updated feature flag
```

### Running Experiments

**Create an A/B test:**

```javascript
usePower("posthog", "posthog", "experiment-create", {
  "name": "Checkout Button Color Test",
ature_flag_key": "checkout-button-color",
  "description": "Testing blue vs green checkout button"
})
// Returns: Created experiment with variants
```

**Get experiment results:**

```javascript
usePower("posthog", "posthog", "experiment-results-get", {
  experiment_id: 123,
});
// Returns: Statistical results, conversion rates, and exposure data
```

### Managing Surveys

**Create a survey:**

```javascript
usePower("posthog", "posthog", "survey-create", {
  name: "NPS Survey",
  type: "popover",
  questions: [
    {
      type: "rating",
      question: "How likely are you to recommend us?",
      scale: 10,
    },
  ],
});
// Returns: Created survey details
```

**Get survey statistics:**

```javascript
usePower("posthog", "posthog", "survey-stats", {
  survey_id: "abc123",
});
// Returns: Response counts, completion rates, answer distributions
```

### Error Tracking

**List recent errors:**

```javascript
usePower("posthog", "posthog", "list-errors", {});
// Returns: List of errors with counts and last seen timestamps
```

**Get error details:**

```javascript
usePower("posthog", "posthog", "error-details", {
  error_id: "error-fingerprint-123",
});
// Returns: Full error details, stack trace, affected users
```

### Dashboard Management

**Create a dashboard:**

```javascript
usePower("posthog", "posthog", "dashboard-create", {
  name: "Product Metrics",
  description: "Key product health metrics",
});
// Returns: Created dashboard
```

**Add insight to dashboard:**

```javascript
usePower("posthog", "posthog", "add-insight-to-dashboard", {
  dashboard_id: 123,
  insight_id: 456,
});
// Returns: Updated dashboard with new insight
```

### LLM Cost Tracking

**Get LLM costs by model:**

```javascript
usePower("posthog", "posthog", "get-llm-total-costs-for-project", {
  days: 30,
});
// Returns: Daily costs broken down (GPT-4, Claude, etc.)
```

## Combining Tools (Workflows)

### Workflow 1: Feature Flag Rollout with Monitoring

```javascript
// Step 1: Create the feature flag
const flag = usePower("posthog", "posthog", "create-feature-flag", {
  key: "new-pricing-page",
  name: "New Pricing Page",
  rollout_percentage: 5,
});

// Step 2: Create an experiment to track impact
const experiment = usePower("posthog", "posthog", "experiment-create", {
  name: "New Pricing Page Impact",
  feature_flag_key: "new-pricing-page",
  description: "Measuring conversion impact of new pricing page",
});

// Step 3: After some time, check results
const results = usePower("posthog", "posthog", "experiment-results-get", {
  experiment_id: experiment.id,
});

// Step 4: If positive, increase rollout
usePower("posthog", "posthog", "update-feature-flag", {
  id: "new-pricing-page",
  rollout_percentage: 50,
});
```

### Workflow 2: User Behavior Investigation

```javascript
// Step 1: Query signup funnel
const funnel = usePower("posthog", "posthog", "query-run", {
  query_type: "funnel",
  events: ["page_view", "signup_click", "signup_complete"],
  date_from: "-7d",
});

// Step 2: Find where users drop off using HogQL
const dropoff = usePower(
  "posthog",
  "posthog",
  "query-generate-hogql-from-question",
  {
    question: "What pages do users visit before abandoning signup?",
  }
);

// Step 3: Check for errors during signup
const errors = usePower("posthog", "posthog", "list-errors", {});

// Step 4: Get details on signup-related errors
const errorDetails = usePower("posthog", "posthog", "error-details", {
  error_id: errors[0].id,
});

// Step 5: Save the funnel as an insight for monitoring
usePower("posthog", "posthog", "insight-create-from-query", {
  name: "Signup Funnel",
  query: funnel.query,
});
```

### Workflow 3: Product Launch Dashboard

```javascript
// Step 1: Create a dashboard for the launch
const dashboard = usePower("posthog", "posthog", "dashboard-create", {
  name: "Feature X Launch Metrics",
  description: "Tracking Feature X adoption and impact",
});

// Step 2: Create adoption trend insight
const adoptionInsight = usePower(
  "posthog",
  "posthog",
  "insight-create-from-query",
  {
    name: "Feature X Daily Active Users",
    query_type: "trend",
    event: "feature_x_used",
  }
);

// Step 3: Add to dashboard
usePower("posthog", "posthog", "add-insight-to-dashboard", {
  dashboard_id: dashboard.id,
  insight_id: adoptionInsight.id,
});

// Step 4: Create a survey for feedback
usePower("posthog", "posthog", "survey-create", {
  name: "Feature X Feedback",
  type: "popover",
  targeting_flag_key: "feature-x-enabled",
  questions: [
    {
      type: "rating",
      question: "How useful is Feature X?",
      scale: 5,
    },
  ],
});
```

### Workflow 4: Error Investigation

```javascript
// Step 1: List recent errors
const errors = usePower("posthog", "posthog", "list-errors", {});

// Step 2: Get details on the most frequent error
const errorInfo = usePower("posthog", "posthog", "error-details", {
  error_id: errors[0].id,
});

// Step 3: Query affected users
const affectedUsers = usePower(
  "posthog",
  "posthog",
  "query-generate-hogql-from-question",
  {
    question:
      "How many unique users experienced the checkout error in the last 24 hours?",
  }
);

// Step 4: Check if there's a pattern with feature flags
const flags = usePower("posthog", "posthog", "feature-flag-get-all", {});

// Step 5: Search docs for error handling best practices
const docs = usePower("posthog", "posthog", "docs-search", {
  query: "error tracking setup",
});
```

## HogQL Query Guide

PostHog uses HogQL, a SQL-like query language for analytics. Here's a quick reference:

### Basic Query Structure

```sql
SELECT
    count() as total_events,
    uniq(distinct_id) as unique_users
FROM events
WHERE
    event = 'pageview'
    AND timestamp > now() - INTERVAL 7 DAY
```

### Common Patterns

**Count events by day:**

```sql
SELECT
    toDate(timestamp) as day,
    count() as events
FROM events
WHERE event = 'signup'
GROUP BY day
ORDER BY day
```

**Unique users by property:**

```sql
SELECT
    properties.$browser as browser,
    uniq(distinct_id) as users
FROM events
WHERE event = 'pageview'
GROUP BY browser
ORDER BY users DESC
```

**Funnel analysis:**

```sql
SELECT
    countIf(event = 'page_view') as step1,
    countIf(event = 'add_to_cart') as step2,
    countIf(event = 'purchase') as step3
FROM events
WHERE timestamp > now() - INTERVAL 30 DAY
```

### Useful Functions

| Function                | Purpose             | Example                                  |
| ----------------------- | ------------------- | ---------------------------------------- |
| `count()`               | Count events        | `count()`                                |
| `uniq(field)`           | Count unique values | `uniq(distinct_id)`                      |
| `sum(field)`            | Sum numeric values  | `sum(properties.amount)`                 |
| `avg(field)`            | Average value       | `avg(properties.duration)`               |
| `toDate(timestamp)`     | Extract date        | `toDate(timestamp)`                      |
| `dateDiff('day', a, b)` | Days between dates  | `dateDiff('day', first_seen, timestamp)` |

### Accessing Properties

- **Event properties:** `properties.property_name` or `properties.$browser`
- **Person properties:** `person.properties.email`
- **System properties:** Use `$` prefix like `properties.$current_url`

## Best Practices

### ✅ Do:

- **Start with natural language queries** - Use `query-generate-hogql-from-question` first
- **Use date filters** - Always scope queries with `date_from` to avoid scanning all data
- **Create insights for repeated queries** - Save useful queries as insights
- **Use feature flags for gradual rollouts** - Start at 5-10% and increase
- **Set up experiments for A/B tests** - Get statistical significance before decisions
- **Monitor errors proactively** - Check `list-errors` regularly
- **Use dashboards for visibility** - Group related insights together
- **Search docs when stuck** - Use `docs-search` for setup and best practices

### ❌ Don't:

- **Query without date filters** - Expensive and slow
- **Roll out features to 100% immediately** - Use gradual rollouts
- **Ignore statistical significance** - Wait for experiment results to stabilize
- **Create duplicate insights** - Check existing insights first
- **Hardcode API keys** - Use environment variables

## Troubleshooting

### Error: "Authentication failed"

**Cause:** Invalid or expired API key
**Solution:**

1. Go to PostHog → Settings → Personal API Keys
2. Create a new key with MCP Server preset
3. Update `POSTHOG_AUTH_HEADER` in your config

### Error: "Project not found"

**Cause:** API key doesn't have access to the project
**Solution:**

1. Verify the API key has access to the correct project
2. Use `switch-project` to change active project
3. Check `projects-get` to see available projects

### Error: "Query timeout"

**Cause:** Query too expensive or time range too large
**Solution:**

1. Add date filters to narrow the time range
2. Add more specific event filters
3. Use sampling for large datasets

### Error: "Feature flag not found"

**Cause:** Flag key doesn't exist or typo in key name
**Solution:**

1. Use `feature-flag-get-all` to list existing flags
2. Check for typos in the flag key
3. Verify you're in the correct project

### Error: "Rate limit exceeded"

**Cause:** Too many API requests
**Solution:**

1. Add delays between requests
2. Batch operations where possible
3. Cache results for repeated queries

## Configuration

**Authentication Required**: PostHog Personal API Key

**Setup Steps:**

1. **Get PostHog API Key:**

   - Log in to PostHog at https://app.posthog.com
   - Go to Settings → Personal API Keys
   - Click "Create personal API key"
   - Select the "MCP Server" preset (or manually select required scopes)
   - Copy the generated key

2. **Configure in mcp.json:**

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
           "POSTHOG_AUTH_HEADER": "Bearer phx_your_api_key_here"
         }
       }
     }
   }
   ```

3. **Alternative SSE endpoint** (if your client doesn't support Streamable HTTP):
   ```json
   {
     "mcpServers": {
       "posthog": {
         "command": "npx",
         "args": [
           "-y",
           "mcp-remote@latest",
           "https://mcp.posthog.com/sse",
           "--header",
           "Authorization:${POSTHOG_AUTH_HEADER}"
         ],
         "env": {
           "POSTHOG_AUTH_HEADER": "Bearer phx_your_api_key_here"
         }
       }
     }
   }
   ```

## MCP Config Placeholders

**IMPORTANT:** Before using this power, replace the following placeholder in `mcp.json`:

- **`YOUR_PERSONAL_API_KEY_HERE`**: Your PostHog Personal API Key
  - **How to get it:**
    1. Go to https://app.posthog.com/settings/user-api-keys?preset=mcp_server
    2. Click "Create personal API key"
    3. Name it (e.g., "Kiro MCP")
    4. The MCP Server preset will auto-select required scopes
    5. Click "Create key" and copy the value (starts with `phx_`)
    6. Replace `YOUR_PERSONAL_API_KEY_HERE` with your key

**After replacing the placeholder, your config should look like:**

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
        "POSTHOG_AUTH_HEADER": "Bearer phx_abc123xyz..."
      }
    }
  }
}
```

## Tips

1. **Use natural language first** - `query-generate-hogql-from-question` is powerful
2. **Start with small rollouts** - Feature flags at 5-10% initially
3. **Wait for significance** - Don't end experiments early
4. **Create dashboards** - Group related metrics for easy monitoring
5. **Use surveys strategically** - Target specific user segments with feature flags
6. **Monitor errors daily** - Catch issues before users report them
7. **Search docs** - PostHog docs are comprehensive, use `docs-search`
8. **Switch projects** - Use `switch-project` when working across environments
9. **Save insights** - Don't re-run the same queries, save them
10. **Track LLM costs** - If using AI features, monitor with `get-llm-total-costs-for-project`

---

**Package:** `mcp-remote` + PostHog MCP Server
**Source:** Official PostHog
**Connection:** Remote MCP server with Bearer token authentication
**Docs:** https://posthog.com/docs/model-context-protocol
