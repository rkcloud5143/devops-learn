# AWS SQS (Simple Queue Service) — Deep Dive

---

## HOW SQS WORKS

```
Producer ──► [  Message Queue  ] ──► Consumer
             msg3  msg2  msg1        (polls for messages)

- Producer sends message to queue
- Message sits in queue until consumed
- Consumer polls (pulls) messages
- Consumer processes message, then deletes it
- If consumer fails, message becomes visible again (retry)
```

---

## QUEUE TYPES

```
┌─── Standard Queue ───────────────────────────────────────────┐
│  Throughput: nearly unlimited                                │
│  Ordering: best-effort (may be out of order)                 │
│  Delivery: at-least-once (may get duplicates)                │
│  Use for: most workloads, high throughput                    │
└──────────────────────────────────────────────────────────────┘

┌─── FIFO Queue ───────────────────────────────────────────────┐
│  Throughput: 300 msg/sec (3000 with batching)                │
│  Ordering: guaranteed (first in, first out)                  │
│  Delivery: exactly-once (no duplicates)                      │
│  Name must end with .fifo                                    │
│  Use for: financial transactions, order processing           │
└──────────────────────────────────────────────────────────────┘
```

---

## KEY CONCEPTS

```
┌─── Visibility Timeout ───────────────────────────────────────┐
│                                                               │
│  Consumer receives message → message becomes INVISIBLE       │
│  Consumer has X seconds to process and delete it             │
│  If not deleted in time → message becomes VISIBLE again      │
│                                                               │
│  Default: 30 seconds                                         │
│  Max: 12 hours                                               │
│                                                               │
│  Too short: message processed twice (duplicate)              │
│  Too long: slow retry if consumer crashes                    │
└──────────────────────────────────────────────────────────────┘

┌─── Dead Letter Queue (DLQ) ──────────────────────────────────┐
│                                                               │
│  Main Queue ──► Consumer fails 3 times ──► DLQ              │
│                                                               │
│  Messages that can't be processed go to DLQ                  │
│  You investigate DLQ messages manually or with alerts        │
│  MaxReceiveCount: how many retries before sending to DLQ     │
│                                                               │
│  ALWAYS set up a DLQ — prevents poison messages from         │
│  blocking your queue forever                                 │
└──────────────────────────────────────────────────────────────┘

┌─── Long Polling vs Short Polling ────────────────────────────┐
│                                                               │
│  Short Polling (default):                                    │
│  - Returns immediately (even if queue is empty)              │
│  - More API calls = more cost                                │
│                                                               │
│  Long Polling (recommended):                                 │
│  - Waits up to 20 seconds for messages to arrive             │
│  - Fewer API calls = less cost                               │
│  - Set WaitTimeSeconds: 1-20                                 │
└──────────────────────────────────────────────────────────────┘

┌─── Delay Queue ──────────────────────────────────────────────┐
│  Messages are invisible for X seconds after being sent       │
│  Default: 0 seconds (no delay)                               │
│  Max: 15 minutes                                             │
│  Use for: delayed processing, rate limiting                  │
└──────────────────────────────────────────────────────────────┘
```

---

## SQS + LAMBDA (Common Pattern)

```
┌──────────┐     ┌───────────┐     ┌──────────┐
│ Producer │────►│  SQS      │────►│  Lambda  │
│          │     │  Queue    │     │ (consumer)│
└──────────┘     └───────────┘     └──────────┘

- Lambda polls SQS automatically (event source mapping)
- Lambda scales up based on queue depth
- Failed messages → retry → DLQ
- No servers to manage
```

---

## SQS vs SNS vs EventBridge

```
┌──────────────────┬──────────────────┬──────────────────────┐
│ SQS              │ SNS              │ EventBridge          │
├──────────────────┼──────────────────┼──────────────────────┤
│ Queue (pull)     │ Topic (push)     │ Event bus (rules)    │
│ 1 consumer per   │ Many subscribers │ Many targets         │
│ message          │ per message      │ per rule             │
│ Decoupling       │ Fan-out          │ Event routing        │
│ Retry built-in   │ No retry         │ Retry + DLQ          │
│ Message persists │ Fire and forget  │ Archive + replay     │
│ up to 14 days    │                  │                      │
└──────────────────┴──────────────────┴──────────────────────┘
```
