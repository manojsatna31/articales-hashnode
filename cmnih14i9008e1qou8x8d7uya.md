---
title: "Testing Illusions – AI‑Generated Tests That Lie"
datePublished: 2026-04-03T05:36:40.979Z
cuid: cmnih14i9008e1qou8x8d7uya
slug: testing-illusions-ai-generated-tests-that-lie
cover: https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/e42e0a5b-c761-4440-9c7e-5c7b3e034ebc.png

---

> AI can generate tests quickly, but quantity doesn’t equal quality. Many AI‑generated tests either assert the wrong thing, miss edge cases, or are not idempotent. This post shows you five testing pitfalls and how to prompt for meaningful test suites.

* * *

![](https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/fce01af5-5f33-48f9-8430-2033f1c8d094.png align="center")

* * *

### Mistake 1: AI Generates Tests That Pass Incorrectly

**Description:** AI generates tests that assert incorrect expected values, passing but not validating actual behavior.

**Realistic Scenario:** AI generates test for calculator that expects wrong result but test passes.

❌ **Wrong Prompt:**

```TEXT
Write tests for add function
```

⚠️ **Why it is wrong:** AI may generate `assertEquals(3, add(1, 1))` expecting 3 when correct result is 2, test passes but validation fails.

✅ **Better Prompt:**

```TEXT
Write tests for add(int a, int b) function with verification.

Requirements:

Test with known inputs: (1,1)=2, (-1,1)=0, (0,0)=0

Test boundary: Integer.MAX_VALUE + 1

Verify actual result matches expected

Use parameterized tests to avoid duplication

Include property-based tests for random inputs

After writing tests, verify they fail if implementation is wrong, then pass with correct implementation.

```

💡 **What changed:** Added explicit expected values and verification.

* * *

### Mistake 2: No Negative Test Cases

**Description:** AI generates only happy path tests, missing error conditions.

**Realistic Scenario:** AI generates tests for user registration API only for success, missing validation failures.

❌ **Wrong Prompt:**

```TEXT
Write tests for user registration endpoint

```

⚠️ **Why it is wrong:** No tests for invalid email, short password, duplicate user, or database errors.

✅ **Better Prompt:**

```TEXT
Write tests for user registration endpoint covering:

Happy path:

Valid email/password returns 201

Negative cases:

Duplicate email returns 409 Conflict

Invalid email format returns 400

Password < 8 chars returns 400

Missing fields returns 400

Database timeout returns 503

Edge cases:

Email with maximum length

Unicode in email

SQL injection attempts

Use JUnit 5 and MockMvc (Spring) with parameterized tests.

```

💡 **What changed:** Comprehensive test coverage including error scenarios.

* * *

### Mistake 3: Mocking Too Much or Too Little

**Description:** AI generates tests that over-mock (testing implementation details) or under-mock (requiring external services).

**Realistic Scenario:** AI generates test mocking repository calls but not testing actual database interactions.

❌ **Wrong Prompt:**

```TEXT
Write unit test for UserService

```

⚠️ **Why it is wrong:** AI may mock UserRepository for all tests, missing integration issues like query correctness.

✅ **Better Prompt:**

```TEXT
Write test strategy for UserService:

Unit tests (mock repository):

Test business logic, validation, error handling

Mock repository to return known states

Verify service calls repository with correct parameters

Integration tests (@DataJpaTest):

Test actual database queries with testcontainers

Verify query correctness with real PostgreSQL

Test transaction boundaries

Contract tests:

Test API layer with @WebMvcTest

Verify request/response formats

Test with real database in CI, not in-memory H2.

```

💡 **What changed:** Balanced testing strategy with appropriate test types.

* * *

### Mistake 4: Tests Lack Assertions or Assert Wrong Behavior

**Description:** AI generates tests that verify only that code runs, not that behavior is correct.

**Realistic Scenario:** AI generates test calling method but not checking return value or side effects.

❌ **Wrong Prompt:**

```TEXT
Write test for sendNotification method

```

⚠️ **Why it is wrong:** Test may call method without verifying notification was sent or content is correct.

✅ **Better Prompt:**

```TEXT
Write test for sendNotification(String userId, String message) method.

Assertions:

Verify notification service called with correct parameters

Verify message content matches expected

Verify timestamp added to notification record

Verify error logged if user not found

Mock NotificationService and verify:
notificationService.send(eq(userId), contains("Welcome"));
verify(notificationService, times(1)).send(any(), any());

Also test failure scenarios: retry on failure, dead letter queue.

```

💡 **What changed:** Added specific assertions and verification of interactions.

* * *

### Mistake 5: AI‑Generated Tests Not Idempotent

**Description:** Tests that leave data in database or state that affects subsequent tests.

**Realistic Scenario:** AI generates test that creates user but doesn't clean up, causing duplicate key failures in next test.

❌ **Wrong Prompt:**

```TEXT
Write test for user creation

```

⚠️ **Why it is wrong:** Test creates user without cleanup; second test run fails due to constraint violation.

✅ **Better Prompt:**

```TEXT
Write test for user creation with idempotent setup/teardown.

Requirements:

Use @Transactional to rollback changes after test

Or use @BeforeEach/@AfterEach to clean test data

Use unique test data (UUIDs) to avoid collisions

For integration tests, truncate tables or use database cleanup utility

Example:
@Transactional
@SpringBootTest
class UserServiceTest {
@Test
void createUser_shouldSucceed() {
// test - transaction rolls back automatically
}
}

```

💡 **What changed:** Ensured tests are isolated and repeatable.

* * *

## Summary & Best Practices

*   **Verify expected values** carefully—don’t assume AI gets them right.
    
*   **Include negative test cases** (validation errors, exceptions, edge inputs).
    
*   **Balance mocking**—use real dependencies for integration tests.
    
*   **Assert behavior, not just execution**—verify outcomes.
    
*   **Keep tests idempotent** by cleaning up after each test.
    

**Good tests are the safety net that allows you to trust AI‑generated code. Invest in them.**

* * *