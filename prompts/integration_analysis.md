# Integration Analysis — Synthesizing Downstream Service Context

After downloading files from the downstream service repository, follow this analysis protocol to build a complete picture of what needs to be integrated.

## Reading order

Read files in this priority order — stop when you have enough context for the user's specific goal:

1. **Integration context file** (`INTEGRATION.md`, `DOWNSTREAM.md`, etc.) — the service owner's explicit guide
2. **README.md** — service overview, capabilities, and quick start
3. **API contracts** — proto files, OpenAPI/Swagger specs, AsyncAPI, GraphQL schemas
4. **Client examples** — SDK code, client libraries, example call patterns
5. **Configuration samples** — `.env.example`, `config.example.*`
6. **Data models** — DTOs, request/response structs, schema definitions
7. **Makefile / docker-compose** — how to run and what ports/services exist

## What to extract

### Service overview
- What does this service do?
- What problem does it solve?
- What are its primary capabilities?

### API surface
For each relevant endpoint or RPC method:
- **Name / path**: e.g., `POST /v1/payments/charge` or `rpc ProcessPayment`
- **Purpose**: what it does
- **Request shape**: fields, types, required vs optional
- **Response shape**: fields, types, error cases
- **Authentication**: how to authenticate this specific call
- **Side effects**: what it creates, updates, or triggers

### Authentication & security
- Authentication mechanism: API key (header name?), Bearer token, OAuth 2.0 (which flow?), mTLS, HMAC signature, Basic auth
- Where credentials come from: env var names, secrets manager paths
- Token expiry and refresh patterns (if applicable)
- Any IP allowlisting or network requirements

### Required configuration
List every environment variable or config value the calling service needs:
```
ENV_VAR_NAME        | purpose                          | example value
--------------------|----------------------------------|------------------
PAYMENT_SVC_URL     | Base URL of the payment service  | https://api.acme.com
PAYMENT_SVC_API_KEY | API key for authentication       | (from secrets manager)
PAYMENT_SVC_TIMEOUT | Request timeout in milliseconds  | 5000
```

### Data models
Key structs/types the calling service must know about:
- Request types (what to send)
- Response types (what to parse)
- Error types (what errors look like)
- Any enums, constants, or shared types

### Operational characteristics
- **Rate limits**: requests per second/minute, burst limits
- **Timeouts**: recommended client timeout values
- **Retry policy**: which errors are retryable, recommended backoff strategy
- **Circuit breaker hints**: failure thresholds, fallback behavior
- **SLAs**: p99 latency, availability guarantees (if documented)

### Client patterns
From any SDK or client example code:
- How is the client initialized?
- How are requests made (sync/async)?
- How are errors returned and handled?
- Are there connection pooling patterns?

## Synthesis output

After reading, produce a structured summary for the user:

```
## Downstream Service Analysis: {Service Name}

### What it does
[2-3 sentence summary]

### Relevant to your goal: "{user's stated goal}"
[Which specific endpoints/methods are needed]

### Authentication
[Mechanism and where credentials come from]

### Required environment variables
[Table of env vars]

### Data models you'll need
[Key request/response types]

### Integration approach
[Recommended: HTTP REST vs gRPC, sync vs async, client pattern]

### Operational notes
[Rate limits, timeouts, retry recommendations]
```

Present this summary to the user BEFORE asking clarifying questions — it shows what you found and lets the user correct any misunderstandings.

## What to do when context is incomplete

| Missing info | Action |
|---|---|
| No auth docs found | Note it; ask user if they have credentials/auth docs |
| No data model files | Infer from API examples in README; flag as uncertain |
| No env var examples | List what you inferred from the code; ask user to confirm |
| Conflicting docs (README vs spec) | Note the conflict; ask user which is authoritative |
| Private endpoints not documented | Ask user if they have internal docs to share |
| gRPC but no proto files found | Ask user for the proto file path; it may be in a different repo |

## Scope discipline

Only analyze what is relevant to the user's specific goal. If they say "I need to integrate the payment charge endpoint," don't deeply analyze the refund, subscription, or webhook endpoints — note their existence but stay focused on the stated goal.
