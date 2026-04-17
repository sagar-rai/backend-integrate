# Writing a Great Integration Context File

This guide is for **downstream service owners** — the teams who maintain APIs that other services call.

When a developer uses `backend-integrate` to integrate your service, Copilot reads your integration file to understand how to build the integration. The better your file, the better the generated code.

---

## Why this file matters

Copilot uses your integration file to:

- Discover what APIs you expose and which ones are relevant to the developer's use case
- Understand transport, auth, and connection details without guessing
- Generate correct, compilable client code in the caller's language
- Know which errors to handle and how (retryable vs fatal)
- Determine what environment variables to add to the calling service

A vague or incomplete integration file produces generic, often wrong code. A specific, concrete one produces code that compiles and works on the first try.

---

## General principles

**Be specific, not aspirational.** Write what your service *actually* does today.

**Tables over prose.** Copilot extracts structured information much more reliably from tables. If you're listing endpoints, error codes, config variables, or RPCs — use a table.

**Provide working examples.** Show actual request/response payloads or client stubs that compile. A partial, real example is far more valuable than a complete pseudocode sketch.

**Keep it focused.** Your integration file is not your full API reference. It needs to contain the minimum information required for a developer to write a working integration.

**Include an Agent Notes section.** A section called `## Agent Notes` containing bullet-point facts about your service significantly improves Copilot's output. See the sections below for what to include.

---

## REST services

For REST APIs, include these sections in order:

### 1. Overview

One paragraph: what the service does, who uses it, primary capabilities. Include:
- API style (`REST`)
- Base URL for production and staging
- Authentication method (Bearer token, API key, OAuth, mTLS)

### 2. Quick Start

A working `curl` command that makes a real call and produces a real response. This is the single most valuable thing you can include — developers copy this directly; Copilot uses it to understand request shape, headers, and URL structure.

```bash
curl -X POST "https://api.your-service.com/v1/resource" \
  -H "Authorization: Bearer $YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"field": "value"}'
```

Include the expected response JSON inline.

### 3. Authentication

State the exact mechanism precisely:
- **Bearer token**: show the exact header name and format
- **API key**: show whether it goes in a header, query param, or body
- **OAuth**: show the token exchange request and token lifetime
- **Internal/no auth**: say so explicitly — tells Copilot not to add auth logic

Include the exact environment variable names to use. Copilot will use these names in generated config code.

### 4. Endpoint reference

A table with one row per endpoint (Method, Path, Purpose). For endpoints integrators are likely to call, add a subsection with:
- Request body as JSON with field descriptions inline
- Response body as JSON with field descriptions
- Error codes specific to this endpoint

### 5. Error handling

Two tables — retryable and non-retryable errors:

| Status | Error Code | Retryable? | Action |
|---|---|---|---|
| 429 | `RATE_LIMITED` | ✅ Yes | Respect `Retry-After` header |
| 500 | `INTERNAL_ERROR` | ✅ Yes | Exponential backoff, max 3 attempts |
| 400 | `INVALID_REQUEST` | ❌ No | Fix the request |
| 404 | `NOT_FOUND` | ❌ No | Resource does not exist |

Without this table, Copilot will either retry everything or nothing.

### 6. Config variables

A table of every environment variable an integrating service needs:

| Variable | Description | Example | Required |
|---|---|---|---|
| `MY_SVC_URL` | Base URL | `https://api.your-service.com/v1` | ✅ Yes |
| `MY_SVC_API_KEY` | Auth key | (from secrets manager) | ✅ Yes |
| `MY_SVC_TIMEOUT_MS` | Request timeout | `5000` | ✅ Yes |

### 7. Agent Notes

A bullet list of facts that don't fit elsewhere but are critical for generating correct code:

- Rate limit numbers and window sizes
- Idempotency key requirements
- Any headers required on every request
- Timeout recommendations
- Known gotchas or non-obvious behaviors

---

## gRPC services

gRPC integrations require more structural information than REST. Include these sections:

### 1. Overview

One paragraph: what the service does, transport details, and RPC type. Include:
- Internal vs external (determines whether TLS is needed)
- gRPC port
- k8s service name and namespace (e.g. `my-svc.my-ns.svc.cluster.local:50051`)
- RPC type: all unary, bidirectional streaming, server streaming, or mixed

