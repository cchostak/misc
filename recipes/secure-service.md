# secure-service.md

> Recipe for scaffolding or elevating a production-facing network service.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> This recipe defines the architectural and operational expectations for a secure service. It does not require a specific language, framework, cloud, CI platform, container runtime, or deployment platform.

## Purpose

Use this recipe when the repository implements a long-running service that accepts network requests or messages and produces responses, events, or side effects.

Typical examples:

- HTTP/REST APIs
- gRPC services
- internal platform services
- webhook receivers
- backend-for-frontend services
- event-driven workers with externally triggered inputs
- service adapters and integration gateways

Do **not** apply this recipe to libraries, pure CLIs, static sites, one-shot batch jobs, firmware, or repositories that do not operate as a long-running service.

The goal is not to make the service "enterprise-looking".

The goal is to produce a service that is:

- explicit about trust boundaries;
- safe by default;
- observable;
- testable;
- deployable;
- resilient to expected failures;
- operationally diagnosable;
- easy for another engineer or agent to understand.

## Architectural invariants

A secure service SHOULD satisfy these invariants unless the repository has a documented reason not to.

### 1. Inputs are untrusted by default

Anything crossing a process, network, queue, file, user, tenant, or external system boundary must be treated as untrusted until validated.

This includes HTTP request data, headers, query parameters, path parameters, request bodies, uploaded files, message payloads, webhook payloads, externally sourced database values, service-to-service responses, and remotely supplied configuration.

Validation must happen at a clear boundary.

Do not rely on downstream components to "probably reject" invalid data.

### 2. Authentication and authorization are separate concerns

Authentication answers: **Who or what is calling?**

Authorization answers: **Is that identity allowed to perform this action on this resource?**

Do not combine the two into ad-hoc conditional logic.

Authorization checks must be applied at the point where protected behavior is performed, not only at routing or UI layers.

### 3. Service identity must be explicit

Where the deployment environment supports workload identity, prefer it over long-lived static credentials.

Do not embed credentials in source, images, test fixtures, or committed configuration.

### 4. Failure is part of the design

The service must have intentional behavior for dependency timeout, dependency unavailability, malformed input, authorization failure, partial failure, cancellation, graceful shutdown, exhausted resources, and duplicate requests where relevant.

Do not treat these as exceptional edge cases if they are normal distributed-systems failure modes.

### 5. Runtime state and durable state are distinct

In-memory process state must not be assumed durable unless the service is explicitly designed as a single-process stateful system.

Durable state belongs in an appropriate backing system.

If the service is intended to scale horizontally, correctness must not depend on requests consistently reaching the same process unless session affinity is an explicit architectural requirement.

### 6. Deployment artifacts are immutable

Build once. Promote the same immutable artifact through environments.

Do not rebuild source separately for each environment.

Prefer artifact digests, immutable versions, or equivalent identifiers over mutable deployment labels.

### 7. Configuration and secrets are not source code

Environment-specific configuration must be supplied through an explicit configuration mechanism.

Secrets must use a secret-delivery mechanism appropriate to the runtime.

Do not use source-controlled `.env` files for real secrets.

### 8. Observability is part of correctness

A production service that cannot be diagnosed is incomplete.

Logs, metrics, traces, and health signals should be sufficient to answer:

- Is it running?
- Is it ready to serve?
- Is it succeeding?
- Is it slow?
- What dependency is failing?
- Which operation is affected?
- Is the failure isolated or systemic?

### 9. Security controls must fail safely

A security control failure must not silently become authorization success.

Examples include policy-engine unavailability, identity-verification failure, malformed tokens, missing permission data, invalid signatures, and failed security-configuration loads.

The default behavior should preserve the intended security boundary.

### 10. Operational simplicity is preferred

Do not introduce service meshes, distributed caches, message brokers, sidecars, custom gateways, policy engines, additional datastores, or orchestration platforms unless the service requirements justify them.

## Service boundary definition

Before implementation, identify and document:

- service purpose;
- consumers;
- exposed protocols;
- trust boundaries;
- authenticated actors;
- authorization model;
- owned data;
- external dependencies;
- emitted artifacts/events;
- runtime assumptions;
- deployment model.

