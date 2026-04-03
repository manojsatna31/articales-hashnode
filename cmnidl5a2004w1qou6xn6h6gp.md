---
title: "Security Blind Spots in AI‑Generated Code"
datePublished: 2026-04-03T04:00:16.637Z
cuid: cmnidl5a2004w1qou6xn6h6gp
slug: security-blind-spots-in-ai-generated-code
cover: https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/ff30febf-5e49-4204-86fe-e2ddee9b5f51.png

---

> AI models are trained on vast amounts of public code, which often includes insecure practices. Without careful prompting and review, AI can introduce critical security vulnerabilities. This post covers five common security mistakes and how to avoid them.

* * *

![](https://cdn.hashnode.com/uploads/covers/69cc8029e4688e4edd77b94e/da1d4fe4-2ef7-45bd-839f-32d8b115a2da.png align="center")

* * *

### Mistake 1: AI‑Generated Hardcoded Secrets

**Description:** AI includes hardcoded API keys, passwords, or tokens in generated code.

**Realistic Scenario:** AI generates AWS S3 client code with hardcoded access keys in the example.

❌ **Wrong Prompt:**

```TEXT
Write code to upload file to S3
```

⚠️ **Why it is wrong:** AI may generate `aws_access_key_id = "AKIAIOSFODNN7EXAMPLE"` which developers might not replace.

✅ **Better Prompt:**

```TEXT
Write code to upload file to S3 using AWS SDK v2.

Security requirements:

NEVER hardcode credentials

Use DefaultCredentialsProvider (IAM roles in production)

For local dev, use environment variables or ~/.aws/credentials

Include comment that credentials must never be committed to repo

Use IAM roles with least privilege principle

Add validation that credentials are properly configured before upload.
```

💡 **What changed:** Explicit security requirements prevent credential exposure.

* * *

### Mistake 2: Unsanitized Input Handling

**Description:** AI generates code that doesn't validate or sanitize user input, enabling injection attacks.

**Realistic Scenario:** AI generates REST endpoint that directly concatenates user input into shell commands.

❌ **Wrong Prompt:**

```TEXT
Write API endpoint to run system command based on user input
```

⚠️ **Why it is wrong:** Direct command injection vulnerability.

✅ **Better Prompt:**

```TEXT
Write API endpoint that runs predefined system commands based on user selection.

Requirements:

User selects from dropdown of allowed commands (reboot, status, logs)

NEVER directly interpolate user input into shell

Use whitelist of allowed commands

Validate input against whitelist

Log all command executions for audit

Run with least privileged user

If implementing file operations, use allowlist for paths and validate input doesn't contain path traversal (../).
```

💡 **What changed:** Added whitelist, validation, and secure coding practices.

* * *

### Mistake 3: No SQL Injection Awareness

**Description:** AI generates SQL queries with string concatenation instead of parameterized queries.

**Realistic Scenario:** AI generates dynamic query builder for search endpoint with user input directly concatenated.

❌ **Wrong Prompt:**

```TEXT
Write search function to query users by name
```

⚠️ **Why it is wrong:** AI may generate `"SELECT * FROM users WHERE name = '" + userName + "'"` creating SQL injection vector.

✅ **Better Prompt:**

```TEXT
Write search function to query users by name using JPA Repository.

Security requirements:

Use parameterized queries (JPA @Query with ?1 or :name)

Never concatenate user input into query strings

Escape special characters for LIKE queries

Use projections to avoid returning sensitive fields

Add input validation (length, allowed characters)

Example using Spring Data JPA:
@Query("SELECT u FROM User u WHERE u.name LIKE %:name%")
List<User> findByName(@Param("name") String name);
```

💡 **What changed:** Enforced parameterized queries and input validation.

* * *

### Mistake 4: Overly Permissive IAM/Service Accounts

**Description:** AI suggests broad IAM roles or permissions without least privilege principle.

**Realistic Scenario:** AI generates Lambda IAM role with `*` permissions for simplicity.

❌ **Wrong Prompt:**

```TEXT
Create IAM role for Lambda function to access S3 and DynamoDB
```

⚠️ **Why it is wrong:** AI may suggest `"Action": "s3:*"` or `"Resource": "*"` instead of scoped permissions.

✅ **Better Prompt:**

```TEXT
Create IAM role for Lambda function following least privilege.

Required actions:

S3: GetObject on specific bucket: my-app-bucket/uploads/*

S3: PutObject on specific bucket: my-app-bucket/processed/*

DynamoDB: GetItem, PutItem on table: user-sessions

DO NOT use wildcard resources or actions unless absolutely necessary.
Include condition for MFA if accessing sensitive data.
Use managed policies only when they match least privilege.

Generate Terraform/IAM policy JSON.
```

💡 **What changed:** Scoped permissions to specific resources and actions.

* * *

### Mistake 5: Exposing Internal Endpoints via AI Suggestions

**Description:** AI generates actuator or admin endpoints that expose sensitive data without authentication.

**Realistic Scenario:** AI suggests adding Spring Boot Actuator endpoints for monitoring without securing them.

❌ **Wrong Prompt:**

```TEXT
Add health checks and monitoring to Spring Boot app
```

⚠️ **Why it is wrong:** AI may suggest adding actuator endpoints that expose env, heap dumps, or shutdown without authentication.

✅ **Better Prompt:**

```TEXT
Add Spring Boot Actuator for monitoring.

Security requirements:

Expose only /health and /info to unauthenticated users

Secure /env, /metrics, /beans behind authentication (admin role)

Disable /shutdown endpoint completely

Use different management port not exposed to internet

Add rate limiting to actuator endpoints

Ensure no sensitive data exposed in /env

Current security: Spring Security with JWT. Add actuator-specific security config.
```

💡 **What changed:** Explicit security controls prevent exposure of sensitive endpoints.

* * *

## Summary & Best Practices

*   **Never hardcode secrets**—use environment variables, secret managers, or IAM roles.
    
*   **Always sanitize and validate** user input, especially for commands, SQL, and file paths.
    
*   **Use parameterized queries** to prevent SQL injection.
    
*   **Apply least privilege** to IAM roles, service accounts, and database users.
    
*   **Secure monitoring endpoints** with authentication and proper network isolation.
    

**Security is not an afterthought; it must be part of your AI interaction workflow.**

* * *