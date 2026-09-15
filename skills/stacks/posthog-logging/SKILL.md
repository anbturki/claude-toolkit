---
name: posthog-logging
description: Set up, audit, and fix PostHog logging via OTLP — pino transport, structured logs, request context, events, and user linking. Use when PostHog logs are missing, delayed, broken, or need setup from scratch.
user-invocable: true
argument-hint: "[server path, PostHog issue, or 'setup' for fresh integration]"
---

# PostHog Logging & Events

Set up or fix PostHog observability. Covers OTLP log ingestion, server-side events, and user profile linking.

## Phase 1: Assess Current State

Before changing anything:

1. **Check if PostHog SDK is installed** — search for `posthog-node` in package.json
2. **Check if OTLP transport is installed** — search for `pino-opentelemetry-transport` in package.json
3. **Find the logger** — search for pino/winston/bunyan setup in the codebase
4. **Check env vars** — find `POSTHOG_API_KEY` (must start with `phc_`), `POSTHOG_HOST`
5. **Check infrastructure** — read Terraform/Docker/k8s configs for env injection, log drivers
6. **Query PostHog directly** — use PostHog MCP tools (`logs-query`, `logs-list-attributes`) to check if logs are arriving
7. **Check CloudWatch/stdout** — use AWS CLI or `docker logs` to verify logs are emitting at all

## Phase 2: PostHog OTLP Log Transport

### Required packages
```
pino
pino-opentelemetry-transport
posthog-node  (for events, not logs)
```

### OTLP Configuration (env var approach — recommended)
The transport reads standard OpenTelemetry env vars. Set these before the logger initializes:

```
OTEL_EXPORTER_OTLP_LOGS_ENDPOINT=https://us.i.posthog.com/i/v1/logs
OTEL_EXPORTER_OTLP_LOGS_HEADERS=Authorization=Bearer <phc_project_token>
OTEL_EXPORTER_OTLP_LOGS_PROTOCOL=http
```

- EU cloud: `https://eu.i.posthog.com/i/v1/logs`
- US cloud: `https://us.i.posthog.com/i/v1/logs`
- Auth: `Bearer <phc_...>` (project API key, NOT personal API key)

### Pino Transport Setup
```typescript
// Set OTEL env vars BEFORE creating the transport
if (!process.env.OTEL_EXPORTER_OTLP_LOGS_ENDPOINT) {
  process.env.OTEL_EXPORTER_OTLP_LOGS_ENDPOINT = 'https://us.i.posthog.com/i/v1/logs'
  process.env.OTEL_EXPORTER_OTLP_LOGS_HEADERS = `Authorization=Bearer ${POSTHOG_API_KEY}`
  process.env.OTEL_EXPORTER_OTLP_LOGS_PROTOCOL = 'http'
}

const transport = pino.transport({
  targets: [
    {
      target: 'pino-opentelemetry-transport',
      level: 'info',
      options: {
        resourceAttributes: {
          'service.name': 'my-service',
          'deployment.environment': process.env.NODE_ENV,
        },
      },
    },
    { target: 'pino/file', level: 'info' },  // stdout for CloudWatch/Docker
  ],
})
```

### Critical: Transport Resilience
`pino.transport({ targets })` runs in a single worker thread. If the OTLP transport fails to initialize, **stdout dies too** — zero logs to CloudWatch. Always wrap in try/catch:

```typescript
try {
  const transport = pino.transport({ targets: [...] })
  transport.on('error', (err) => console.error('[logger] transport error:', err.message))
  return pino(opts, transport)
} catch (err) {
  console.error('[logger] OTLP transport failed, falling back to stdout:', err)
  return pino(opts)  // plain stdout, no worker thread
}
```

### Batch Tuning
The default `BatchLogRecordProcessor` buffers for 5 seconds. To reduce delay:

```
OTEL_BLRP_SCHEDULE_DELAY=1000   # flush every 1s instead of 5s
```

Add this to ECS task definition / Docker env / k8s deployment.

### Shutdown
Pino's `onExit` handler tries to synchronously flush the OTLP transport, which can block for 10s and crash. Always flush async before exit:

