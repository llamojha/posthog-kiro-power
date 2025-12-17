# PostHog Query Syntax & Advanced Workflows

This steering file provides comprehensive queryL patterns, and advanced workflows for the PostHog power.

---

## HogQL Deep Dive

HogQL is PostHog's SQL-like query language. It's based on ClickHouse SQL with PostHog-specific extensions.

### Data Model

**Core Tables:**

- `events` - All tracked events
- `persons` - User/person records
- `sessions` - Session data
- `groups` - Group analytics data

**Event Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `uuid` | String | Unique event ID |
| `event` | String | Event name |
| `timestamp` | DateTime | When event occurred |
| `distinct_id` | String | User identifier |
| `properties` | Object | Event properties |
| `person` | Object | Person data at event time |
| `person_id` | String | Person UUID |

### Property Access Patterns

**Event Properties:**

```sql
-- Standard properties
properties.button_name
properties.page_url
properties.amount

-- System properties ($ prefix)
properties.$browser
properties.$os
properties.$current_url
properties.$referrer
properties.$device_type

-- Nested properties
properties.user.plan
properties.order.items[0].name
```

**Person Properties:**

```sql
-- Access person properties
person.properties.email
person.properties.plan
person.properties.created_at
person.properties.$initial_referrer
```

### Time Functions

```sql
-- Current time
now()

-- Date extraction
toDate(timestamp)                    -- Extract date
toStartOfDay(timestamp)              -- Start of day
toStartOfWeek(timestamp)             -- Start of week
toStartOfMonth(timestamp)            -- Start of month
toHour(timestamp)                    -- Hour (0-23)
toDayOfWeek(timestamp)               -- Day of week (1-7)

-- Time intervals
timestamp > now() - INTERVAL 7 DAY
timestamp > now() - INTERVAL 1 HOUR
timestamp BETWEEN '2024-01-01' AND '2024-01-31'

-- Date differences
dateDiff('day', created_at, timestamp)
dateDiff('hour', session_start, timestamp)
```

### Aggregation Functions

```sql
-- Counting
count()                              -- Total count
count(DISTINCT field)                -- Distinct count
uniq(distinct_id)                    -- Unique users (optimized)
uniqExact(distinct_id)               -- Exact unique count

-- Math
sum(properties.amount)
avg(properties.duration)
min(properties.price)
max(properties.price)
median(properties.load_time)

-- Percentiles
quantile(0.5)(properties.duration)   -- Median
quantile(0.95)(properties.duration)  -- P95
quantile(0.99)(properties.duration)  -- P99

-- Conditional
countIf(event = 'purchase')
sumIf(properties.amount, event = 'purchase')
avgIf(properties.duration, properties.success = true)
```

### String Functions

```sql
-- Matching
event LIKE '%click%'
event ILIKE '%Click%'                -- Case insensitive
match(properties.$current_url, '.*checkout.*')  -- Regex

-- Extraction
substring(properties.url, 1, 100)
extract(properties.url, '/product/([0-9]+)')
splitByChar('/', properties.path)[2]

-- Transformation
lower(properties.email)
upper(properties.country)
trim(properties.name)
concat(properties.first_name, ' ', properties.last_name)
```

### Array Functions

```sql
-- Array operations
arrayJoin(properties.tags)           -- Expand array to rows
has(properties.features, 'premium')  -- Check if contains
length(properties.items)             -- Array length
arrayElement(properties.items, 1)    -- Get element (1-indexed)
```

---

## Advanced Query Patterns

### Retention Analysis

```sql
-- Weekly retention cohorts
SELECT
    toStartOfWeek(first_event) as cohort_week,
    dateDiff('week', first_event, timestamp) as weeks_since_signup,
    uniq(distinct_id) as users
FROM events
INNER JOIN (
    SELECT distinct_id, min(timestamp) as first_event
    FROM events
    WHERE event = 'signup'
    GROUP BY distinct_id
) cohorts ON events.distinct_id = cohorts.distinct_id
WHERE event = 'active'
GROUP BY cohort_week, weeks_since_signup
ORDER BY cohort_week, weeks_since_signup
```

### Funnel with Time Constraints

```sql
-- Users who completed funnel within 1 hour
SELECT
    count(DISTINCT step1.distinct_id) as started,
    count(DISTINCT step2.distinct_id) as completed,
    count(DISTINCT step2.distinct_id) / count(DISTINCT step1.distinct_id) as conversion_rate
FROM events step1
LEFT JOIN events step2 ON
    step1.distinct_id = step2.distinct_id
    AND step2.event = 'purchase'
    AND step2.timestamp > step1.timestamp
    AND step2.timestamp < step1.timestamp + INTERVAL 1 HOUR
WHERE step1.event = 'add_to_cart'
    AND step1.timestamp > now() - INTERVAL 30 DAY
```

### Session Analysis

