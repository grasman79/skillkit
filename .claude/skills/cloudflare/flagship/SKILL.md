---
name: flagship
description: Cloudflare Flagship feature flag service for controlling feature visibility without redeploying code. Covers dashboard setup, wrangler binding configuration, binding API (getBooleanValue/getStringValue/getNumberValue/getObjectValue/getDetails), OpenFeature SDK (server and client providers), targeting rules, percentage rollouts, evaluation context, evaluation reasons, and TypeScript types. Trigger words - flagship, feature flags, feature flag, feature toggle, feature toggles, flag evaluation, percentage rollout, gradual release, targeting rules.
---

# Cloudflare Flagship

Feature flag service for controlling feature visibility without redeploying code. Evaluate flags locally at the edge with targeting rules, percentage rollouts, and multi-type variations.

## Source of Truth

Official Cloudflare developer documentation is the primary source of truth:

- https://developers.cloudflare.com/flagship/

## When to Use This Skill

- Adding feature flags to a Cloudflare Worker
- Implementing gradual rollouts to a percentage of users
- Targeting specific user segments with different feature variations
- Using the OpenFeature SDK for vendor-neutral flag evaluation
- Setting up feature flag bindings in wrangler.jsonc

## Concepts

### Core Model

**Apps** group related flags and map to a project or product surface. One Cloudflare account can have multiple apps.

**Flags** are named toggles with a unique key (letters, numbers, hyphens, underscores). Disabled flags always return their default variation regardless of rules.

**Variations** are the possible values a flag can return. Types: `boolean`, `string`, `number`, `JSON object`. Each flag needs at least one variation designated as the default.

**Targeting rules** evaluate sequentially - the first matching rule wins. If no rule matches, the default variation is returned.

**Evaluation context** is a set of key-value attributes describing the current user or request (`userId`, `country`, `plan`, etc.). Consistent context produces consistent rollout results.

**Flag propagation** is automatic and global within seconds after a dashboard change.

### Flag Lifecycle

1. **Configure** - create flags in the dashboard (Compute > Flagship)
2. **Propagate** - automatic global distribution
3. **Evaluate** - local computation inside your Worker, no server round-trips

## Dashboard Setup

1. Go to Cloudflare dashboard > Compute > Flagship
2. Create an **App** (e.g., `checkout-service`) - this gives you an `APP_ID`
3. Add a **Flag** with a key (e.g., `new-checkout`) and choose its type (boolean, string, number, JSON)
4. Optionally configure **Targeting Rules** before saving

## Wrangler Configuration

Add a `flagship` binding to `wrangler.jsonc`:

```jsonc
{
  "flagship": [
    {
      "binding": "FLAGS",
      "app_id": "<APP_ID>"
    }
  ]
}
```

For multiple apps in one Worker:

```jsonc
{
  "flagship": [
    {
      "binding": "FLAGS",
      "app_id": "<APP_ID_1>"
    },
    {
      "binding": "EXPERIMENT_FLAGS",
      "app_id": "<APP_ID_2>"
    }
  ]
}
```

After adding the binding, regenerate TypeScript types:

```bash
wrangler types
```

This types each binding as `Flagship` in the generated `Env` interface.

## Binding API

All methods are async, never throw, and return the default value on any error (flag not found, type mismatch, network failure).

### Value Methods

```typescript
// Boolean
const enabled = await env.FLAGS.getBooleanValue("dark-mode", false, {
  userId: "user-42",
});

// String
const variant = await env.FLAGS.getStringValue("checkout-flow", "v1", {
  userId: "user-42",
  country: "US",
});

// Number
const maxRetries = await env.FLAGS.getNumberValue("max-retries", 3, {
  plan: "enterprise",
});

// Typed object
interface ThemeConfig {
  primaryColor: string;
  fontSize: number;
}
const theme = await env.FLAGS.getObjectValue<ThemeConfig>(
  "theme-config",
  { primaryColor: "#000", fontSize: 14 },
  { userId: "user-42" },
);

// Raw value (no type checking)
const value = await env.FLAGS.get("checkout-flow", "v1", {
  userId: "user-42",
});
```

### Details Methods (with evaluation metadata)

Use `*Details` methods when you need to know why a value was returned:

