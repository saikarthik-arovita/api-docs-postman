# Why Splitting Auth into 3 Lambdas Won't Fix Cold Starts

## What Was Proposed
Split the current `auth-service` Lambda (which handles `/auth/*`, `/hrms/*`, `/organization/*`) 
into three separate Lambda functions to reduce cold start time.

---

## Why It's Not the Right Fix

### 1. The Heavy Cost Doesn't Go Away

Cold start time in Python Lambda is dominated by **runtime and dependency initialization**, 
not by your application code size:

```
Python runtime startup         ~300–400ms  → same in all 3 Lambdas
pydantic import                ~100–150ms  → same in all 3 Lambdas
psycopg2 (database driver)     ~80–100ms   → same in all 3 Lambdas
boto3 / botocore               ~150–200ms  → same in all 3 Lambdas
─────────────────────────────────────────────────────────────────
Total cold start               ~700–900ms
```

All three split services import the same `shared/` codebase, the same database driver, 
and the same AWS SDK. Splitting doesn't remove any of that. You save at best 50–100ms 
from a smaller zip — but you still pay the full Python startup cost three times over.

**We already address the app code portion** using a lazy-loading router pattern — service 
imports are deferred until the first request. That's the one part splitting would have helped 
with, and it's already done.

---

### 2. You End Up With More Cold Starts, Not Fewer

This is the critical counter-argument.

`/hrms/*` routes are used for staff onboarding — not on every request, not every minute.  
`/organization/*` routes are used only by SYSTEM_ADMINISTRATOR for branch switching — rare.

If these become separate Lambdas, they go cold between uses and cold-start **every time 
they're invoked after idle**. Currently, a warm auth Lambda also handles these paths instantly.

```
Current:  1 Lambda can go cold
Proposed: 3 Lambdas can each go cold independently
```

Splitting trades one warm surface for three cold surfaces. That's the wrong direction.

---

### 3. Significant Operational Overhead for Minimal Gain

Each new Lambda means:
- Separate CloudWatch log group to monitor
- Separate deployment pipeline and versioning
- Separate alarm/alert configuration
- Separate API Gateway integration
- More IAM roles and policies to manage
- Coordinated deployments when `shared/` code changes

For a cold start saving of 50–100ms (best case), this overhead is not justified.

---

### 4. The Actual Fix Is One Config Line

**Provisioned Concurrency** keeps Lambda instances pre-warmed — zero cold start, 
no code changes, no architectural risk.

```yaml
ProvisionedConcurrencyConfig:
  ProvisionedConcurrentExecutions: 1
```

Cost: ~$15–20/month per provisioned instance.  
Result: 0ms cold start on the auth Lambda — the most latency-critical path.

Additional quick wins (all config changes, no code changes):

| Change | Effort | Cold Start Impact |
|--------|--------|------------------|
| Provisioned Concurrency: 1 | 1 line | Eliminates cold start |
| Switch to ARM64 (Graviton2) | 1 line | ~15% faster + 20% cheaper |
| Increase memory to 512MB | 1 line | ~40% faster init (more CPU) |
| Lambda Layer for heavy deps | ~1 day | ~150ms improvement |

---

## Summary

Splitting into 3 Lambdas:
- ❌ Does not remove the dominant cold start costs (Python runtime + shared imports)
- ❌ Increases total cold starts across the system (3 surfaces instead of 1)
- ❌ Adds operational complexity with 3 deployments, log groups, and integrations
- ❌ Duplicates shared middleware across services
- ✅ Saves at best 50–100ms — which is already addressed by our lazy-loading router

The correct fix is **Provisioned Concurrency on the auth Lambda**, which directly eliminates 
cold start with a single config change and no architectural disruption.
