---
title: "Architecture Traps – When AI Over‑Engineers"
datePublished: 2026-04-03T05:52:29.465Z
cuid: cmnihlgd2009i1qou7rd8cckq
slug: architecture-traps-when-ai-over-engineers
cover: https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/cb7f387b-7bf4-4220-ae56-c16cdad0848a.png

---

> AI models are trained on a wide range of architectures, from simple monoliths to massive distributed systems. When asked for design advice, they often default to complex, “enterprise‑grade” solutions that may be entirely wrong for your actual scale and team. This post highlights five architectural mistakes AI can lead you into and how to stay grounded.

* * *

![](https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/b9693b64-7676-423b-80eb-a054cfb2e9b4.png align="center")

* * *

### Mistake 1: Over‑Engineering with AI Suggestions

**Description:** AI suggests complex distributed solutions when simpler approaches would suffice.

**Realistic Scenario:** Team needs to store user preferences. AI suggests microservice, event sourcing, and Kafka.

❌ **Wrong Prompt:**

```TEXT
Design user preferences storage system
```

⚠️ **Why it is wrong:** AI may over-engineer without knowing scale (10K users, low write volume).

✅ **Better Prompt:**

```TEXT
Design user preferences storage for SaaS app with 10K users.

Constraints:

Reads: 10 req/min, Writes: 1 req/min

Simple JSON structure (notification settings, theme)

Existing PostgreSQL database

No budget for additional infrastructure

Need ability to add new preferences without schema changes

Prefer simple solution: JSONB column in users table with partial indexing for queries.
If this needs to scale to 1M users, then consider caching.
```

💡 **What changed:** Added scale and constraints to guide toward appropriate simplicity.

* * *

### Mistake 2: Ignoring Team's Existing Tech Stack

**Description:** AI recommends new technologies not used by the team, increasing cognitive load and maintenance burden.

**Realistic Scenario:** Team uses Java Spring. AI suggests Node.js for a new microservice without reason.

❌ **Wrong Prompt:**

```TEXT
How to implement real-time notifications?
```

⚠️ **Why it is wrong:** AI may suggest WebSockets with Node.js/Socket.io instead of leveraging existing tech stack.

✅ **Better Prompt:**

```TEXT
Implement real-time notifications within existing tech stack.

Current stack:

Backend: Java Spring Boot 3.2

Frontend: React 18

Message broker: RabbitMQ (already used for async tasks)

Deployment: Kubernetes

Prefer solutions using Spring WebSocket (STOMP) over RabbitMQ or Server-Sent Events (SSE) if simpler. Avoid introducing new languages or infrastructure unless absolutely necessary.
```

💡 **What changed:** Constrained to existing stack to avoid fragmentation.

* * *

### Mistake 3: AI Recommends Anti‑Patterns (Distributed Monolith)

**Description:** AI suggests microservice boundaries that create distributed monoliths with tight coupling.

**Realistic Scenario:** AI suggests splitting payment service into 10 microservices that all need to call each other synchronously.

❌ **Wrong Prompt:**

```TEXT
Design microservices for payment system
```

⚠️ **Why it is wrong:** AI may create services that are highly coupled, requiring distributed transactions and complex orchestration.

✅ **Better Prompt:**

```TEXT
Design microservices for payment system following Domain-Driven Design.

Guidelines:

Services should be loosely coupled, communicating asynchronously where possible

Identify bounded contexts: Payment Processing, Fraud Detection, Refunds, Reporting

Prefer eventual consistency over distributed transactions

Each service should own its data (no shared databases)

Avoid synchronous dependencies between services

Start with modular monolith until boundaries are proven

Generate service boundaries with API contracts and data ownership.
```

💡 **What changed:** Added principles to prevent distributed monolith anti-pattern.

* * *

### Mistake 4: No Consideration of Data Consistency

**Description:** AI proposes solutions without addressing consistency requirements between services.

**Realistic Scenario:** AI suggests separate services for orders and inventory without discussing eventual consistency implications.

❌ **Wrong Prompt:**

```TEXT
Split orders and inventory into separate services
```

⚠️ **Why it is wrong:** No discussion of how to handle order placement when inventory is temporarily inconsistent.

✅ **Better Prompt:**

```TEXT
Split orders and inventory into separate services with consistency requirements.

Consistency requirements:

When order placed, inventory must be reserved

Inventory can be eventually consistent (5 sec max)

Order confirmation must show reserved stock

Need to handle inventory service outage during order placement

Options:

Saga pattern with compensating transactions

Outbox pattern with idempotent consumers

Reserve stock synchronously, update asynchronously

Current system: 1000 orders/day, PostgreSQL. Prefer pragmatic approach with transactional outbox.
```

💡 **What changed:** Addressed consistency and failure scenarios upfront.

* * *

### Mistake 5: AI Suggests New Services When Existing Would Suffice

**Description:** AI recommends building new services instead of extending existing ones, increasing operational complexity.

**Realistic Scenario:** AI suggests new "audit-log" microservice when existing logging infrastructure could be extended.

❌ **Wrong Prompt:**

```TEXT
Design audit logging system for compliance
```

⚠️ **Why it is wrong:** AI may suggest new service without considering existing ELK stack or database.

✅ **Better Prompt:**

```TEXT
Design audit logging system leveraging existing infrastructure.

Current infrastructure:

Centralized logging: Elasticsearch (already used)

Message queue: Kafka (already used for events)

Retention: 90 days in Elasticsearch

Requirements:

Compliance: audit trail for sensitive operations

Immutable logs (WORM storage)

Searchable by user, operation, timestamp

10K events/second peak

Prefer: write audit events to Kafka with schema registry, index in Elasticsearch with restricted delete permissions. Avoid creating new service if existing pipeline can be extended.
```

💡 **What changed:** Leveraged existing infrastructure to avoid operational overhead.

* * *

## Summary & Best Practices

*   **Start simple** and scale only when needed.
    
*   **Stick to your team’s existing tech stack** unless there’s a compelling reason to change.
    
*   **Avoid microservices** until you have clear bounded contexts and can handle eventual consistency.
    
*   **Explicitly address data consistency** and failure scenarios.
    
*   **Reuse existing infrastructure** instead of creating new services.
    

**Good architecture is about balance. Use AI to explore options, but always weigh them against your real constraints.**

* * *