```typescript
const details = await env.FLAGS.getBooleanDetails("dark-mode", false, {
  userId: "user-42",
});
// details.value    → true
// details.reason   → "TARGETING_MATCH"
// details.variant  → "enabled"

const details = await env.FLAGS.getStringDetails("checkout-flow", "v1", {
  userId: "user-42",
});
// details.value    → "v2"
// details.variant  → "new"
// details.reason   → "TARGETING_MATCH"
```

Typed variants: `getBooleanDetails`, `getStringDetails`, `getNumberDetails`, `getObjectDetails`.

### TypeScript Types

```typescript
interface FlagshipEvaluationContext {
  [key: string]: string | number | boolean;
}

interface FlagshipEvaluationDetails<T> {
  value: T;
  variant?: string;
  reason: "TARGETING_MATCH" | "SPLIT" | "DEFAULT" | "DISABLED" | "ERROR";
  errorCode?: "TYPE_MISMATCH" | "GENERAL" | "FLAG_NOT_FOUND";
  errorMessage?: string;
}
```

## Complete Worker Example

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    const userId = url.searchParams.get("userId") ?? "anonymous";

    const showNewCheckout = await env.FLAGS.getBooleanValue(
      "new-checkout",
      false,
      { userId }
    );

    return new Response(
      showNewCheckout
        ? "Welcome to the new checkout experience!"
        : "Standard checkout."
    );
  },
};
```

## OpenFeature SDK

Use `@cloudflare/flagship` with the OpenFeature SDK for a vendor-neutral API that works across runtimes.

### Installation

```bash
# Server-side (Workers, Node.js)
bun add @cloudflare/flagship @openfeature/server-sdk

# Browser
bun add @cloudflare/flagship @openfeature/web-sdk
```

### Server Provider (Workers - recommended)

Pass the binding directly to avoid HTTP overhead:

```typescript
import { OpenFeature } from "@openfeature/server-sdk";
import { FlagshipServerProvider } from "@cloudflare/flagship";

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    await OpenFeature.setProviderAndWait(
      new FlagshipServerProvider({ binding: env.FLAGS })
    );
    const client = OpenFeature.getClient();

    const showNewCheckout = await client.getBooleanValue(
      "new-checkout",
      false,
      { targetingKey: "user-42", plan: "enterprise" }
    );

    return showNewCheckout
      ? new Response("New checkout enabled!")
      : new Response("Standard checkout.");
  },
};
```

### Server Provider (Node.js / External)

```typescript
import { OpenFeature } from "@openfeature/server-sdk";
import { FlagshipServerProvider } from "@cloudflare/flagship";

await OpenFeature.setProviderAndWait(
  new FlagshipServerProvider({
    appId: "<APP_ID>",
    accountId: "<ACCOUNT_ID>",
    authToken: "<API_TOKEN>",
  })
);

const client = OpenFeature.getClient();
const enabled = await client.getBooleanValue("new-checkout", false, {
  targetingKey: "user-42",
  plan: "enterprise",
  country: "US",
});
```

Provider config: provide either `binding` (Workers) OR the triple `appId` + `accountId` + `authToken` (external).

### Client Provider (Browser)

Pre-fetches all flag values on init, evaluates synchronously from in-memory cache:

```typescript
import { OpenFeature } from "@openfeature/web-sdk";
import { FlagshipClientProvider } from "@cloudflare/flagship";

await OpenFeature.setProviderAndWait(
  new FlagshipClientProvider({
    appId: "<APP_ID>",
    accountId: "<ACCOUNT_ID>",
    authToken: "<API_TOKEN>",
    context: { targetingKey: userId },
  })
);

const client = OpenFeature.getClient();
const enabled = client.getBooleanValue("dark-mode", false); // synchronous
```

### Hooks

```typescript
import { LoggingHook, TelemetryHook } from "@cloudflare/flagship";

