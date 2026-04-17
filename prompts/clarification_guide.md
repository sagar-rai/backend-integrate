# Clarification Guide — Questions Before Planning

Before generating any integration plan or writing any code, Copilot MUST ask the developer these questions. Present the analysis summary first, then ask questions.

## When to ask

After completing the integration analysis (see `prompts/integration_analysis.md`), present your synthesis to the user, then say:

> "Before I create the integration plan, I need to understand your current service. Could you answer these questions?"

Ask questions conversationally — you can group related ones together, but always cover all required questions.

## Required questions (must ask all)

### 1. Language and framework

> "What language and framework is your current service written in?"

Examples to offer: Go + Gin/Echo/Fiber, Java + Spring Boot, Node.js + Express/Fastify/NestJS, Python + FastAPI/Django/Flask, Ruby + Rails, Rust + Axum, PHP + Laravel

**Why it matters:** Determines client library patterns, error handling idioms, DI patterns, and file structure.

### 2. Integration scope

> "Which specific capabilities from {downstream service} do you need? I found: [list endpoints/methods]. Are all of these in scope, or just some?"

**Why it matters:** Keeps the integration focused. Don't build what isn't needed.

### 3. Existing client patterns

> "Does your service already have a pattern for HTTP clients or gRPC clients? (e.g., a base HTTP client wrapper, a middleware chain, a specific client interface to implement)"

Follow-up if yes: "Can you point me to an example file?"

**Why it matters:** The new client should match existing patterns, not introduce a new one.

### 4. Error handling

> "How should errors from {downstream service} be handled in your service?"

Options to suggest:
- Propagate the error up (return it to the caller)
- Return a fallback/default value on error
- Retry with exponential backoff (up to N times)
- Return a domain-specific error type
- Panic/fatal (for infrastructure-level failures only)

**Why it matters:** Error handling is the most common source of integration bugs.

### 5. Testing requirements

> "What testing do you need for this integration?"

Options:
- Unit tests with mocked downstream client (most common)
- Integration tests against a real or stubbed endpoint
- Contract tests (e.g., Pact)
- No tests needed right now

Follow-up: "Which testing framework? (e.g., Go testing + testify, JUnit 5, Jest, pytest, RSpec)"

**Why it matters:** Test scaffolding needs to be planned as part of the integration, not bolted on.

### 6. Service/repository layer patterns

> "Does your codebase follow a service layer or repository layer pattern? Should the new integration live in a service, a repository, or directly in handlers?"

Follow-up if yes: "Can you tell me which directory? (e.g., `internal/services/`, `src/services/`, `app/services/`)"

**Why it matters:** The integration must fit the existing architecture, not fight it.

### 7. Configuration management

> "How is configuration managed in your service?"

Options:
- Environment variables read directly (e.g., `os.Getenv`, `process.env`)
- Config struct populated at startup from env vars
- Config file (YAML/JSON/TOML)
- Secrets manager (AWS SSM, HashiCorp Vault, GCP Secret Manager)
- Kubernetes secrets/configmaps

**Why it matters:** The new config values (URLs, API keys, timeouts) need to be added the right way.

## Optional questions (ask if context suggests they're relevant)

### Circuit breaker / retry
> "Is there a circuit breaker or retry library already in use?" (e.g., resilience4j, go-retry, polly, cockatiel)

Ask if: the downstream service has documented rate limits or error rates, or if the integration is to a critical path.

### Observability
> "Do you need OpenTelemetry tracing or metrics instrumentation on the new client?"

Ask if: the user's codebase shows existing OTel usage, or the downstream service is latency-sensitive.

### Sync vs async
> "Should this integration be synchronous (direct HTTP/gRPC call) or asynchronous (publish to a message queue / event bus)?"

Ask if: the context file mentions event-driven patterns, or the operation is potentially slow (e.g., payment processing, report generation).

### API versioning
> "Does your service need to support multiple versions of the downstream API, or just the latest?"

Ask if: the downstream service has multiple API versions documented.

## Decision gate

**DO NOT proceed to plan creation until:**
- All 7 required questions have been answered
- At least the relevant optional questions have been addressed
- You have confirmed the integration scope with the user

If the user asks you to skip questions, acknowledge the request but explain:
> "I want to make sure the integration fits your codebase correctly — skipping these could mean I write code in the wrong pattern or miss critical error handling. Can we at least cover questions 1 (language/framework), 4 (error handling), and 7 (config management)?"

## After questions are answered

Summarize the answers back to the user for confirmation:

```
Got it! Here's what I'll build:

- Language/framework: {answer}
- Scope: {endpoints/methods}
- Client pattern: {new pattern / follow existing in {path}}
- Error handling: {approach}
- Tests: {types + framework}
- Location: {directory/layer}
- Config: {approach}

Ready to generate the integration plan?
```

Wait for user confirmation before generating the plan.