### 2. Service catalog

A table listing every gRPC service exposed. This is the first thing Copilot reads to understand scope — it must be complete and accurate.

| # | Service (fully-qualified) | RPCs | Proto path (repo-relative) |
|---|---|---|---|
| 1 | `com.example.myservice.v1.FooService` | 4 | `proto/foo/v1/service.proto` |
| 2 | `com.example.myservice.v1.BarService` | 2 | `proto/bar/v1/service.proto` |

### 3. RPC reference

For each service, a table of RPCs with exact message type names:

| RPC | Request | Response |
|---|---|---|
| `GetFoo` | `GetFoo.Request` | `Foo` |
| `ListFoos` | `google.protobuf.Empty` | `ListFoos.Response` |

### 4. Proto dependency graph

**This is the most critical section for gRPC integrations.** Copilot needs to know exactly which `.proto` files to fetch to generate a working client.

List three categories:

**Service protos** — files containing `service` definitions (entry points):
```
proto/foo/v1/service.proto
proto/bar/v1/service.proto
```

**Model protos** — files containing `message` definitions used by the services:
```
proto/foo/v1/models.proto
proto/shared/common.proto
```

**Dependency order** — which models are imported by which services, so Copilot knows the full transitive closure of files to pull. An ASCII diagram works well:

```
common.proto
    └── models.proto
            └── service.proto   ← start here for client generation
```

Then provide a per-service recipe: "To call FooService, pull these N files in this order."

### 5. Working client stub

A minimal, compilable client in the language most commonly used by your integrators. Even a partial example (connection setup + one RPC call) is far more valuable than no example.

For **Go**, show:
- How to create the gRPC connection (`grpc.NewClient` with the correct credentials)
- How to construct the stub (`pb.NewFooServiceClient(conn)`)
- One real RPC call with error handling

For **Scala/ScalaPB**, note any non-default codegen flags (e.g. `flatPackage = true`) — these cause hard-to-debug import errors if missed.

If you publish a pre-built client artifact, say so — integrators can skip proto codegen entirely.

### 6. Agent Notes

A bullet list of integration facts Copilot must know before generating code:

- **Transport**: plaintext (`insecure.NewCredentials()`) or TLS — be explicit
- **RPC types**: all unary, streaming — be explicit
- **Load balancing**: recommended policy (e.g. `round_robin`)
- **Error codes**: which gRPC status codes your service returns and when
- **k8s target format**: how to construct the service target in production vs local
- **Client artifact**: if you publish one, name it here
- **Codegen flags**: any non-default flags that will break compilation if missed
- **gRPC port**: the port the service listens on

---

## GraphQL services

For GraphQL APIs, include:

### 1. Overview

What the service does. Include the endpoint URL, HTTP method (GET or POST), and auth method.

### 2. Schema overview

Don't paste the full schema. Describe the main types and list the queries and mutations integrators are most likely to use.

### 3. Key queries and mutations

For each important operation, show:
- Operation name and type (query/mutation)
- Variables it accepts (name, type, required)
- A working example query/mutation body
- The response shape

### 4. Auth

How to authenticate. Confirm the exact header name and format.

### 5. Agent Notes

- Whether the schema is introspectable at runtime
- Rate limits
- Any operation-specific gotchas

---

## Sections required for all API styles

### Config variables

Always include a config table regardless of API style. See format above.

### Support

Where integrators go when something breaks:

```markdown
- **Slack**: #my-service-integrations
- **Issues**: https://github.com/org/my-service/issues
- **On-call**: link to runbook
```

---

## What to avoid

- **Internal implementation details** — integrators need to know how to call your service, not how it works inside
- **Real credentials** — never commit API keys, tokens, or secrets
- **Long prose over tables** — Copilot parses tables much more reliably than paragraphs
- **Duplicating your README** — INTEGRATION.md has a different audience and goal

---

## Using the template

Copy [`examples/INTEGRATION.md`](../examples/INTEGRATION.md) from this repo into your service repository and fill in the placeholders. Remove sections that don't apply to your API style.

The template is intentionally concise. Add detail where it helps — the service catalog, proto dependency graph, and agent notes sections have the highest impact on integration quality.