OpenFeature.addHooks(new LoggingHook(), new TelemetryHook());
```

## Targeting Rules

Rules are evaluated top to bottom. The first matching rule wins. If nothing matches, the default variation is returned.

### Rule Structure

Each rule has:
- **Conditions** - attribute comparisons against the evaluation context
- **Variation** - what value to return when the rule matches
- **Rollout** (optional) - percentage of matching users to apply this rule to

### Supported Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `equals` | Exact match | `country equals "US"` |
| `not_equals` | Does not match | `plan not_equals "free"` |
| `greater_than` | Numeric greater than | `age greater_than 18` |
| `less_than` | Numeric less than | `loginCount less_than 5` |
| `greater_than_or_equals` | Numeric >= | `score greater_than_or_equals 90` |
| `less_than_or_equals` | Numeric <= | `createdAt less_than_or_equals "2025-01-01T00:00:00Z"` |
| `contains` | Substring match | `email contains "@cloudflare.com"` |
| `starts_with` | Prefix match | `path starts_with "/api/v2"` |
| `ends_with` | Suffix match | `domain ends_with ".dev"` |
| `in` | Value in array | `country in ["US", "CA", "UK"]` |
| `not_in` | Value not in array | `userId not_in ["blocked-1", "blocked-2"]` |

### Logical Grouping

Conditions can be grouped with `AND`/`OR` and nested up to six levels deep.

Example - enterprise users in North America:
```
plan equals "enterprise"
AND (country equals "US" OR country equals "CA")
```

### Evaluation Context in Code

Pass any attributes your targeting rules reference:

```typescript
const value = await env.FLAGS.getBooleanValue("new-dashboard", false, {
  userId: "user-42",      // used for percentage rollout bucketing
  plan: "enterprise",     // used by plan-based rules
  country: "US",          // used by geo rules
  email: "user@company.com",
});
```

## Percentage Rollouts

Any targeting rule can include a rollout percentage (0-100) to enable gradual releases.

### How It Works

1. Rule conditions are checked first
2. If conditions match, Flagship hashes a configurable attribute (default: `targetingKey`) to assign the user to a bucket
3. Users within the rollout percentage get the rule's variation; others fall through to subsequent rules

**Consistent bucketing:** the same user always receives the same result for a given rollout configuration - provided you pass a stable `targetingKey` (or configured bucketing attribute). Without a stable key, the bucket is random per evaluation.

### Example - Staged Rollout

`new-checkout` flag targeting rules (evaluated in order):

1. `plan equals "enterprise"` → always return `true` (100% rollout)
2. All other users → return `true` at 25% rollout (bucketed by `userId`)
3. Default variation → `false`

```typescript
const enabled = await env.FLAGS.getBooleanValue("new-checkout", false, {
  userId: "user-42",   // stable bucketing key
  plan: "standard",
});
// 25% of standard users get true, 75% get false
```

Increase the percentage over time as confidence builds, up to 100%.

## Evaluation Reasons

| Reason | When |
|--------|------|
| `TARGETING_MATCH` | A targeting rule's conditions matched |
| `SPLIT` | A percentage rollout rule matched and user fell within the rollout |
| `DEFAULT` | No rule matched - default variation returned |
| `DISABLED` | Flag is disabled - default variation returned |
| `ERROR` | Evaluation failed (check `errorCode`) |

| Error Code | Cause |
|------------|-------|
| `TYPE_MISMATCH` | Flag variation type doesn't match the method called (e.g., `getBooleanValue` on a string flag) |
| `GENERAL` | Unexpected error (e.g., network failure) |
| `FLAG_NOT_FOUND` | Flag key doesn't exist in the app |

In all error cases the default value is returned. Use `*Details` methods to inspect the reason.

## Common Gotchas

1. **Always pass a stable `targetingKey`** for consistent percentage rollouts - anonymous or random keys cause different results per request
2. **`*Value` methods never throw** - if evaluation fails silently, use `*Details` to see the reason and error code
3. **Type mismatches return the default** - calling `getBooleanValue` on a string flag returns the default boolean, not an error thrown
4. **Disabled flags ignore all rules** - they always return the default variation even if targeting rules would match
5. **Rule order matters** - rules are evaluated top to bottom; more specific rules should come before catch-all rules
6. **Run `wrangler types` after binding changes** - the generated `Flagship` type on `env.FLAGS` won't update automatically
7. **Multiple apps need multiple bindings** - each Flagship app requires its own `flagship` entry in `wrangler.jsonc`
8. **Brief propagation window** - immediately after a flag change, different regions may briefly serve different values

## How to Verify

- `wrangler dev` starts without binding errors
- `wrangler types` generates `FLAGS: Flagship` in the `Env` interface
- Querying with a matching evaluation context returns the expected variation
- Querying with a non-matching context returns the default variation
- `*Details` methods return the correct `reason` field
