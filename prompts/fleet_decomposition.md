# Fleet Decomposition — Parallelizing the Integration

After the developer approves the integration plan, decompose the work into parallel agent tracks. This guide defines how to structure those tracks for maximum parallelism while respecting true dependencies.

## Core principle

An integration has a natural dependency graph. Only serialize tasks that truly depend on each other. Everything else can run in parallel.

## Standard integration tracks

### Track A — Client & config foundation (runs immediately, in parallel with Track B)

**Agent A1: Downstream client**
- Create the HTTP or gRPC client for the downstream service
- Implements connection setup, authentication, base URL configuration, timeout handling
- Exposes a clean interface (interface/protocol/abstract class) that the service layer will use
- Does NOT contain business logic — only transport concerns
- Files: `src/clients/{service}_client.{ext}`, `src/clients/{service}_client_interface.{ext}`

**Agent A2 (parallel with A1): Config struct / config registration**
- Create the configuration struct or class for the new client
- Add new environment variables to `.env.example` with descriptions
- Register the new config in the service's config loading mechanism
- Files: `src/config/{service}_config.{ext}`, `.env.example` (updated)

### Track B — Service layer (starts after Track A completes)

**Agent B1: Service layer**
- Create the service/repository that uses the client from Track A
- Implements business logic: which endpoints to call, how to map request/response types
- Handles error translation (downstream errors → domain errors)
- Applies retry/fallback logic as specified by the developer
- Wires the client into dependency injection / factory / container
- Files: `src/services/{service}_service.{ext}`, DI wiring file (updated)

### Track C — Tests & documentation (starts after Track B completes)

**Agent C1: Unit tests**
- Write unit tests for the service layer from Track B
- Mocks the client interface (not the real downstream service)
- Covers: happy path, error cases, retry behavior, edge cases (empty responses, timeouts)
- Files: `src/services/{service}_service_test.{ext}`

**Agent C2 (parallel with C1): Integration tests**
- Write integration or contract tests if requested
- Tests against a real or wiremock/stub endpoint
- Covers: authentication works, request/response shapes match, error codes are handled
- Files: `src/integration/{service}_integration_test.{ext}`

**Agent C3 (parallel with C1, C2): Documentation**
- Update API docs, README, or runbooks with the new integration
- Add the new env vars to deployment docs
- Document the service layer's public interface
- Files: `docs/api.md` (updated), `README.md` (updated if needed)

## Dependency graph

```
Track A (A1 + A2 in parallel)
    │
    ▼
Track B (B1 — service layer + DI)
    │
    ▼
Track C (C1 + C2 + C3 all in parallel)
```

## Per-agent task specification

Each agent MUST receive a task spec containing:

```
## Agent Task: {Track and Agent ID}

### Goal
{One sentence: what this agent creates}

### Files to create
- `{exact/path/to/file.ext}` — {purpose}

### Files to modify
- `{exact/path/to/existing/file.ext}` — {what to add/change}

### Context provided
- Downstream service: {name}
- Relevant endpoints: {list}
- Auth mechanism: {type and credential env var}
- Request/response types: {key types}
- Developer's framework: {language + framework}
- Error handling approach: {specified approach}

### Acceptance criteria
- [ ] File compiles / parses without errors
- [ ] All methods/functions have the correct signatures
- [ ] Error cases are handled as specified
- [ ] No hardcoded credentials or URLs (use config)
- [ ] Follows existing code patterns in the codebase
```

## Language-specific patterns

### Go
- Client: struct implementing an interface, constructor returns `(Client, error)`
- Service layer: struct with interface, injected via constructor
- DI: register in wire/fx/dig provider
- Tests: `_test.go` files using `testing` + `testify/mock`

### Java (Spring Boot)
- Client: `@Component` or `@Service` class using `RestTemplate`/`WebClient`/`FeignClient`
- Service layer: `@Service` interface + implementation
- DI: Spring auto-wiring via `@Autowired` / constructor injection
- Tests: JUnit 5 + Mockito

### Node.js / TypeScript
- Client: class implementing an interface, instantiated with config
- Service layer: class injected via constructor (NestJS `@Injectable()` or manual DI)
- DI: NestJS module / tsyringe / manual factory
- Tests: Jest with `jest.mock()`

### Python (FastAPI)
- Client: class with `httpx.AsyncClient` or `requests.Session`
- Service layer: class with dependency injection via FastAPI `Depends()`
- DI: FastAPI dependency injection system
- Tests: pytest + `pytest-mock` / `respx` for HTTP mocking

### Ruby (Rails)
- Client: PORO (plain Ruby object) or use Faraday
- Service layer: service object in `app/services/`
- DI: passed as constructor argument or accessed via class method
- Tests: RSpec + WebMock

## Handling partial parallelism

If the developer's codebase or framework requires sequential steps (e.g., schema migrations before service layer), adjust the tracks accordingly. The tracks above are a starting template — adapt to the real dependency graph.

## What to include in the plan presented to the user

Before launching agents, show the developer:

```
Integration Plan: {downstream service} → {current service}

Track A (starting immediately, in parallel):
  Agent A1 — Client:  creates {file}
  Agent A2 — Config:  creates {file}, updates {file}

Track B (after Track A):
  Agent B1 — Service: creates {file}, updates {file}

Track C (after Track B, all in parallel):
  Agent C1 — Unit tests:    creates {file}
  Agent C2 — Integration tests: creates {file}
  Agent C3 — Docs:          updates {file}

Total: {N} files created, {M} files modified
Estimated tracks: A → B → C (3 sequential stages, max parallelism within each)

Proceed?
```

Do not start any agents until the user confirms.
