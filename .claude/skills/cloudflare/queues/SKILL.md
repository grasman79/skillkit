---
name: cloudflare-queues
description: Cloudflare Queues - message queue for asynchronous processing on Workers. Covers producers (send, sendBatch), push consumers (queue handler, batching, ack/retry), pull consumers (HTTP API), dead letter queues, delays, retries with backoff, and integration with Agents SDK. Trigger words - cloudflare queues, message queue, queue consumer, queue producer, async processing, background jobs cloudflare, dead letter queue, sendBatch, queue handler, cloudflare messaging, worker queue
---

# Cloudflare Queues

Guaranteed-delivery message queue for asynchronous processing on Cloudflare Workers. Producers send messages, consumers process them in batches with automatic retries and dead letter queue support.

## Source of Truth

Official documentation: https://developers.cloudflare.com/queues/

Always check the official docs when the skill content conflicts or when you need the latest API details.

## When to Use This Skill

- Offloading time-consuming work from incoming HTTP requests
- Processing tasks asynchronously (email sending, image processing, data pipelines)
- Worker-to-Worker communication
- Buffering and batching data for bulk operations
- Building event-driven architectures
- Background job processing with guaranteed delivery

## When NOT to Use This Skill

- Real-time WebSocket communication (use `cloudflare/agents`)
- Simple cron triggers on a schedule (use Workers Cron Triggers)
- Stateful AI agent orchestration (use `cloudflare/agents` with scheduling)
- One-off HTTP requests between Workers (use Service Bindings)

## Prerequisites

Read [workers-core](../workers-core/SKILL.md) for wrangler CLI and deployment fundamentals.

## Setup

### Create a Queue

```bash
npx wrangler queues create my-queue
```

Queue naming rules:
- 1-63 characters, start and end with letter or number
- Only letters, numbers, and dashes (`-`)
- Names are permanent - cannot be renamed after creation

### wrangler.jsonc Configuration

```jsonc
{
  "name": "my-worker",
  "main": "src/index.ts",
  "compatibility_flags": ["nodejs_compat"],
  "queues": {
    "producers": [
      {
        "queue": "my-queue",
        "binding": "MY_QUEUE"
      }
    ],
    "consumers": [
      {
        "queue": "my-queue",
        "max_batch_size": 10,
        "max_batch_timeout": 5,
        "max_retries": 3,
        "dead_letter_queue": "my-queue-dlq"
      }
    ]
  }
}
```

A single Worker can be both producer and consumer for the same queue.

### TypeScript Types

```typescript
export interface Env {
  MY_QUEUE: Queue<MyMessageType>;
}

type MyMessageType = {
  action: string;
  userId: string;
  data: Record<string, unknown>;
};
```

Run `npx wrangler types` to auto-generate bindings.

## Producer API

### send() - Single Message

```typescript
await env.MY_QUEUE.send({
  action: "process-image",
  userId: "user-123",
  data: { imageUrl: "https://example.com/photo.jpg" },
});
```

With options:

```typescript
await env.MY_QUEUE.send(messageBody, {
  contentType: "json",    // "json" (default) | "text" | "bytes" | "v8"
  delaySeconds: 600,      // Delay delivery up to 24 hours (0-86400)
});
```

### sendBatch() - Multiple Messages

Up to 100 messages per batch, 256 KB total:

```typescript
const messages = items.map((item) => ({
  body: { action: "process", itemId: item.id },
  delaySeconds: 0,
}));

await env.MY_QUEUE.sendBatch(messages);
```

Batch-level delay (applies to all messages):

```typescript
await env.MY_QUEUE.sendBatch(messages, { delaySeconds: 300 });
```

### metrics() - Queue Stats

```typescript
const stats = await env.MY_QUEUE.metrics();
// stats.backlogCount - messages waiting
// stats.backlogBytes - total size in bytes
// stats.oldestMessageTimestamp - epoch ms of oldest message
```

### Content Types

| Type | Use for | Dashboard preview |
|------|---------|-------------------|
| `json` (default) | Objects, arrays | Yes |
| `text` | Plain strings | Yes |
| `bytes` | ArrayBuffer, binary | Base64 |
| `v8` | Date, Map, Set, RegExp | Base64 |

`v8` uses the structured clone algorithm for types that JSON cannot serialize. Only works with push-based (Worker) consumers, not pull consumers.

## Push Consumer (Worker)

The `queue()` handler receives batches of messages. Each queue can have only one push consumer.

