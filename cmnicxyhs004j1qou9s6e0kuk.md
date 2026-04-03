---
title: "Prompting Like a Pro – How to Talk to AI"
seoDescription: "The way you phrase your request determines the quality of AI’s output. Most developers treat AI like a search engine—asking vague questions and accept"
datePublished: 2026-04-03T03:42:14.755Z
cuid: cmnicxyhs004j1qou9s6e0kuk
slug: prompting-like-a-pro-talk-to-ai
cover: https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/e85a9f81-8ab1-4a0d-ba79-b5c91f647b1a.png
tags: ai, software-development, software-architecture, developer, devops, ai-tools

---

> The way you phrase your request determines the quality of AI’s output. Most developers treat AI like a search engine—asking vague questions and accepting the first answer. In this series post, we’ll explore five common prompting mistakes and show you how to fix them with concrete examples.

* * *

![](https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/bf532af0-07eb-439d-a1c6-e7f6d492e0d8.png align="center")

* * *

### Mistake 1: Vague Debugging Requests

**Description:** Developer provides no context while asking AI to fix an error, forcing the AI to guess.

**Realistic Scenario:** Production API failing after deployment. Developer pastes only the error message without any surrounding code or stack trace.

❌ **Wrong Prompt:**

```TEXT
Fix this error: "NullPointerException"
```

⚠️ **Why it is wrong:** No stack trace, code context, or environment details to locate the actual null reference.

✅ **Better Prompt:**

```TEST
Fix the NullPointerException in the below Spring Boot service method.

Environment: Spring Boot 3.2, Java 17, running on Kubernetes
Error stack trace:
java.lang.NullPointerException at com.example.UserService.getUserById(UserService.java:42)
at com.example.UserController.getUser(UserController.java:18)

Code snippet:
public User getUserById(String id) {
return userRepository.findById(id).get();
}

Expected behavior: Return a default User object or throw a custom NotFoundException when user doesn't exist.
```

💡 **What changed:** Added stack trace pinpointing exact line, code context, and expected behavior so AI can suggest proper null handling.

* * *

### Mistake 2: Asking for Code Without Context

**Description:** Requesting implementation without providing existing codebase patterns, leading to inconsistent style and architecture.

**Realistic Scenario:** Junior developer asks AI to implement authentication without knowing the project uses JWT with specific custom claims format.

❌ **Wrong Prompt:**

```TEXT
Write me authentication for a Spring Boot app
```

⚠️ **Why it is wrong:** No information about existing auth mechanism, user store, or security requirements.

✅ **Better Prompt:**

```TEXT
Implement authentication for Spring Boot 3.2 using our existing pattern.

Our codebase currently uses:

JWT with custom claims: userId, tenantId, roles

User details loaded from PostgreSQL via Spring Security

OAuth2 client for Google login

Custom AuthenticationProvider for username/password

Add a new endpoint /api/auth/login that accepts email/password, validates against database, and returns JWT with our standard claims format. Follow existing package structure under com.example.auth.
```

💡 **What changed:** Provided existing patterns, tech stack details, and specific requirements to generate consistent code.

* * *

### Mistake 3: No Iteration After First Response

**Description:** Accepting first AI response without iterating to refine, test, or adapt to edge cases.

**Realistic Scenario:** Developer needs to parse large CSV files. AI gives initial solution that works for small files but fails with memory issues in production.

❌ **Wrong Prompt:**

```TEXT
How to parse a CSV file in Python?
```

*Developer copies first solution without further questions.*

⚠️ **Why it is wrong:** AI provided a basic solution that may not handle large files, encoding issues, or malformed data.

✅ **Better Prompt (Iterative Approach):**

```TEXT
Round 1: "Write Python code to parse a 5GB CSV file efficiently. Use generators and avoid loading entire file into memory."

Round 2 (after review): "Now add error handling for malformed rows, skip corrupted lines, and log errors to a file."

Round 3: "Optimize for speed using pandas with chunksize. Include progress bar for processing 10M rows."
```

💡 **What changed:** Iterative refinement addressed scale, error handling, and performance progressively.

* * *

### Mistake 4: Ignoring AI’s Clarifying Questions

**Description:** AI asks for clarification but developer proceeds with incomplete information, resulting in incorrect output.

**Realistic Scenario:** AI asks which database technology is being used for a query optimization request. Developer ignores and uses generic advice.

❌ **Wrong Prompt:**

```TEXT
Optimize this query: SELECT * FROM orders WHERE customer_id = 123
```

*AI: "Which database are you using?"* *Developer: (ignores) "Just optimize it"*

⚠️ **Why it is wrong:** Optimization strategies differ vastly between PostgreSQL, MySQL, MongoDB, etc. Generic advice may even degrade performance.

✅ **Better Prompt:**

```TEXT
Optimize this query for PostgreSQL 15:

SELECT * FROM orders WHERE customer_id = 123;

Current execution plan shows sequential scan on orders table (500K rows). customer_id has low cardinality (10K distinct values). Write the appropriate index and rewritten query if needed.
```

💡 **What changed:** Answered AI's implicit questions upfront—database type, current performance, data distribution.

* * *

### Mistake 5: Using Outdated Examples in Prompts

**Description:** Using deprecated library versions or syntax in prompts, leading to AI generating outdated code.

**Realistic Scenario:** Developer asks for React code using class components in 2024, missing modern patterns.

❌ **Wrong Prompt:**

```TEXT
Write a React component with state and lifecycle methods
```

⚠️ **Why it is wrong:** AI may generate class components with deprecated lifecycle methods instead of modern hooks.

✅ **Better Prompt:**

```TEXT
Write a React 18 functional component using TypeScript that:

Fetches user data from /api/users on mount

Shows loading state

Uses React Query for caching

Handles errors with ErrorBoundary

Follows project pattern: hooks in separate files under src/hooks/
```

💡 **What changed:** Specified modern React version, TypeScript, preferred libraries, and project structure.

* * *

## Summary & Best Practices

*   **Always provide context:** stack traces, code snippets, environment details, expected behavior.
    
*   **Be specific about versions and libraries:** avoid ambiguous “latest” or “modern”.
    
*   **Iterate:** treat AI as a pair programmer, not a one‑shot code generator.
    
*   **Answer AI’s questions proactively** to save time.
    
*   **Keep prompts up‑to‑date** with your actual tech stack.
    

With these techniques, you’ll turn AI from a generic code machine into a precise, context‑aware assistant.

* * *