A simple representation is sufficient.

```text
Client
  |
  | HTTPS
  v
Service
  |
  +--> Primary datastore
  |
  +--> External dependency
  |
  +--> Metrics / logs
```

If different trust zones exist, show them.

## API and protocol design

Use the repository's established protocol and conventions.

For new HTTP APIs:

- use stable, predictable resource naming;
- use standard HTTP semantics where practical;
- use appropriate status codes;
- distinguish client errors from server failures;
- validate request content types;
- bound request sizes;
- avoid leaking internal implementation details;
- define pagination for potentially unbounded collections;
- define idempotency semantics for retry-sensitive writes where needed.

For RPC or message protocols:

- define schema compatibility rules;
- preserve backward compatibility where required;
- reject malformed messages explicitly;
- document retry and duplicate-delivery semantics;
- version contracts deliberately.

Do not introduce API versioning before there is a concrete compatibility need, but do not accidentally make breaking public contracts either.

## Input validation

Validate as close to the trust boundary as practical.

Validation should include, where relevant:

- type;
- required fields;
- ranges;
- lengths;
- enum membership;
- encoding;
- content type;
- identifier format;
- path safety;
- URL safety;
- resource ownership;
- cross-field invariants.

Prefer structured schemas or typed validation over scattered string checks.

Reject ambiguous input when ambiguity affects security or correctness.

## Authentication

If authentication is required:

- use established standards and libraries;
- validate token signatures correctly;
- validate issuer and audience where relevant;
- validate expiration and not-before claims;
- handle key rotation;
- reject unsupported algorithms;
- avoid custom cryptographic protocols;
- make authentication failure observable without leaking sensitive details.

Do not log raw tokens.

For service-to-service authentication, prefer workload identity or short-lived credentials where supported.

## Authorization

Authorization must be explicit and testable.

Prefer a clear policy model such as RBAC, ABAC, ownership-based access control, policy-engine evaluation, or capability-based authorization.

Authorization tests should include:

- allowed action;
- denied action;
- missing identity;
- insufficient privilege;
- cross-tenant/cross-owner access where relevant;
- policy dependency failure where applicable.

Do not rely on front-end visibility as authorization.

## Multi-tenancy

If the service is multi-tenant:

- define the tenant boundary explicitly;
- propagate tenant identity through internal operations;
- scope queries by tenant;
- scope caches by tenant;
- scope authorization by tenant;
- prevent cross-tenant metadata leakage;
- test negative cross-tenant cases.

Tenant identity must not be accepted blindly from untrusted request data if it can be derived from authenticated identity.

## Secrets

Secrets include passwords, tokens, API keys, signing keys, private keys, database credentials, and encryption keys.

Requirements:

- do not hard-code secrets;
- do not print secrets in logs;
- do not place secrets in command-line arguments when a safer mechanism exists;
- rotate where the platform allows;
- minimize secret lifetime;
- minimize secret scope;
- avoid sharing one credential across unrelated services;
- fail clearly when required secrets are absent.

Local development may use obviously fake values or local secret-generation scripts, but they must not resemble production credentials.

## Configuration

Configuration must be explicit and validated.

Prefer a typed configuration object or equivalent.

At startup:

- validate required values;
- validate ranges and formats;
- reject contradictory settings;
- fail early on configuration that makes safe operation impossible.

Do not silently ignore unknown security-sensitive configuration where doing so could hide an operator mistake.

## External dependencies

For every remote dependency, define:

- timeout;
- cancellation behavior;
- retry policy;
- backoff policy;
- connection limits;
- expected error handling;
- fallback behavior if any;
- observability.

Retries must be bounded.

Retries should use backoff and jitter when appropriate.

Do not retry validation failures, authorization failures, deterministic application errors, or unsafe non-idempotent operations unless idempotency is provided.

Dependency failure must not cause unbounded thread, process, memory, or request growth.

## Idempotency

For operations likely to be retried, determine whether duplicate execution is safe.

If duplicate execution is unsafe, consider idempotency keys, request deduplication, transactional constraints, unique operation identifiers, or outbox/inbox patterns where justified.