### Basic Consumer

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    await env.MY_QUEUE.send({ url: request.url });
    return new Response("Queued");
  },

  async queue(batch: MessageBatch<MyMessageType>, env: Env, ctx: ExecutionContext): Promise<void> {
    for (const message of batch.messages) {
      console.log(`Processing: ${message.body.action} (attempt ${message.attempts})`);
      await processMessage(message.body);
    }
  },
} satisfies ExportedHandler<Env>;
```

### Message Object

| Property | Type | Description |
|----------|------|-------------|
| `message.id` | `string` | Unique message ID |
| `message.body` | `Body` | Message content |
| `message.timestamp` | `Date` | When the message was sent |
| `message.attempts` | `number` | Processing attempt count (starts at 1) |

### Acknowledgment

By default, all messages in a batch are acknowledged when the handler returns successfully. Throwing an error retries the entire batch.

For fine-grained control, use explicit ack/retry:

```typescript
async queue(batch: MessageBatch<MyMessageType>, env: Env, ctx: ExecutionContext): Promise<void> {
  for (const message of batch.messages) {
    try {
      await processMessage(message.body);
      message.ack();
    } catch (error) {
      if (message.attempts >= 3) {
        // Give up - will go to DLQ if configured
        message.ack();
        console.error(`Permanently failed: ${message.id}`);
      } else {
        message.retry({ delaySeconds: 60 });
      }
    }
  }
}
```

Rules:
- Once `ack()` or `retry()` is called on a message, subsequent calls are ignored
- Per-message calls take precedence over `batch.ackAll()` / `batch.retryAll()`
- Use `batch.ackAll()` to acknowledge everything at once
- Use `batch.retryAll()` to retry everything at once

### Exponential Backoff

```typescript
async queue(batch: MessageBatch<MyMessageType>, env: Env): Promise<void> {
  for (const message of batch.messages) {
    try {
      await processMessage(message.body);
      message.ack();
    } catch (error) {
      const delay = Math.min(30 ** message.attempts, 86400);
      message.retry({ delaySeconds: delay });
    }
  }
}
```

### Async Operations

Use `waitUntil()` for fire-and-forget work after processing:

```typescript
async queue(batch: MessageBatch, env: Env, ctx: ExecutionContext): Promise<void> {
  for (const message of batch.messages) {
    await processMessage(message.body);
  }
  // Fire-and-forget logging
  ctx.waitUntil(logBatchMetrics(batch));
}
```

Do not use `forEach` with async operations - use `for...of` or `Promise.all(batch.messages.map(...))`.

## Consumer Settings

| Setting | Default | Range | Description |
|---------|---------|-------|-------------|
| `max_batch_size` | 10 | 1-100 | Messages per batch |
| `max_batch_timeout` | 5 | 0-60 seconds | Wait time before delivering partial batch |
| `max_retries` | 3 | 0-100 | Retry attempts before DLQ or deletion |
| `max_concurrency` | auto | 1-250 | Concurrent consumer invocations |
| `dead_letter_queue` | none | - | Queue name for permanently failed messages |
| `retry_delay` | 0 | 0-86400 seconds | Default delay between retries |

Batches are delivered when either `max_batch_size` or `max_batch_timeout` is reached - whichever comes first.

## Dead Letter Queue (DLQ)

Messages that fail after `max_retries` are routed to the DLQ. Without a DLQ, failed messages are permanently deleted.

### Setup

```jsonc
{
  "queues": {
    "consumers": [
      {
        "queue": "my-queue",
        "max_retries": 3,
        "dead_letter_queue": "my-queue-dlq"
      }
    ]
  }
}
```

Create the DLQ:

```bash
npx wrangler queues create my-queue-dlq
```

The DLQ is a regular queue - you can add a consumer to process failed messages, inspect them, or forward them. Without a consumer, DLQ messages are deleted after 4 days (or 24 hours on free plans).

### DLQ Consumer

```typescript
async queue(batch: MessageBatch, env: Env): Promise<void> {
  for (const message of batch.messages) {
    console.error(`DLQ message from ${batch.queue}:`, message.body);
    // Log to external system, alert, or store for manual review
    await env.FAILED_MESSAGES_DB.put(message.id, JSON.stringify({
      body: message.body,
      attempts: message.attempts,
      timestamp: message.timestamp.toISOString(),
    }));
  }
}
```

## Message Delays

### On Send

```typescript
// Delay individual message
await env.MY_QUEUE.send(body, { delaySeconds: 600 });

// Delay entire batch
await env.MY_QUEUE.sendBatch(messages, { delaySeconds: 300 });