```sql
-- Average session duration by source
SELECT
    properties.$initial_referrer_domain as source,
    avg(session_duration) as avg_duration,
    count() as sessions
FROM sessions
WHERE timestamp > now() - INTERVAL 7 DAY
GROUP BY source
ORDER BY sessions DESC
LIMIT 10
```

### User Segmentation

```sql
-- Power users (>10 events per day)
SELECT
    distinct_id,
    count() / 7 as daily_events,
    uniq(event) as unique_events
FROM events
WHERE timestamp > now() - INTERVAL 7 DAY
GROUP BY distinct_id
HAVING daily_events > 10
ORDER BY daily_events DESC
```

### Revenue Analysis

```sql
-- Daily revenue with running total
SELECT
    toDate(timestamp) as day,
    sum(properties.amount) as daily_revenue,
    sum(sum(properties.amount)) OVER (ORDER BY day) as cumulative_revenue
FROM events
WHERE event = 'purchase'
    AND timestamp > now() - INTERVAL 30 DAY
GROUP BY day
ORDER BY day
```

---

## Feature Flag Patterns

### Gradual Rollout Strategy

```javascript
// Phase 1: Internal testing (1%)
usePower("posthog", "posthog", "create-feature-flag", {
  key: "new-feature",
  name: "New Feature",
  rollout_percentage: 1,
  filters: {
    groups: [
      {
        properties: [
          { key: "email", value: "@yourcompany.com", operator: "icontains" },
        ],
      },
    ],
  },
});

// Phase 2: Beta users (10%)
usePower("posthog", "posthog", "update-feature-flag", {
  id: "new-feature",
  rollout_percentage: 10,
});

// Phase 3: General availability (50%)
usePower("posthog", "posthog", "update-feature-flag", {
  id: "new-feature",
  rollout_percentage: 50,
});

// Phase 4: Full rollout (100%)
usePower("posthog", "posthog", "update-feature-flag", {
  id: "new-feature",
  rollout_percentage: 100,
});
```

### Targeting Specific Users

```javascript
// Target by user property
usePower("posthog", "posthog", "create-feature-flag", {
  key: "premium-feature",
  filters: {
    groups: [
      {
        properties: [{ key: "plan", value: "premium", operator: "exact" }],
      },
    ],
  },
});

// Target by cohort
usePower("posthog", "posthog", "create-feature-flag", {
  key: "beta-feature",
  filters: {
    groups: [
      {
        properties: [{ key: "id", value: "cohort_id_here", type: "cohort" }],
      },
    ],
  },
});
```

### Multivariate Flags

```javascript
// A/B/C test with different variants
usePower("posthog", "posthog", "create-feature-flag", {
  key: "button-color-test",
  multivariate: {
    variants: [
      { key: "control", rollout_percentage: 34 },
      { key: "blue", rollout_percentage: 33 },
      { key: "green", rollout_percentage: 33 },
    ],
  },
});
```

---

## Experiment Best Practices

### Setting Up a Valid Experiment

```javascript
// 1. Create feature flag for the experiment
usePower("posthog", "posthog", "create-feature-flag", {
  key: "checkout-redesign-experiment",
  multivariate: {
    variants: [
      { key: "control", rollout_percentage: 50 },
      { key: "test", rollout_percentage: 50 },
    ],
  },
});

// 2. Create the experiment
usePower("posthog", "posthog", "experiment-create", {
  name: "Checkout Redesign Impact",
  feature_flag_key: "checkout-redesign-experiment",
  description: "Testing new checkout flow vs current",
  start_date: "2024-01-15",
  parameters: {
    minimum_detectable_effect: 0.05, // 5% lift
    recommended_sample_size: 10000,
  },
});

// 3. Monitor results (wait for statistical significance)
usePower("posthog", "posthog", "experiment-results-get", {
  experiment_id: 123,
});
```

### Interpreting Results

When checking experiment results, look for:

- **Statistical significance**: p-value < 0.05
- **Confidence interval**: Doesn't cross zero
- **Sample size**: Meets minimum requirement
- **Duration**: At least 1-2 weeks to account for weekly patterns

---

## Survey Patterns

### NPS Survey

```javascript
usePower("posthog", "posthog", "survey-create", {
  name: "NPS Survey Q1 2024",
  type: "popover",
  questions: [
    {
      type: "rating",
      question: "How likely are you to recommend us to a friend or colleague?",
      scale: 10,
      lowerBoundLabel: "Not likely",
      upperBoundLabel: "Very likely",
    },
    {
      type: "open",
      question: "What's the main reason for your score?",
    },
  ],
  conditions: {
    url: "/dashboard",
  },
  targeting_flag_key: "active-users",
});
```

### Feature Feedback Survey