Do not introduce idempotency infrastructure without an actual retry/duplicate risk.

## Persistence and data access

If the service owns persistent data:

- use parameterized/safe query APIs;
- define schema changes explicitly;
- use migrations;
- define transaction boundaries;
- preserve backward compatibility during rolling deployment where relevant;
- avoid destructive migrations coupled tightly to one deployment step;
- test migrations;
- consider backup/recovery requirements.

Do not expose database internals as public API contracts without a reason.

If the service does not need durable persistence, do not add a database.

## Concurrency

Where the runtime is concurrent:

- identify shared mutable state;
- avoid unsafe global mutation;
- bound worker concurrency;
- bound queue sizes;
- handle cancellation;
- avoid unbounded goroutine/thread/task creation;
- make shutdown deterministic.

Concurrency should be driven by actual throughput or isolation needs, not introduced for sophistication.

## Resource limits

Protect the service from accidental or hostile exhaustion.

As applicable, bound:

- request body size;
- header size;
- uploaded file size;
- batch size;
- pagination size;
- recursion depth;
- decompression ratio;
- connection count;
- worker count;
- queue depth;
- timeout duration;
- in-memory cache size.

Reject or shed load intentionally rather than failing unpredictably under resource pressure.

## Health endpoints

Long-running services should expose operational health signals where the runtime/deployment environment can use them.

### Liveness

Answers: **Is this process alive enough that restarting it may help?**

Do not make liveness depend on every downstream service.

### Readiness

Answers: **Can this process safely receive traffic right now?**

Readiness may depend on critical initialization or mandatory dependencies.

### Startup

Use a startup probe/signal where slow initialization would otherwise conflict with liveness behavior.

Health responses must not expose secrets, credentials, stack traces, or sensitive internal topology.

## Graceful lifecycle

The service must handle normal termination intentionally.

As applicable:

1. receive termination signal;
2. stop accepting new work;
3. mark not-ready;
4. drain in-flight requests/work;
5. stop background workers;
6. flush required telemetry/state;
7. close resources;
8. exit within a bounded period.

Test graceful shutdown where practical.

## Logging

Prefer structured logs.

Logs should include useful operational context such as request/correlation ID, operation, result, duration, dependency, and error category.

Do not log sensitive values.

Do not log complete request/response payloads by default.

Avoid duplicate logging of the same error at every call layer.

## Metrics

Where production operation matters, expose metrics that allow basic service health to be understood.

Consider:

- request count;
- error count/rate;
- latency;
- active requests;
- dependency latency/errors;
- queue depth;
- worker saturation;
- resource exhaustion signals.

Avoid unbounded high-cardinality labels such as raw user IDs, request IDs, arbitrary URLs, or stack traces.

## Tracing

Use distributed tracing when the service participates in meaningful distributed request flows and the environment supports it.

Trace propagation must not be trusted as authorization.

Do not place secrets or sensitive payloads in spans.

## Error model

Define predictable error behavior.

Internal errors should preserve context for operators while exposing only appropriate information to clients.

Avoid returning stack traces, SQL errors, filesystem paths, secret values, internal hostnames, or implementation details that provide no client value.

## Security headers and transport

For HTTP services exposed beyond a trusted local boundary:

- use TLS at an appropriate layer;
- redirect or reject plaintext traffic where relevant;
- configure CORS deliberately rather than using `*` by reflex;
- use secure cookie attributes where cookies exist;
- set relevant browser security headers for browser-facing responses.

Do not cargo-cult browser headers onto non-browser APIs without understanding their applicability.

## Webhooks

If receiving webhooks:

- authenticate the sender;
- verify signatures using the provider's documented scheme;
- verify timestamps/nonces where applicable;
- use the raw body when signature schemes require it;
- protect against replay where necessary;
- validate event type and schema;
- make duplicate delivery safe;
- avoid performing slow work synchronously when reliability requires queuing.

Webhook signature failure must result in rejection.

## Outbound requests and SSRF

If user-controlled input can influence outbound network destinations:

