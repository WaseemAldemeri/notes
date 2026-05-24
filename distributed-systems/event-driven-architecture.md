# Event-Driven Architecture — Patterns & Terminology

Synthesis note (not from a single book). Cross-references DDIA Ch 6 (Replication) and Ch 11 (Stream Processing).

#### Table of Contents

<!-- vim-markdown-toc GFM -->

* [The Core Problem](#the-core-problem)
* [Vocabulary](#vocabulary)
* [Delivery Patterns](#delivery-patterns)
  * [1. In-process fan-out](#1-in-process-fan-out)
  * [2. Transactional outbox + queue](#2-transactional-outbox--queue)
  * [3. DB trigger → webhook (Supabase-style)](#3-db-trigger--webhook-supabase-style)
  * [4. CDC (Change Data Capture)](#4-cdc-change-data-capture)
  * [5. WebSocket push](#5-websocket-push)
* [Choosing a Queue Backend](#choosing-a-queue-backend)
* [Workflow Engines (Durable Execution)](#workflow-engines-durable-execution)
* [Problems With This Exact Shape](#problems-with-this-exact-shape)
* [Reading List](#reading-list)

<!-- vim-markdown-toc -->

## The Core Problem

> Something happened in service A, and other code needs to react to it — durably, with at-least-once delivery, possibly to multiple subscribers.

Once you see this shape, it's everywhere: replication, webhooks, search index sync, cache invalidation, materialized view maintenance, audit logging, push notifications, analytics pipelines, workflow triggers. All the same problem at different layers of the stack, with different durability / ordering / fan-out tradeoffs.

## Vocabulary

| Term | Meaning |
|---|---|
| **Event-Driven Architecture (EDA)** | Umbrella term for systems where state changes emit events that other components react to. |
| **Pub/Sub** | Producers emit events without knowing subscribers; a broker fans out. The generic name for the dispatch shape. |
| **Event bus / message bus** | The dispatch component itself. |
| **Domain events** | DDD term — business-meaningful facts (`TicketCreated`, `RefundIssued`). Signals "part of the domain model, not just a notification." |
| **Transactional outbox** | Write the event row in the *same DB transaction* as the state change, then a separate process drains it to the queue. The standard fix for atomic publish. Named by Chris Richardson. |
| **Inbox pattern** | Consumer-side mirror of outbox: dedup table keyed by event ID so retries are idempotent. |
| **Saga** | Long-running multi-step business process with compensations. Two flavors: **orchestration** (central coordinator drives steps) and **choreography** (services react to each other's events with no central brain). Named by Garcia-Molina, 1987. |
| **Durable execution** | Code-as-workflow with replay-based recovery. Coined by Temporal; used by Cloudflare Workflows, Inngest, Restate, Hatchet, Resonate. |
| **CDC (Change Data Capture)** | Watching the DB's replication log to emit events on row changes — no app-level publish needed. Postgres logical replication, Debezium. |
| **At-least-once / at-most-once / exactly-once** | Delivery guarantees. "Exactly-once" is mostly a marketing claim; the real production posture is at-least-once + idempotent consumers. |

## Delivery Patterns

Ordered from naive to production-grade, with the failure mode each one fixes.

### 1. In-process fan-out

```ts
await db.insert('tickets', ticket);
await Promise.all(subscribers.map(s => dispatch(s, event)));
```

- **Fails when:** host crashes between insert and dispatch → event lost. Slow subscriber blocks the request. No retries.
- **Use when:** prototype, internal tool, lost trigger = mild annoyance.

### 2. Transactional outbox + queue

Write the event row in the same DB transaction as the state change. A consumer drains the queue and dispatches.

```ts
await db.transaction(async (tx) => {
  const ticket = await tx.insert('tickets', { ... });
  await boss.send('ticket.created', payload, { tx }); // pgboss: same txn
});
```

- **Fixes:** atomic "ticket exists ↔ event will be delivered." Survives consumer crashes (queue persists). Retries until consumer succeeds.
- **Trade:** "exactly-once" is impossible — consumer must be idempotent (use deterministic instance IDs, e.g. `${eventId}:${workflowId}`).
- **This is the default for anything you can't afford to lose.**

### 3. DB trigger → webhook (Supabase-style)

`AFTER INSERT/UPDATE` trigger on a table fires an HTTP call (via `pg_net` or similar) to an edge function, which fans out.

- **Pros:** zero polling, near-instant, no consumer process to operate.
- **Cons:** trigger fires *after commit*, so if the HTTP call fails the row is committed but the event is lost. No durable queue, no retry semantics (or weak ones). Edge function timeouts cap fan-out width. Surge of inserts → surge of concurrent invocations → backpressure problems.
- **Closer to:** lightweight CDC, but without durability.
- **Use when:** loss is acceptable (UI notifications, analytics, "nice to have" side effects). **Avoid when:** money / regulatory / can't-drop-this.

### 4. CDC (Change Data Capture)

A consumer reads the DB's replication log (Postgres logical decoding, MySQL binlog) and emits events for every row change. Application code knows nothing.

- **Pros:** maximally decoupled — the DB itself is the event source. Atomic by construction (the log *is* the commit).
- **Cons:** infra-heavy (Debezium + Kafka usually), tight coupling to schema (renames break consumers).
- **Use when:** multiple downstream systems each need to see every change (search index, warehouse, cache, notifications).

### 5. WebSocket push

Persistent connection per subscriber, push events on commit.

- **Real pattern for:** server → browser (live UI updates, chat, notifications).
- **Wrong tool for:** server → server event bus. No durability (in-memory), reinvents reconnect / heartbeat / backpressure / acks badly. Workflows aren't long-lived listeners anyway — they're definitions instantiated per event.
- **Hybrid is the right architecture:** queue between servers (durable), WebSocket as the last hop to a human's screen.

## Choosing a Queue Backend

Stay on Postgres until you hit a *specific* problem you can name.

| Need | Reach for |
|---|---|
| Modest scale, job queue semantics | **pg-boss / BullMQ** |
| Transactional enqueue in same DB | **pg-boss** (the killer feature — no outbox table needed) |
| Push semantics, complex routing | RabbitMQ |
| Event log, replay, multiple independent consumer groups | Kafka / Redpanda / NATS JetStream |
| Already on AWS | SQS (+ SNS for fan-out) |
| Already on Cloudflare | Cloudflare Queues |

**Signals you've outgrown Postgres-as-queue** (roughly in order):
1. Vacuum / table bloat from high enqueue+delete rate. Fix: move pgboss to its own Postgres DB before changing tech.
2. Queue traffic degrades OLTP queries. Same fix.
3. Need real pub/sub fan-out (N independent consumers each see every event), not queue semantics.
4. Need replay ("re-run all events from last Tuesday"). pgboss deletes completed jobs; Kafka keeps the log.
5. Cross-service coupling (service B reaching into A's DB feels wrong).
6. Throughput ceiling (low thousands of jobs/sec). Most apps never hit this.

**The trap:** picking Kafka on day one because it "scales." Three months on ZooKeeper/KRaft instead of building features.

## Workflow Engines (Durable Execution)

What you build when "event → multi-step business process" becomes a recurring shape.

- **Temporal, Cloudflare Workflows, Inngest, Restate, Hatchet, Resonate** — code-as-workflow, replay-based recovery. The defining trait: the handler runs, hits an `await step.do(...)`, the result is persisted, and on any future replay the same step returns the cached value instead of re-executing.
- **Airflow, Prefect, Dagster, Argo Workflows** — older, data-pipeline lineage. Same shape, more YAML/DAG-flavored.
- **Sagas** — these *are* sagas in microservices terminology. Orchestration style (central engine drives) vs choreography (services react to each other).

Implementation note (from building [[open-workflows]]): the workflow input is durable on the engine side, but to make your *own* telemetry log dedupe across replays, wrap the trigger payload in a `step.do('__trigger__', ...)` so it's memoized exactly once.

## Problems With This Exact Shape

Once the pattern clicks, you start seeing it everywhere:

| Problem | What it is |
|---|---|
| **Replication** | Leader's WAL is an event stream; followers are subscribers. (DDIA Ch 6.) |
| **Webhooks** | Same as workflow triggers, subscriber is an external HTTP endpoint. |
| **Search index sync** | Row changed → update Elasticsearch. Usually CDC + consumer. |
| **Cache invalidation** | "X changed → invalidate caches keyed on X." Pub/sub for cache busts. |
| **Materialized view maintenance** | Write to base table → update derived table. |
| **Audit logging** | "Thing happened → durably record it elsewhere." Outbox is the standard fix. |
| **Push notifications / email / SMS** | Queue + delivery worker. |
| **Analytics events** | "User clicked → durable pipeline to warehouse." Kafka-flavored. |

The unifying insight: **replication, CDC, pub/sub, webhooks, workflow triggers, and cache invalidation are the same problem at different layers of the stack.** Different durability / ordering / delivery tradeoffs, same shape.

## Reading List

1. **DDIA Chapter 11, "Stream Processing"** — literally this material; generalizes Ch 6.
2. **Chris Richardson, *Microservices Patterns*** — outbox, saga, inbox, eventual consistency.
3. **Martin Fowler, "What do you mean by 'Event-Driven'?"** — disambiguates the four meanings of EDA (event notification, event-carried state transfer, event sourcing, CQRS). Short, essential.
4. **Greg Young on event sourcing** — events as the source of truth, current state as a derived view.
5. **Temporal docs on durable execution** — clearest writing on the workflow-as-code model.
