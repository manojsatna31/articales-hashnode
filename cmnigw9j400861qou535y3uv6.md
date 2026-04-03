---
title: "Performance Pitfalls – AI That Kills Your Latency"
datePublished: 2026-04-03T05:32:54.210Z
cuid: cmnigw9j400861qou535y3uv6
slug: performance-pitfalls-ai-that-kills-your-latency
cover: https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/74561732-665c-478e-ac15-3dadda87dbf0.png

---

> AI is great at generating functional code, but it often misses performance considerations. The result can be slow endpoints, database overload, and wasted cloud costs. This post covers five common performance mistakes AI makes and how to prompt for efficient solutions.

* * *

![](https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/96550c59-4905-4d66-9e3f-6560ca9725b7.png align="center")

* * *

### Mistake 1: AI‑Generated N+1 Queries

**Description:** AI generates code that performs database queries in loops, causing N+1 problem.

**Realistic Scenario:** AI generates endpoint that fetches users then loops to fetch each user's orders individually.

❌ **Wrong Prompt:**

```TEXT
Write endpoint to get users with their orders
```

⚠️ **Why it is wrong:** AI may generate `for user in users: orders = db.query("SELECT * FROM orders WHERE user_id = ?", user.id)` causing N+1 queries.

✅ **Better Prompt:**

```TEXT
Write endpoint to get users with their orders.

Performance requirements:

Fetch 100 users with their orders

Use JOIN or batch query to avoid N+1

For JPA: use FETCH JOIN or @EntityGraph

For SQL: use SELECT ... WHERE user_id IN (list)

Measure query count with logs or DataSource proxy

Example using JPA:
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

💡 **What changed:** Explicitly addressed N+1 prevention with examples.

* * *

### Mistake 2: Missing Index Recommendations

**Description:** AI generates queries without suggesting appropriate indexes for production scale.

**Realistic Scenario:** AI generates search query on unindexed column that works locally but times out in production with millions of rows.

❌ **Wrong Prompt:**

```TEXT
Write query to find orders by customer email
```

⚠️ **Why it is wrong:** AI generates `SELECT * FROM orders WHERE customer_email = ?` without mentioning index on customer\_email.

✅ **Better Prompt:**

```TEXT
Write query to find orders by customer email for production (10M orders).

Performance requirements:

Response time < 100ms

Query should use index on customer_email

Include index creation statement

Consider covering index if selecting specific columns

Generated solution:
CREATE INDEX idx_orders_customer_email ON orders(customer_email);
SELECT order_id, order_date, amount FROM orders WHERE customer_email = ?;

Add EXPLAIN ANALYZE to verify index usage.
```

💡 **What changed:** Included index creation and performance validation.

* * *

### Mistake 3: Inefficient Loops in Generated Code

**Description:** AI generates nested loops or O(n²) algorithms without considering data size.

**Realistic Scenario:** AI generates code comparing two lists with nested loops for 10K items each, causing 100M operations.

❌ **Wrong Prompt:**

```TEXT
Find users who are in both list A and list B
```

⚠️ **Why it is wrong:** AI may generate nested loops: `for a in listA: for b in listB: if a.id == b.id` O(n\*m).

✅ **Better Prompt:**

```TEXT
Find users who are in both list A (10K users) and list B (10K users).

Performance requirements:

Use HashSet for O(n) complexity

Avoid nested loops

Prefer set intersection with O(n+m)

Example:
Set<String> idsA = listA.stream().map(User::getId).collect(Collectors.toSet());
List<User> common = listB.stream().filter(u -> idsA.contains(u.getId())).collect(Collectors.toList());

Complexity: O(n+m) vs O(n*m)
```

💡 **What changed:** Provided complexity expectations and efficient approach.

* * *

### Mistake 4: No Caching Strategy from AI Suggestion

**Description:** AI suggests adding cache without specifying TTL, invalidation, or cache key strategy.

**Realistic Scenario:** AI suggests Redis cache for product catalog but doesn't handle invalidation on product updates.

❌ **Wrong Prompt:**

```TEXT
Add caching for product catalog API
```

⚠️ **Why it is wrong:** No TTL, no invalidation, cached data becomes stale.

✅ **Better Prompt:**

```TEXT
Add caching for product catalog API.

Requirements:

Cache product details by ID

TTL: 5 minutes (accepts eventual consistency)

Invalidation: when product updated via admin API, invalidate cache key

Cache key pattern: product:{id}

Handle cache stampede: use single-flight pattern

Fallback to database if cache unavailable

Current load: 10K req/s, product updates: 10/min.
Use Redis with Spring Cache (@Cacheable, @CacheEvict).
```

💡 **What changed:** Added TTL, invalidation strategy, and failure handling.

* * *

### Mistake 5: AI Ignores Connection Pool Limits

**Description:** AI generates code that creates new database connections per request, exhausting pool.

**Realistic Scenario:** AI generates code that creates new JDBC connection for each API call instead of using connection pool.

❌ **Wrong Prompt:**

```TEXT
Write function to query database in Java
```

⚠️ **Why it is wrong:** AI may generate `DriverManager.getConnection(url, user, pass)` for each call, exhausting connections under load.

✅ **Better Prompt:**

```TEXT
Write function to query database using HikariCP connection pool.

Configuration:

Maximum pool size: 20

Connection timeout: 30s

Idle timeout: 10min

Use try-with-resources to ensure connections returned to pool

Code pattern:
@Autowired private DataSource dataSource;
try (Connection conn = dataSource.getConnection();
PreparedStatement stmt = conn.prepareStatement(sql)) {
// execute
}

Never create new DriverManager connections per request.
```

💡 **What changed:** Enforced connection pool usage and proper resource management.

* * *

## Summary & Best Practices

*   **Avoid N+1 queries** by using joins or batch fetching.
    
*   **Always consider indexes** for columns used in WHERE, JOIN, ORDER BY.
    
*   **Choose efficient algorithms**—prefer O(n) over O(n²) for large data.
    
*   **Add caching with clear TTL and invalidation** to reduce database load.
    
*   **Reuse database connections** via connection pools.
    

**Performance is not a nice‑to‑have; it’s a requirement for production systems. Make it part of your AI prompts.**

* * *