```javascript
usePower("posthog", "posthog", "survey-create", {
  name: "New Feature Feedback",
  type: "popover",
  questions: [
    {
      type: "rating",
      question: "How useful do you find the new export feature?",
      scale: 5,
    },
    {
      type: "single_choice",
      question: "How often do you use this feature?",
      choices: ["Daily", "Weekly", "Monthly", "Rarely"],
    },
    {
      type: "open",
      question: "Any suggestions for improvement?",
    },
  ],
  targeting_flag_key: "new-export-feature",
});
```

---

## Error Tracking Workflows

### Prioritizing Errors

```javascript
// 1. List all errors sorted by frequency
const errors = usePower("posthog", "posthog", "list-errors", {});

// 2. For each high-frequency error, get details
for (const error of errors.slice(0, 5)) {
  const details = usePower("posthog", "posthog", "error-details", {
    error_id: error.id,
  });

  // 3. Query affected users
  const impact = usePower(
    "posthog",
    "posthog",
    "query-generate-hogql-from-question",
    {
      question: `How many unique users saw error "${error.message}" in the last 24 hours?`,
    }
  );
}
```

### Correlating Errors with Releases

```javascript
// Query errors by version
usePower("posthog", "posthog", "query-generate-hogql-from-question", {
  question: "Show error counts grouped by app version for the last 7 days",
});
```

---

## Dashboard Templates

### Product Health Dashboard

```javascript
// Create dashboard
const dashboard = usePower("posthog", "posthog", "dashboard-create", {
  name: "Product Health",
  description: "Key metrics for product health monitoring",
});

// Add DAU trend
const dauInsight = usePower("posthog", "posthog", "insight-create-from-query", {
  name: "Daily Active Users",
  query_type: "trend",
  event: "active",
  date_from: "-30d",
});
usePower("posthog", "posthog", "add-insight-to-dashboard", {
  dashboard_id: dashboard.id,
  insight_id: dauInsight.id,
});

// Add signup funnel
const funnelInsight = usePower(
  "posthog",
  "posthog",
  "insight-create-from-query",
  {
    name: "Signup Funnel",
    query_type: "funnel",
    events: ["page_view", "signup_start", "signup_complete"],
  }
);
usePower("posthog", "posthog", "add-insight-to-dashboard", {
  dashboard_id: dashboard.id,
  insight_id: funnelInsight.id,
});
```

---

## Common Query Templates

### User Acquisition

```sql
-- New users by source
SELECT
    properties.$initial_referrer_domain as source,
    count(DISTINCT distinct_id) as new_users
FROM events
WHERE event = 'signup'
    AND timestamp > now() - INTERVAL 30 DAY
GROUP BY source
ORDER BY new_users DESC
```

### Engagement Metrics

```sql
-- Weekly active users trend
SELECT
    toStartOfWeek(timestamp) as week,
    uniq(distinct_id) as wau
FROM events
WHERE timestamp > now() - INTERVAL 90 DAY
GROUP BY week
ORDER BY week
```

### Revenue Metrics

```sql
-- Monthly recurring revenue
SELECT
    toStartOfMonth(timestamp) as month,
    sum(properties.amount) as mrr,
    uniq(distinct_id) as paying_customers
FROM events
WHERE event = 'subscription_payment'
GROUP BY month
ORDER BY month
```

### Feature Adoption

```sql
-- Feature usage by user segment
SELECT
    person.properties.plan as plan,
    event,
    count() as usage_count,
    uniq(distinct_id) as unique_users
FROM events
WHERE event IN ('feature_a_used', 'feature_b_used', 'feature_c_used')
    AND timestamp > now() - INTERVAL 30 DAY
GROUP BY plan, event
ORDER BY plan, usage_count DESC
```

---

## Troubleshooting Guide

### Query Performance Issues

**Symptom:** Query takes too long or times out

**Solutions:**

1. Add time filters: `WHERE timestamp > now() - INTERVAL 7 DAY`
2. Add specific event filters: `WHERE event = 'specific_event'`
3. Reduce cardinality in GROUP BY
4. Use `uniq()` instead of `count(DISTINCT)`
5. Sample data for exploratory queries

### Missing Data

**Symptom:** Query returns no results

**Checklist:**

1. Verify event name spelling (case-sensitive)
2. Check time range includes expected data
3. Verify property names exist
4. Check project is correct (`switch-project`)
5. Verify data is being sent (check live events)

### Property Access Errors

**Symptom:** "Unknown column" or null values

**Solutions:**

1. Use correct syntax: `properties.name` not `properties['name']`
2. Check property exists in event definitions
3. Handle nulls: `coalesce(properties.name, 'unknown')`
4. Use `JSONExtractString` for complex nested access

---

## Security Considerations

- **API Key Scope**: Use minimal required scopes for your API key
- **Data Access**: Be mindful of PII in query results
- **Sharing**: Don't share API keys or query results containing user data
- **Audit**: PostHog logs API access for compliance

---

**Reference:** https://posthog.com/docs/hogql
