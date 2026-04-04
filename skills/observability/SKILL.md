---
name: observability
description: Audit and improve server observability — logging, log transports, request context, and structured log coverage. Use when logs are missing, broken, or insufficient for debugging production issues.
user-invocable: true
argument-hint: "[server path, log issue, or scope to audit]"
---

# Observability Audit

Systematic, evidence-based. No guessing.

## Phase 1: Understand Current State

Before changing anything, map the existing observability setup:

1. **Find the logger** — search for logger setup (pino, winston, bunyan, console)
2. **Trace log transports** — where do logs go? (stdout, file, CloudWatch, PostHog, Datadog, etc.)
3. **Check infrastructure** — read Terraform/Docker/k8s configs for log drivers, retention, log groups
4. **Read the log output** — use AWS CLI, `docker logs`, or local dev to see what's actually being emitted
5. **Map all routes and handlers** — list every endpoint, middleware, plugin, and background job

Do NOT proceed until you have a complete picture.

## Phase 2: Audit Log Coverage

For every file in the server `src/` directory (excluding `node_modules`):

### Routes — Every endpoint needs:
- Business-context log after meaningful operations (not request entry — the request logger handles that)
- Structured data: IDs, amounts, statuses, counts — anything needed to debug without reproducing
- Warn/error logs for rejected requests with the reason (ownership mismatch, insufficient balance, not found)
- Error logs with full error object for catch blocks

### Libs — Every function with side effects needs:
- Log before irreversible operations (external API calls, payments, purchases)
- Log after successful completion with result context
- Debug-level log for cache hits and routine lookups
- Info-level log for creates, updates, deletes
- Warn-level log for expected failures (race conditions, retries)
- Error-level log for unexpected failures

### Plugins/Middleware:
- Auth failures must log what was missing (header, cookie, session)
- Rate limit hits are logged by error handler — verify this
- Request logger must capture: method, path, status, duration, client IP, request ID

### Infrastructure:
- Verify log transport config matches the provider's expected format
- Check OTLP endpoints, auth headers, protocols
- Verify env vars are set in deployment (Terraform secrets, ECS task def, etc.)

## Rules

### Log Levels
- **fatal**: Data integrity risk, manual intervention needed
- **error**: Operation failed, needs investigation
- **warn**: Expected failure or suspicious activity (auth failures, scanner probes, validation rejections)
- **info**: Successful operations worth tracking (signups, purchases, payments, deletions)
- **debug**: Routine lookups, cache hits, internal state (suppressed in prod unless LOG_LEVEL=debug)

### What to Include in Structured Data
- User IDs, wallet IDs, session IDs — for correlation
- Amounts, balances, counts — for financial audit trail
- External service IDs (Stripe session, Twilio SID) — for cross-system debugging
- Boolean flags (found, enabled, hasCredential) — for quick filtering

### What NOT to Log
- Secrets, tokens, passwords, API keys
- Full request/response bodies (use specific fields instead)
- Duplicate data already logged by another layer (don't log in both lib and route for the same operation)
- Request entry/exit — the request logger plugin handles this globally

### Avoid Duplicate Logging
Pick ONE layer to own each log:
- **Lib functions** own: operation start, external API calls, DB mutations, result context
- **Route handlers** own: business decisions (rejections, validation, user-facing errors)
- **Plugins** own: cross-cutting concerns (auth, rate limiting, request lifecycle)

If a lib function logs the operation details, the route should NOT re-log the same data.

### Request Context
Every log line should automatically include:
- `requestId` — UUID generated per request
- `clientIp` — from X-Forwarded-For
- `userId` / `userEmail` — enriched after auth

Use `AsyncLocalStorage` with a pino mixin to inject these automatically.

## Phase 3: Check Live Logs

After changes, verify logs are actually flowing:

```bash
# CloudWatch (AWS)
aws logs tail /ecs/<service-name> --follow --region <region>

# Docker
docker logs -f <container>

# Local dev
# Just run the server and hit endpoints
```

## Output Format

```
[STATUS] file:line — Description
  Current: <what exists>
  Fix: <what to change>
  Why: <brief reason>
```

Status levels:
- **MISSING** — No logging exists where it should
- **BROKEN** — Logging exists but doesn't work (wrong transport, missing config)
- **NOISY** — Over-logging that dilutes signal (duplicates, info-level for routine ops)
- **LEAK** — Sensitive data in logs
- **OK** — Adequate coverage

### Summary

"Audited N files. Found: X missing, Y broken, Z noisy, W leaks."