- parse URLs with a safe library;
- constrain allowed schemes;
- consider hostname allowlists;
- restrict internal/private address access where needed;
- handle redirects deliberately;
- re-evaluate destination after redirects;
- protect cloud metadata endpoints;
- apply timeouts and response-size limits.

Do not rely on string prefix checks as the sole SSRF control.

## File handling

If the service accepts files:

- limit size;
- validate type/content appropriately;
- generate safe storage names;
- avoid trusting client filenames;
- prevent path traversal;
- isolate processing;
- clean temporary files;
- consider archive bombs;
- avoid unsafe automatic execution/rendering.

Use stronger isolation for hostile document or media processing when justified.

## Serialization and deserialization

Use safe, schema-aware serializers.

Do not enable unsafe object deserialization features for untrusted data.

Set reasonable nesting/size limits where parser exhaustion is possible.

## Container image

If the service is containerized:

- prefer multi-stage or equivalent minimal builds;
- include only runtime requirements;
- run as non-root where practical;
- use a read-only filesystem where practical;
- avoid package managers/debug tools in the runtime image unless needed;
- pin base image versions/digests appropriately;
- do not bake secrets into layers;
- use an explicit entrypoint;
- handle signals correctly;
- define required ports only;
- include OCI metadata where useful.

Containerization is optional if another deployment artifact is more suitable.

## Runtime/deployment security

Where supported by the target environment, prefer:

- non-root execution;
- dropped Linux capabilities;
- no privilege escalation;
- read-only root filesystem;
- scoped filesystem mounts;
- explicit network policy;
- workload identity;
- least-privilege service accounts;
- resource requests/limits.

Do not add platform-specific controls if the deployment platform is not part of the repository's requirements.

## Build and supply chain

For distributable service artifacts:

- lock dependencies;
- pin build toolchains sufficiently for reproducibility;
- avoid mutable dependency sources where practical;
- build in CI from reviewed source;
- produce immutable artifacts;
- record artifact digest/version;
- generate SBOM/provenance/signatures when required by applied profiles or project risk.

Signing without verification is incomplete.

If artifacts are signed, define where verification occurs.

## CI expectations

CI should invoke the same core checks available locally.

As appropriate:

```text
format check
    ->
lint/static analysis
    ->
unit tests
    ->
integration tests
    ->
build
    ->
security checks
    ->
artifact validation
```

Do not give CI deployment credentials unless CI genuinely owns deployment.

Prefer separating build from deployment when the delivery architecture supports it.

## Test strategy

The test suite should reflect real service risk.

### Unit tests

Test deterministic business logic and boundary behavior.

Avoid tests that merely reproduce mocks.

### Handler/API tests

Test:

- valid request;
- malformed request;
- unsupported method/content type where relevant;
- authentication failure;
- authorization failure;
- not-found behavior;
- domain validation failure;
- internal error mapping.

### Integration tests

Use real dependencies where their integration behavior matters.

Prefer ephemeral/local disposable dependencies.

### Security regression tests

Where applicable, include regression tests for:

- authorization bypass;
- cross-tenant access;
- injection;
- path traversal;
- SSRF;
- unsafe redirects;
- webhook signature bypass;
- malformed authentication data;
- excessive input size.

### Smoke tests

A smoke test should prove the built/deployed service can:

1. start;
2. become ready;
3. serve one expected request;
4. reject one invalid or unauthorized request;
5. shut down cleanly.

### Fuzz/property testing

Consider fuzzing or property-based testing for parsers, protocol handlers, request validation, serialization, and security-sensitive boundary code.

Use risk, not fashion, to determine depth.

## Local development

A developer should have obvious commands to:

- bootstrap dependencies;
- run the service;
- run tests;
- run static checks;
- format code;
- build the artifact;
- run a smoke test.

Use the ecosystem's idiomatic tooling.

Do not require a full production-like cluster for ordinary unit development unless the system truly cannot be exercised otherwise.

## Documentation

The service documentation should answer:

- What does this service do?
- Who calls it?
- How do I run it locally?
- How do I configure it?
- How do I test it?
- How do I build it?
- How is it deployed?
- What dependencies does it require?
- What are its trust boundaries?
- How do I determine if it is healthy?
- Where are logs/metrics/traces emitted?
- What are the common failure modes?