// Per-message delay in batch
await env.MY_QUEUE.sendBatch([
  { body: urgent, delaySeconds: 0 },
  { body: canWait, delaySeconds: 3600 },
]);
```

### Queue-Level Default Delay

```bash
npx wrangler queues create my-queue --delivery-delay-secs=60
npx wrangler queues update my-queue --delivery-delay-secs=120
```

Per-message `delaySeconds` overrides the queue-level default. Setting `delaySeconds: 0` bypasses the default.

### On Retry

```typescript
message.retry({ delaySeconds: 300 });
batch.retryAll({ delaySeconds: 60 });
```

## Pull Consumer (HTTP API)

For consuming messages from outside Cloudflare Workers via HTTP.

### Enable Pull Consumer

```bash
npx wrangler queues consumer http add my-queue
```

Each queue can have either a push consumer OR a pull consumer, not both.

### Authentication

Create an API token with `Account > Queues > Edit` permission. Use as Bearer token.

### Pull Messages

```typescript
const response = await fetch(
  `https://api.cloudflare.com/client/v4/accounts/${accountId}/queues/${queueId}/messages/pull`,
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${apiToken}`,
    },
    body: JSON.stringify({
      batch_size: 50,
      visibility_timeout_ms: 30000,
    }),
  }
);

const { result } = await response.json();
// result.messages - array of messages
// result.message_backlog_count - remaining messages
```

### Acknowledge Messages

```typescript
await fetch(
  `https://api.cloudflare.com/client/v4/accounts/${accountId}/queues/${queueId}/messages/ack`,
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${apiToken}`,
    },
    body: JSON.stringify({
      acks: [{ lease_id: "lease_id_1" }, { lease_id: "lease_id_2" }],
      retries: [{ lease_id: "lease_id_3", delay_seconds: 600 }],
    }),
  }
);
```

### Pull Consumer Notes

- Uses short polling (returns immediately, empty if no messages)
- Multiple HTTP clients can pull concurrently from the same queue
- Cannot consume `v8` content type (Workers-specific)
- `visibility_timeout_ms`: how long you have to ack before the message is requeued (default 30s, max 12 hours)

## Integration with Agents SDK

Agents can produce to queues from within agent methods, and queue consumers can interact with agents:

### Agent as Producer

```typescript
import { Agent, callable } from "agents";

export class TaskAgent extends Agent<Env, TaskState> {
  @callable()
  async submitTask(task: TaskData) {
    await this.env.TASK_QUEUE.send({
      agentId: this.name,
      task,
    });
    this.setState({ ...this.state, pendingTasks: this.state.pendingTasks + 1 });
  }
}
```

### Queue Consumer Updating Agent State

```typescript
import { getAgentByName } from "agents";

export default {
  async queue(batch: MessageBatch<TaskMessage>, env: Env): Promise<void> {
    for (const message of batch.messages) {
      const result = await processTask(message.body.task);

      // Notify the agent that submitted the task
      const agent = getAgentByName(env.TaskAgent, message.body.agentId);
      await agent.taskCompleted(result);

      message.ack();
    }
  },
} satisfies ExportedHandler<Env>;
```

## CLI Commands

```bash
# Create
npx wrangler queues create my-queue
npx wrangler queues create my-queue --delivery-delay-secs=60

# List
npx wrangler queues list

# Update
npx wrangler queues update my-queue --delivery-delay-secs=120 --message-retention-period-secs=3000

# Delete
npx wrangler queues delete my-queue

# Add push consumer
npx wrangler queues consumer worker add my-queue my-worker-script

# Add pull consumer
npx wrangler queues consumer http add my-queue

# Remove consumer
npx wrangler queues consumer worker remove my-queue my-worker-script

# Tail (debug)
npx wrangler tail
```

## Limits

| Resource | Limit |
|----------|-------|
| Queues per account | 10,000 |
| Message size | 128 KB |
| Messages per sendBatch | 100 (or 256 KB total) |
| Per-queue throughput | 5,000 messages/second |
| Per-queue backlog | 25 GB |
| Max retries | 100 |
| Max batch size | 100 messages |
| Max batch timeout | 60 seconds |
| Max delivery delay | 24 hours (86,400 seconds) |
| Concurrent consumer invocations | 250 (push-based) |
| Consumer wall-clock time | 15 minutes |
| Consumer CPU time | Up to 5 minutes |
| Visibility timeout (pull) | 12 hours |
| Message retention (paid) | Up to 14 days (configurable) |
| Message retention (free) | 24 hours (not configurable) |

Messages include ~100 bytes of internal metadata that counts toward limits.

## Common Gotchas

1. **One push consumer per queue** - you cannot connect multiple Worker consumers to the same queue. Use separate queues or a single consumer that routes internally.

2. **Queue names are permanent** - you cannot rename a queue after creation. Choose descriptive names like `user-events-prod` or `image-processing`.

3. **Don't use `forEach` with async** - use `for...of` or `Promise.all(batch.messages.map(...))` for async message processing.

4. **Batch ack is implicit** - if your handler returns without error and you didn't call `ack()` or `retry()` on individual messages, all messages are acknowledged. An uncaught error retries the entire batch.

5. **Per-message calls override batch calls** - once you call `message.ack()` or `message.retry()`, `batch.ackAll()` / `batch.retryAll()` won't affect that message.

6. **Retries count as reads** - each retry is a separate billable read operation.

7. **No DLQ = deleted** - without a dead letter queue configured, messages that exhaust retries are permanently lost.

8. **`v8` content type is Worker-only** - pull consumers cannot handle `v8` encoded messages. Use `json` for cross-platform compatibility.

9. **Empty queues don't trigger consumers** - push consumers are only invoked when messages are available, so no unnecessary compute charges.

10. **Free plan retention is 24 hours** - messages on the free plan expire after 24 hours with no way to extend. Paid plans support up to 14 days.
