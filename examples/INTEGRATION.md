<!--
  INTEGRATION.md — Template for downstream service owners

  Copy this file into your service repo and fill in the placeholders.
  Replace everything in {curly braces} with your actual values.
  Remove sections that don't apply to your API style (REST vs gRPC vs GraphQL).
  Remove HTML comment blocks before committing.

  See docs/writing-integration-context.md for guidance on each section.
-->

# {Service Name} — Integration Guide

> For developers integrating {Service Name} into their backend service.

**API style:** {REST / gRPC / GraphQL}  
**Transport:** {HTTPS / internal gRPC plaintext / internal gRPC TLS}  
**Last updated:** {YYYY-MM-DD}

---

## Overview

{One paragraph: what this service does, who uses it, and its primary capabilities.}

---

## Quick Start

<!-- REST: show a working curl command -->
```bash
export {SERVICE}_URL="{https://api.example.com/v1 OR grpc-target:50051}"
export {SERVICE}_API_KEY="your-key"

curl -X POST "${{SERVICE}_URL}/{primary-endpoint}" \
  -H "Authorization: Bearer ${{SERVICE}_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"field": "value"}'
```

<!-- gRPC: describe the stub call instead of curl -->
<!-- Connect to {SERVICE_GRPC_TARGET} (see Connection section), call {ServiceName}.{PrimaryRPC} with google.protobuf.Empty or {RequestType}. -->

Expected response:
```json
{"id": "abc123", "status": "ok"}
```

---

## Authentication / Connection

**Method:** {Bearer token / API key header / OAuth 2.0 / gRPC plaintext (no auth) / mTLS}

<!-- REST -->
```
Authorization: Bearer {YOUR_API_KEY}
```

<!-- gRPC -->
<!-- Internal service — use insecure.NewCredentials() (Go) or usePlaintext() (Scala/Java). No TLS. -->
<!-- k8s target: {service-name}.{namespace}.svc.cluster.local:{port} -->

---

## Config Variables

| Variable | Description | Example | Required |
|---|---|---|---|
| `{SERVICE}_URL` | Base URL or gRPC target | `https://api.example.com/v1` | ✅ Yes |
| `{SERVICE}_API_KEY` | Auth key | (from secrets manager) | ✅ Yes |
| `{SERVICE}_TIMEOUT_MS` | Request timeout in ms | `5000` | ✅ Yes |

---

## Endpoints / RPCs

<!-- REST: one row per endpoint -->
| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/{resource}` | {What it does} |
| `GET` | `/v1/{resource}/{id}` | {What it does} |

<!-- gRPC: service catalog + RPC reference -->
<!--
| Service (fully-qualified) | RPCs | Proto path |
|---|---|---|
| `com.example.{service}.v1.{ServiceName}` | {N} | `proto/{service}/v1/service.proto` |

| RPC | Request | Response |
|---|---|---|
| `{MethodName}` | `{RequestType}` | `{ResponseType}` |
| `{MethodName}` | `google.protobuf.Empty` | `{ResponseType}` |
-->

**Request:**
```json
{"{field}": "string — {description}", "{field2}": "integer — {description}"}
```

**Response:**
```json
{"id": "string", "status": "string", "{field}": "{value}"}
```

---

## Error Handling

<!-- REST -->
| Status | Error Code | Retryable? | Action |
|---|---|---|---|
| 429 | `RATE_LIMITED` | ✅ Yes | Respect `Retry-After` |
| 500 | `INTERNAL_ERROR` | ✅ Yes | Backoff, max 3 attempts |
| 400 | `INVALID_REQUEST` | ❌ No | Fix the request |
| 401 | `UNAUTHORIZED` | ❌ No | Check credentials |
| 404 | `NOT_FOUND` | ❌ No | Resource does not exist |

<!-- gRPC: replace HTTP status with gRPC status codes -->
<!-- NOT_FOUND, INTERNAL → retryable; INVALID_ARGUMENT, UNAUTHENTICATED → not retryable -->

Retry strategy: initial delay `100ms`, multiplier `2x`, max delay `5s`, max attempts `3`.

---

## Agent Notes

<!-- Key facts for Copilot — keep this section, it improves generated code quality -->

- **Transport**: {plaintext — no TLS / HTTPS / mTLS}
- **RPC type**: {all unary / server streaming / bidirectional streaming}
- **Load balancing**: {round_robin / pick_first}
- **Port**: {50051 / 443 / 8080}
- **Rate limits**: {N req/min per key — or "none"}
- **Idempotency**: {Idempotency-Key header supported / not supported}
- **Error behavior**: {describe any non-obvious error codes}

---

## Support

- **Slack**: #{service-name}-integrations
- **Issues**: {GitHub issues link}
- **On-call**: {runbook link}