Document real configuration only.

Do not invent production URLs, owners, security contacts, or SLAs.

## Architecture decisions

Use ADRs for consequential choices such as protocol selection, authentication model, authorization model, persistence technology, deployment model, queue/event architecture, significant dependencies, or unusual resilience strategies.

Do not create ADRs for trivial implementation choices.

## Recommended repository shape

Follow ecosystem conventions first.

A generic service may resemble:

```text
.
├── AGENTS.md
├── README.md
├── src/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── smoke/
├── docs/
│   └── adr/
├── config/
├── scripts/
├── .editorconfig
├── .gitignore
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

## Optional profile hooks

This recipe is intended to compose with additional profiles.

Common combinations:

```text
secure-service + zero-trust
```

For explicit workload identity, trust zones, policy enforcement, and stronger service-to-service controls.

```text
secure-service + software-supply-chain
```

For SBOM, provenance, signing, verification, admission policy, and promotion controls.

```text
secure-service + internet-facing
```

For stronger abuse resistance, edge controls, request limiting, attack-surface review, and public exposure hardening.

```text
secure-service + hostile-input
```

For parsers, upload processors, document handlers, webhook gateways, and other services where attacker-controlled input is central.

```text
secure-service + high-assurance
```

For deeper testing, independent controls, stricter release evidence, and failure-injection requirements.

Do not duplicate profile requirements inside the service implementation if a profile already owns them.

## Anti-patterns

Do not scaffold these without a concrete requirement:

- Kubernetes for a single simple service;
- service mesh;
- API gateway;
- message broker;
- Redis/cache;
- database;
- GraphQL;
- custom authentication;
- custom authorization engine;
- distributed tracing platform;
- custom retry framework;
- circuit breaker framework;
- feature flag platform;
- secrets platform;
- elaborate dependency injection container;
- "clean architecture" layers with no independent reason;
- repository/service abstractions around every function;
- microservices for internal modularity;
- fake enterprise metadata.

Do not mistake quantity of infrastructure for production readiness.

## Acceptance criteria

A secure-service implementation is not complete until the repository can demonstrate the applicable subset of the following.

### Build

- dependencies resolve reproducibly;
- the service builds successfully;
- the deployable artifact is identifiable.

### Static verification

- formatting check passes;
- lint/static checks pass;
- type checking passes where applicable;
- configuration validation passes.

### Functional behavior

- service starts successfully;
- readiness becomes healthy;
- expected request succeeds;
- malformed request is rejected;
- protected operation rejects an unauthorized caller where auth exists;
- important dependency failure produces intentional behavior.

### Tests

- unit tests pass;
- relevant integration tests pass;
- smoke test passes;
- security regression tests pass where applicable.

### Operations

- termination is handled cleanly;
- logs provide useful context;
- sensitive values are not emitted;
- metrics/health signals operate as documented.

### Security

- no real secrets are committed;
- dependency/security checks run where applicable;
- trust boundaries have explicit enforcement;
- unsafe fallback behavior is absent.

### Documentation

- local setup is documented;
- configuration is documented;
- test/build commands are documented;
- architectural/security boundaries are understandable.

## Completion evidence

When this recipe is applied, the final report should state:

1. service architecture;
2. exposed interfaces;
3. trust boundaries;
4. authentication/authorization model, or explicitly `not applicable`;
5. persistence and external dependencies;
6. resilience behavior;
7. observability provided;
8. security controls implemented;
9. tests added;
10. commands actually executed;
11. observed verification results;
12. controls intentionally not added and why;
13. assumptions and remaining risks.

Never claim that a security control, test, build, scan, deployment, or runtime behavior works unless it was actually verified or clearly marked as unverified.

## Guiding principle

A production service should be boring to operate.

Its boundaries should be explicit.

Its failure modes should be intentional.

Its permissions should be narrow.

Its behavior should be testable.

Its artifacts should be immutable.

Its state should be understood.

Its telemetry should explain what happened.

Its security should come from architecture and enforceable controls, not from the number of security tools mentioned in the repository.