```typescript
function shutdown() {
  log.info('Shutting down...')
  app.stop()
    .then(() => Promise.all([closeDb(), posthog?.shutdown()]))
    .then(() => new Promise<void>((resolve) => log.flush(() => resolve())))
    .finally(() => process.exit(0))
}
```

## Phase 3: User Linking (posthog_distinct_id)

To link logs to PostHog user profiles, include `posthog_distinct_id` as a log attribute:

```typescript
// In your pino mixin (via AsyncLocalStorage):
if (store.userId) {
  ctx.userId = store.userId
  ctx.posthog_distinct_id = store.userId  // PostHog uses this to link to person profiles
}
```

The user must also be identified via `posthog.identify()` or `posthog.capture()` with person properties for the profile to exist.

## Phase 4: Server-Side Events (posthog-node)

Separate from logs — use `posthog-node` for product analytics events:

```typescript
import { PostHog } from 'posthog-node'

const posthog = new PostHog('<phc_project_token>', {
  host: 'https://us.i.posthog.com',
})

// Capture events
posthog.capture({ distinctId: userId, event: 'wallet_topup', properties: { amount } })

// MUST call on shutdown
await posthog.shutdown()
```

### Event Plugin Pattern (Elysia/Express)
Fire events in `onAfterResponse` hooks — don't block request handlers:

```typescript
export const posthogPlugin = new Elysia({ name: 'posthog' })
  .onAfterResponse({ as: 'global' }, (ctx) => {
    if (!posthog || !ctx.user?.id) return
    posthog.capture({
      distinctId: ctx.user.id,
      event: 'api_request',
      properties: { method: ctx.request.method, path: ctx.path, status: ctx.set.status },
    })
  })
```

## Phase 5: Audit Log Coverage

For every source file in the server:

### Routes need:
- Business-context logs after meaningful operations (not request entry — the request logger handles that)
- Structured data: IDs, amounts, statuses, counts
- Warn/error for rejected requests with reason
- Error logs with full error object in catch blocks

### Libs need:
- Log before irreversible operations (external APIs, payments)
- Log after successful completion with result context
- Debug-level for cache hits and routine lookups
- Info-level for creates, updates, deletes
- Error-level for unexpected failures

### Avoid duplicate logging
- **Libs** own: operation details, external API calls, DB mutations
- **Routes** own: business decisions, rejections, user-facing errors
- **Plugins** own: cross-cutting (auth, rate limiting, request lifecycle)

If a lib logs the operation, the route should NOT re-log the same data.

### Request Context (AsyncLocalStorage)
Every log line should automatically include:
- `requestId` — UUID per request
- `clientIp` — from X-Forwarded-For
- `userId` / `userEmail` — enriched after auth
- `posthog_distinct_id` — same as userId, for PostHog person linking

## Phase 6: Verify End-to-End

### Check PostHog is receiving logs
Use PostHog MCP tools:
```
logs-query: dateFrom=<today>, serviceNames=["my-service"], limit=10
logs-list-attributes: attributeType="log" — verify custom attributes appear
```

### Check CloudWatch/stdout
```bash
aws logs tail /ecs/<service> --follow --region <region>
```

### Check for silent failures
- Search CloudWatch for `[logger] transport error` — indicates OTLP transport issues
- Search for `_flushSync took too long` — indicates shutdown crash
- If zero pino JSON lines in CloudWatch but server is healthy — transport worker thread died

## Known PostHog Limitations

- **Live Tail is buggy** — [PostHog/posthog#43467](https://github.com/PostHog/posthog/issues/43467) (Dec 2025, still open). Events show up in processed feed but live tail misses them. No client-side fix.
- **Ingestion delay** — ~1-5s client-side batch + PostHog pipeline. Reduce with `OTEL_BLRP_SCHEDULE_DELAY=1000`.
- **Protocol** — Use `http` (JSON format), not `http/protobuf`. PostHog may not accept protobuf on all endpoints.

## Output Format

```
[STATUS] file:line — Description
  Current: <what exists>
  Fix: <what to change>
  Why: <brief reason>
```

Statuses: **MISSING**, **BROKEN**, **NOISY**, **LEAK**, **OK**

### Summary
"Audited N files. PostHog transport: [WORKING/BROKEN]. Log coverage: X missing, Y broken, Z noisy."
