# api-platform.md

> Recipe for scaffolding or elevating a shared API platform.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md`, `control-plane.md`,
> `identity-platform.md`, `developer-platform.md`, `secure-service.md`,
> `library-sdk.md`, `cli.md`, and profiles such as `zero-trust-service.md`,
> `software-supply-chain.md`, `internet-facing.md`, `high-assurance.md`, or
> `hostile-input.md` where appropriate.
>
> This recipe defines architecture expectations for platforms that expose,
> govern, route, secure, observe, version, and operate APIs across multiple
> teams, services, tenants, or external consumers.
>
> It does not require a specific API gateway, service mesh, ingress controller,
> protocol, cloud, schema registry, developer portal, or API management vendor.

## Purpose

Use this recipe when the platform provides shared API capabilities to multiple
service owners or consumers.

Typical examples:

- enterprise API platforms;
- internal API platforms;
- public API programs;
- partner API platforms;
- API gateway/control-plane architectures;
- service exposure platforms;
- API governance platforms;
- multi-tenant API management;
- service-to-service API platforms;
- developer-platform API layers;
- AI/tool API gateways.

The goal is not to centralize every network request through one product.

The goal is to create a coherent API platform where teams can answer:

- What is the API contract?
- Who owns it?
- Who may call it?
- How is identity established?
- Where is authorization enforced?
- How are versions evolved?
- How are consumers migrated?
- How are traffic and abuse bounded?
- How are failures surfaced?
- What belongs in the gateway versus the service?
- How is runtime behavior observed?
- How are public, partner, and internal APIs distinguished?

---

## Core principle

An API platform should standardize contracts and enforcement without becoming
the place where application semantics go to die.

A useful model is:

```text
Consumers
    |
    v
API Edge / Entry
    |
    +--> authentication
    +--> authorization context
    +--> rate limits / quotas
    +--> routing
    +--> protocol enforcement
    +--> observability
    |
    v
Service / Domain API
    |
    +--> business authorization
    +--> domain validation
    +--> application semantics
    |
    v
Backend capabilities
```

The platform may enforce cross-cutting concerns.

The application still owns domain behavior.

Network routing is not API governance.

---

## Architectural invariants

An API platform SHOULD satisfy these invariants unless there is a documented
reason not to.

### 1. API ownership is explicit

Every API should have an accountable owner.

Ownership includes:

- contract;
- lifecycle;
- security;
- documentation;
- migration;
- support.

Do not create "shared APIs" nobody owns.

### 2. The contract is independent of the gateway product

The API contract should survive implementation changes.

Consumers should depend on:

- protocol;
- schema;
- semantics;
- lifecycle guarantees.

They should not depend unnecessarily on vendor-specific gateway behavior.

### 3. Authentication and authorization are distinct

The platform may establish identity at the edge.

The target service still needs resource-aware authorization where domain
semantics require it.

### 4. API versions evolve deliberately

Breaking changes need explicit lifecycle.

Do not make consumers discover breaking behavior through runtime errors.

### 5. Gateway policy is cross-cutting, not domain-specific

Good gateway concerns include:

- TLS;
- authentication;
- coarse authorization;
- quotas;
- request limits;
- routing;
- protocol normalization;
- telemetry.

Avoid placing business workflows or domain rules in the gateway.

### 6. Internal does not mean trusted

Internal APIs should authenticate and authorize according to risk.

Do not use network location as the sole trust boundary.

### 7. Failure semantics are part of the contract

Clients need predictable:

- status/error codes;
- retry guidance;
- idempotency behavior;
- timeout behavior.

### 8. APIs are automation contracts

Human-readable documentation is necessary but not sufficient.

Machine-readable schemas should be authoritative where practical.

### 9. Observability is end-to-end

The platform should correlate edge, gateway, service, and backend behavior.

Do not stop observability at the gateway.

### 10. Traffic policy is bounded

Rate limits, quotas, retries, timeouts, and payload limits should prevent one
consumer from destabilizing shared services.

### 11. Data exposure is intentional

Public, partner, internal, and privileged APIs should have explicit exposure
classification.

### 12. Compatibility is measured against consumers

A syntactically valid schema change can still be behaviorally breaking.

---

## API classes

Classify APIs explicitly.

A useful taxonomy may include:

```text
public
partner
internal
privileged/admin
control-plane
service-to-service
event-driven
```

Different classes may require different:

- authentication;
- rate limiting;
- documentation;
- support;
- lifecycle;
- exposure;
- audit.

Do not apply one identical policy to every API.

---

## North-south versus east-west

Distinguish:

```text
north-south
    = client/user/partner -> platform/service

east-west
    = service -> service
```

These often have different:

- identity;
- network path;
- trust;
- latency requirements;
- failure behavior.

Do not assume the same gateway must mediate both.

---

## Public APIs

Public APIs should have explicit controls for:

- authentication;
- abuse;
- quotas;
- rate limits;
- payload limits;
- documentation;
- versioning;
- support.

Compose with `internet-facing.md` where available.

---

## Partner APIs

Partner APIs should define:

- partner identity;
- tenant mapping;
- scopes;
- quotas;
- support expectations;
- offboarding.

Do not treat all partners as one shared trust domain.

---

## Internal APIs

Internal APIs should still define:

- caller identity;
- audience;
- authorization;
- versioning;
- ownership.

Avoid "it's inside the VPC" as the security model.

---

## Privileged/admin APIs

Separate high-risk administrative APIs.

Use stronger controls for:

- global config;
- tenant admin;
- policy changes;
- destructive operations;
- credential operations.

Do not expose admin endpoints through the same anonymous/public path.

---

## API contract

An API contract includes more than endpoints.

It may include:

- resource model;
- request/response schema;
- error model;
- authentication;
- authorization expectations;
- pagination;
- idempotency;
- retryability;
- rate limits;
- versioning;
- deprecation;
- latency expectations.

---

## Schema

Prefer explicit schemas.

Common choices include:

- OpenAPI;
- Protobuf;
- GraphQL schema;
- JSON Schema;
- AsyncAPI for event-oriented APIs.

The exact format matters less than having one authoritative contract.

---

## Schema ownership

The service/domain owner should own API semantics.

The platform may own:

- linting;
- policy;
- publication;
- compatibility checks.

Do not centralize semantic ownership into the platform team.

---

## Schema-first versus code-first

Either can work.

What matters:

- source of truth is clear;
- generated artifacts stay synchronized;
- compatibility is tested.

Do not maintain two authoritative schemas manually.

---

## Generated clients

Generated SDKs may improve consistency.

Compose with `library-sdk.md`.

Generated clients should not hide:

- retries;
- timeouts;
- authentication;
- pagination.

---

## API naming

Use consistent resource and operation naming.

Prefer domain language over infrastructure language.

Avoid exposing internal storage tables or implementation class names.

---

## Resource modeling

For resource-oriented APIs:

- stable IDs;
- clear lifecycle;
- predictable verbs;
- explicit ownership.

Avoid creating action endpoints for operations that are naturally resource
state changes unless action semantics are clearer.

---

## Command-style APIs

Some domains are action-oriented.

Examples:

```text
rotate-key
promote-release
revoke-session
```

Use explicit command semantics where resource CRUD would be misleading.

---

## HTTP semantics

If using HTTP:

- use methods intentionally;
- use status codes consistently;
- make idempotency clear;
- define caching behavior;
- define redirects.

Do not tunnel every operation through POST without reason.

---

## gRPC semantics

If using gRPC:

- define service boundaries;
- stable message schemas;
- error mapping;
- deadlines;
- streaming behavior.

Treat Protobuf evolution as a compatibility contract.

---

## GraphQL

If using GraphQL:

- bound query complexity;
- authorize fields/resources;
- control introspection as appropriate;
- prevent N+1 amplification.

Do not assume one endpoint simplifies security.

---

## WebSockets / streaming APIs

Define:

- authentication;
- session lifetime;
- backpressure;
- reconnect;
- heartbeat;
- message ordering;
- cancellation.

Do not leave long-lived connection lifecycle undefined.

---

## Protocol selection

Choose protocol based on consumer needs.

Evaluate:

- interoperability;
- latency;
- streaming;
- tooling;
- schema evolution;
- browser support;
- operational visibility.

Do not select gRPC, GraphQL, or REST by fashion.

---

## API gateway role

A gateway may handle:

- routing;
- TLS termination;
- authentication;
- quotas;
- basic policy;
- transformation;
- telemetry.

Keep the role narrow enough to remain operable.

---

## Gateway anti-role

Do not turn the gateway into:

- business workflow engine;
- application database;
- domain authorization oracle for every resource;
- large transformation layer;
- hidden integration platform.

This creates brittle coupling.

---

## Gateway versus service authorization

A useful split:

```text
gateway:
  coarse-grained access
  token validation
  route-level policy

service:
  domain/resource authorization
  ownership checks
  business rules
```

Do not rely only on route-level authorization for resource-sensitive actions.

---

## Authentication at edge

The edge may validate:

- OAuth/OIDC;
- API keys;
- mTLS;
- signed requests;
- workload identity.

Compose with `identity-platform.md`.

---

## Identity propagation

If the gateway authenticates the caller, propagate identity through a trusted
mechanism.

Avoid trusting raw inbound headers such as:

```text
X-User
X-Role
X-Tenant
```

unless the edge strips and recreates them and backend trust is explicit.

---

## Audience validation

Every token consumer should validate audience.

Do not accept a token minted for another service merely because signature is
valid.

---

## API keys

API keys may be appropriate for some integrations.

Treat them as bearer secrets.

Scope:

- owner;
- API;
- environment;
- quota;
- lifetime.

Do not use one global API key for an organization.

---

## mTLS

mTLS authenticates peers.

It does not provide application authorization by itself.

Use identity from the certificate as policy input.

---

## Signed requests

Signed requests may help high-value APIs.

Define:

- signing algorithm;
- canonicalization;
- nonce/timestamp;
- replay window.

Do not invent custom signing lightly.

---

## Authorization model

Authorization should consider:

```text
subject
action
resource
tenant
environment
context
```

Do not stop at "authenticated".

---

## Policy enforcement point

Identify where authorization is enforced:

- edge;
- service;
- both;
- external PDP.

The service remains responsible for business/resource-aware enforcement unless
the domain genuinely belongs elsewhere.

---

## Central policy engine

A shared policy engine may improve consistency.

But it adds:

- runtime dependency;
- latency;
- availability coupling;
- policy distribution complexity.

Use where justified.

---

## Local policy enforcement

Local policy can reduce runtime dependency.

Keep policy distribution/versioning explicit.

---

## Authorization caching

If authorization decisions are cached:

- scope;
- TTL;
- invalidation;
- revocation lag.

Do not cache high-risk decisions indefinitely.

---

## Tenant routing

If tenant determines route/backend:

- derive tenant from trusted identity/context;
- validate route ownership;
- isolate caches.

Do not trust tenant solely from URL or user-supplied header.

---

## Multi-tenancy

API platforms should isolate:

- credentials;
- quotas;
- logs;
- policies;
- rate limits;
- routing.

One tenant should not be able to infer another tenant's configuration or data.

---

## Rate limiting

Rate-limit by appropriate dimensions:

- identity;
- API key;
- tenant;
- route;
- operation;
- source where useful.

Avoid IP-only limits for authenticated high-value APIs.

---

## Quotas

Quotas may bound:

- requests/day;
- tokens;
- payload bytes;
- concurrent sessions;
- expensive operations.

Expose quota exhaustion clearly.

---

## Burst control

Separate sustained quota from burst limits.

Use token bucket/leaky bucket or equivalent where appropriate.

Do not let short bursts collapse backend capacity.

---

## Concurrency limits

Some expensive APIs need per-caller or per-route concurrency limits.

Useful for:

- model inference;
- exports;
- bulk processing;
- report generation.

---

## Request size limits

Bound:

- body size;
- header size;
- multipart parts;
- compressed payload size.

Do not accept unbounded uploads.

---

## Decompression bombs

For compressed input:

- bound expanded size;
- validate formats;
- reject abusive ratios.

Compose with `hostile-input.md`.

---

## Response size

Large responses should use:

- pagination;
- streaming;
- asynchronous export.

Do not return unbounded arrays by default.

---

## Pagination

Define stable pagination.

Possible models:

- cursor;
- continuation token;
- offset.

Cursor-based pagination often handles mutable datasets better.

Document ordering.

---

## Filtering

Define allowed filters.

Avoid arbitrary query-language injection into backend stores.

---

## Sorting

Provide deterministic sort semantics.

Avoid default ordering that changes unpredictably.

---

## Search

Search APIs should clarify:

- relevance;
- consistency;
- pagination;
- freshness.

Do not imply transactional consistency if backed by eventual indexes.

---

## Idempotency

For side-effecting operations, define whether retries are safe.

Possible approaches:

- naturally idempotent PUT;
- idempotency key;
- operation resource;
- precondition/version.

Do not require clients to guess.

---

## Idempotency keys

If supported, define:

- scope;
- TTL;
- conflict behavior;
- reuse rules.

Bind key to request semantics.

---

## Conditional requests

Use revisions/ETags where lost updates matter.

Support:

```text
If-Match
```

or equivalent.

Do not allow blind overwrite of concurrent changes.

---

## Optimistic concurrency

On conflict:

- return machine-usable error;
- expose current revision;
- require client re-read/merge.

---

## Asynchronous operations

Long-running API operations should often return operation identity.

Example:

```text
POST /exports
    ->
202 Accepted
    ->
operation resource
```

Do not keep HTTP requests open for arbitrarily long provisioning/export tasks.

---

## Operation resource

An operation may include:

- ID;
- target;
- state;
- progress;
- result;
- error;
- started/finished;
- initiating actor.

---

## Cancellation

Define whether async operations can be canceled.

Do not promise cancellation after irreversible work.

---

## Retry guidance

Error responses should tell clients whether retry is appropriate.

Use:

- status/error class;
- retry-after;
- idempotency semantics.

Do not make clients infer transient failure from prose.

---

## Error model

Define stable machine-readable errors.

A generic structure may include:

```json
{
  "code": "RESOURCE_CONFLICT",
  "message": "human-readable summary",
  "request_id": "...",
  "details": {}
}
```

Do not leak internal stack traces.

---

## Error taxonomy

Useful categories may include:

```text
INVALID_ARGUMENT
AUTHENTICATION_REQUIRED
AUTHORIZATION_DENIED
NOT_FOUND
CONFLICT
RATE_LIMITED
QUOTA_EXCEEDED
PRECONDITION_FAILED
DEPENDENCY_UNAVAILABLE
TIMEOUT
INTERNAL_ERROR
```

Keep them stable.

---

## HTTP status mapping

Map domain errors consistently.

Avoid returning `200 OK` with `"success": false`.

---

## Problem details

Standardized error formats may be useful.

Use established conventions where they improve interoperability.

---

## Validation

Validate requests early.

Check:

- schema;
- required fields;
- ranges;
- enums;
- references;
- cross-field constraints.

Do not let malformed input reach deep backend layers.

---

## Semantic validation

Schema validation is not enough.

Business rules remain service/domain concerns.

---

## Unknown fields

Choose intentionally:

- reject;
- ignore;
- preserve.

For forward-compatible client/server APIs, ignoring unknown response fields is
often useful.

Security-sensitive request config may need strictness.

---

## Versioning strategy

Choose a versioning model deliberately.

Possible approaches:

- URI version;
- header/media type;
- protocol package version;
- schema version;
- capability negotiation.

Do not create new major versions for every implementation change.

---

## Breaking change definition

Breaking changes may include:

- removing field;
- changing type;
- changing semantics;
- changing default;
- making optional field required;
- changing error code;
- changing pagination;
- narrowing accepted input.

Document this.

---

## Additive changes

Additive changes are usually safer.

But consider:

- exhaustive enum switches;
- unknown JSON fields;
- client validators.

Not every additive schema change is behaviorally non-breaking.

---

## Enum evolution

Adding enum values may break clients with exhaustive handling.

Document unknown-value behavior.

---

## Field deprecation

Deprecate fields with:

- replacement;
- timeline;
- telemetry on use;
- migration guidance.

Do not silently ignore deprecated fields before removal.

---

## API deprecation

A deprecation should identify:

- affected consumer;
- replacement;
- support end;
- migration path.

---

## Consumer inventory

Track which consumers use versions/endpoints where practical.

This makes deprecation real rather than hopeful.

---

## Compatibility tests

Automate:

- schema diff;
- generated-client compile;
- contract tests;
- behavioral regression.

Do not rely only on manual API review.

---

## Consumer-driven contract tests

Useful when multiple independent consumers exist.

Avoid letting consumer-specific tests freeze accidental implementation details.

---

## Backward compatibility

New servers should serve supported older clients where promised.

---

## Forward compatibility

Older clients should tolerate additive server behavior where expected.

---

## API lifecycle

A useful lifecycle:

```text
experimental
beta
stable
deprecated
retired
```

Define entry/exit criteria.

---

## Experimental APIs

Mark them clearly.

Avoid production-critical consumers depending on unstable contracts without
acknowledged risk.

---

## Documentation

API documentation should include:

- purpose;
- authentication;
- authorization;
- request schema;
- response schema;
- errors;
- pagination;
- limits;
- examples;
- versioning;
- deprecation.

---

## OpenAPI / schema publication

Publish machine-readable schemas.

Ensure published schema matches runtime behavior.

Do not let docs drift from deployed APIs.

---

## Examples

Examples should:

- work;
- use secure auth;
- avoid hard-coded secrets;
- show common error handling.

---

## API catalog

A catalog can improve discoverability.

Metadata may include:

- owner;
- lifecycle;
- audience;
- environment;
- docs;
- schema;
- support.

Do not treat catalog metadata as automatically authoritative.

---

## Developer portal

A portal can present API docs, keys, usage, and onboarding.

The API platform must still be operable without the portal where practical.

---

## API onboarding

A good onboarding flow may include:

```text
discover API
    ->
request access
    ->
obtain credentials
    ->
test in non-production
    ->
observe quotas
    ->
move to production
```

---

## Sandbox environments

Sandboxes can reduce risk.

Define:

- data;
- credentials;
- limits;
- compatibility with production.

Do not let sandbox credentials work in production.

---

## Test credentials

Use non-production credentials.

Never publish real secrets in examples.

---

## SDK lifecycle

If SDKs are generated or maintained:

- version separately if needed;
- align API compatibility;
- deprecate old methods.

Compose with `library-sdk.md`.

---

## CLI lifecycle

If CLI wraps APIs:

- preserve machine output;
- map errors consistently;
- expose endpoint/context clearly.

Compose with `cli.md`.

---

## Routing

Routing policy may depend on:

- host;
- path;
- tenant;
- version;
- region;
- canary.

Keep routing rules explicit.

---

## Route ownership

Every route should map to an owning service/domain.

Avoid orphan routes.

---

## Service discovery

Backends should use stable discovery.

Do not hard-code ephemeral instance addresses.

---

## Load balancing

Choose based on service behavior.

Consider:

- connection reuse;
- streaming;
- locality;
- session affinity.

Avoid sticky sessions unless state model requires them.

---

## Retries

Gateway retries can amplify traffic and duplicate effects.

Retry only when:

- operation is safe/idempotent;
- failure is likely transient;
- timeout budget allows.

Do not retry arbitrary POST writes blindly.

---

## Retry budget

Bound retries across:

```text
client
gateway
service
backend
```

Otherwise each layer may multiply attempts.

---

## Timeout budget

Define end-to-end timeout budget.

A useful pattern:

```text
client timeout
  >
gateway timeout
    >
service downstream timeout
```

with margin.

Do not give every hop the same large timeout.

---

## Circuit breaking

Use circuit breakers where repeated backend failure would cause cascading load.

Do not add them by default if simple timeout/backpressure suffices.

---

## Load shedding

Under overload, shed lower-priority or excess work deliberately.

Do not let queues grow without bound.

---

## Health checks

Gateway health checks should reflect backend readiness.

Do not treat one failed dependency as reason to remove every backend if service
can degrade.

---

## Connection pools

Tune:

- max connections;
- idle timeout;
- per-host limits.

Avoid unbounded connection growth.

---

## HTTP/2 and multiplexing

Understand how protocol multiplexing affects:

- head-of-line blocking;
- connection reuse;
- load balancing.

---

## Streaming

For streaming APIs:

- disable incompatible buffering;
- propagate cancellation;
- handle timeouts carefully.

---

## WebSocket routing

Long-lived sessions affect:

- load balancing;
- deploy drain;
- scale-in;
- timeout configuration.

Plan lifecycle.

---

## DNS

DNS is discovery, not authentication.

---

## TLS

Use TLS across relevant trust boundaries.

Decide termination points intentionally.

Possible locations:

- edge;
- gateway;
- service;
- mesh.

---

## Re-encryption

If TLS terminates at edge and re-encrypts upstream, validate internal peer
identity.

---

## Certificate management

Automate issuance/renewal.

Monitor expiration.

---

## Public certificate policy

Use trusted public certificate authorities for public APIs unless architecture
requires private trust.

---

## Private PKI

For internal APIs, private PKI may be appropriate.

Compose with `identity-platform.md`.

---

## CORS

CORS is a browser policy control.

It is not authentication.

Configure allowed origins narrowly.

---

## CSRF

Cookie-authenticated browser APIs need CSRF protection.

Bearer-token APIs may have different threat models.

---

## Cookies

If APIs use session cookies:

- Secure;
- HttpOnly;
- SameSite;
- narrow domain/path;
- rotation.

---

## Security headers

For browser-facing APIs, set relevant headers.

Do not apply browser-only controls blindly to machine APIs.

---

## WAF

A WAF may provide defense in depth.

Do not treat it as a substitute for application validation and authorization.

---

## Threat protection

Public APIs should consider:

- credential abuse;
- scraping;
- injection;
- payload bombs;
- enumeration;
- SSRF;
- abusive automation.

---

## Input normalization

Canonicalize carefully.

Avoid security decisions before normalization if multiple representations exist.

---

## Header trust

Define which proxy-generated headers are trusted.

Strip conflicting inbound values.

---

## Forwarded headers

Use trusted proxy lists.

Do not trust arbitrary `X-Forwarded-For` from the public internet.

---

## Client IP

Client IP may be useful context.

Do not treat it as strong identity.

---

## Geo/location controls

Use only where requirements justify.

Do not infer identity or authorization solely from geography.

---

## API discovery

If APIs are discoverable dynamically, protect metadata according to sensitivity.

Do not leak internal privileged routes unnecessarily.

---

## Service mesh

A mesh may provide:

- mTLS;
- routing;
- telemetry;
- retries.

Do not let mesh policy replace application-level resource authorization.

---

## East-west gateway

A centralized east-west gateway may improve governance but can add:

- latency;
- bottleneck;
- blast radius.

Use only where requirements justify it.

---

## Direct service-to-service

Direct authenticated service-to-service traffic may be simpler.

Platform can still standardize identity and telemetry.

---

## API management control plane

If API config is centrally managed, define:

- desired state;
- ownership;
- validation;
- rollout;
- rollback;
- audit.

Compose with `control-plane.md`.

---

## Gateway config rollout

Config changes can break all traffic.

Use:

- validation;
- staged rollout;
- versioning;
- rollback.

Do not apply unvalidated global config atomically to every edge.

---

## Route conflict detection

Detect conflicting:

- hosts;
- paths;
- priorities.

Do not let one team accidentally shadow another API.

---

## Policy conflict detection

Global and API-local policies may interact.

Define precedence.

Avoid ambiguous overrides.

---

## Configuration ownership

Separate:

- platform defaults;
- tenant config;
- API-owner config;
- emergency overrides.

---

## Config as code

If gateway config lives in Git:

- review;
- validation;
- reconciliation;
- rollback.

Avoid direct manual config drift.

---

## Dynamic config

If runtime dynamic config is needed, preserve:

- revision;
- audit;
- rollout state.

---

## Multi-region routing

For multi-region APIs, define:

- active/active vs active/passive;
- data locality;
- failover;
- consistency.

Do not hide data residency changes behind generic routing.

---

## Failover

Failover should respect:

- data availability;
- identity;
- tenant policy;
- dependency readiness.

---

## Regional affinity

Some consumers may require region-pinned traffic.

Expose this explicitly where needed.

---

## Edge caching

Caching can improve performance.

Define:

- cache key;
- authorization scope;
- TTL;
- invalidation.

Never cache personalized/tenant data without including security context.

---

## Shared cache isolation

Cache keys must include relevant tenant/user dimensions.

Avoid cross-tenant leakage.

---

## HTTP caching

Use standard cache headers when possible.

Do not cache unsafe methods accidentally.

---

## CDN

For public static/cacheable API content, CDNs may help.

Do not put sensitive API responses into shared CDN cache without strict policy.

---

## Compression

Compression improves bandwidth.

Consider side-channel and decompression-bomb risks where sensitive content is
reflected.

---

## Content negotiation

Support only formats needed.

Do not maintain many equivalent formats without real consumer need.

---

## Uploads

For upload APIs:

- size limits;
- content type validation;
- virus/malware checks where required;
- storage isolation;
- filename/path safety.

Compose with `hostile-input.md`.

---

## Downloads

For downloads:

- authorization;
- content disposition;
- signed URLs if appropriate;
- expiry;
- tenant scope.

Do not issue long-lived public URLs to sensitive objects.

---

## Signed URLs

Signed URLs are delegated capabilities.

Bind:

- object;
- operation;
- expiry.

Keep lifetime short enough for risk.

---

## Webhooks

Webhook APIs need:

- authentication/signature;
- replay protection;
- idempotency;
- retry semantics;
- event identity.

Do not trust source IP alone.

---

## Webhook delivery

If the platform sends webhooks:

- sign payloads;
- retry boundedly;
- expose delivery history;
- avoid leaking secrets in payloads.

---

## Callback URLs

Validate callback destinations.

Protect against SSRF.

---

## API-to-event bridge

Some APIs trigger events.

Make transaction boundary explicit.

Use outbox/transactional patterns where consistency matters.

---

## Event-driven APIs

Compose with `eventing-platform.md` for durable event contracts.

Do not force request/response semantics onto naturally asynchronous domains.

---

## Bulk APIs

Bulk operations should:

- bound item count;
- provide per-item status;
- support async execution if expensive.

Do not create giant synchronous transactions.

---

## Search/export APIs

Large export APIs should often use asynchronous jobs and object retrieval.

Avoid 30-minute HTTP requests.

---

## Data classification

Classify data exposed through APIs.

Use classification to drive:

- authentication strength;
- logging;
- retention;
- audit;
- caching.

---

## Sensitive fields

Avoid returning sensitive fields by default.

Use explicit scopes/projections.

---

## Field-level authorization

For APIs exposing mixed-sensitivity data, field-level authorization may be
required.

Keep complexity justified.

---

## Data minimization

Return only what consumers need.

Avoid giant "everything" endpoints.

---

## Redaction

Redact sensitive values in:

- logs;
- traces;
- errors.

---

## Logging

At the gateway log metadata such as:

- request ID;
- route;
- caller identity class;
- status;
- latency;
- bytes.

Do not log full request/response bodies by default.

---

## Request IDs

Generate or propagate stable request/correlation IDs.

Do not trust externally supplied IDs blindly if they can cause collisions or
log injection.

---

## Tracing

Propagate trace context.

Track:

```text
client
  ->
gateway
  ->
service
  ->
backend
```

Do not place secrets in spans.

---

## Metrics

Useful API metrics:

- request rate;
- error rate;
- latency;
- saturation;
- rate-limit denials;
- auth failures;
- payload size;
- backend failures.

Keep labels low-cardinality.

---

## RED metrics

For request-driven APIs, Rate/Errors/Duration are useful.

Add Saturation where relevant.

---

## Consumer metrics

Per-tenant/API usage may be useful for:

- quotas;
- billing;
- support.

Avoid high-cardinality metrics if logs/events can serve better.

---

## SLOs

Define SLOs by API class and consumer need.

Examples:

- availability;
- latency;
- successful request rate.

Do not invent universal platform SLOs.

---

## Error budget

If SLOs are real, error budgets can inform release/risk decisions.

Do not use them ceremonially.

---

## Audit

Audit high-impact API operations.

Examples:

- admin change;
- privilege grant;
- destructive action;
- tenant configuration;
- credential lifecycle.

---

## Audit identity

Preserve:

- caller;
- delegated actor;
- target;
- action;
- decision;
- result.

---

## Privacy

API telemetry may contain personal data.

Minimize and control retention.

---

## Retention

Define retention separately for:

- access logs;
- audit logs;
- traces;
- request bodies if ever captured.

---

## Developer experience

A good API platform reduces friction in:

- discovery;
- onboarding;
- auth;
- testing;
- debugging;
- migration.

---

## API linting

Lint contracts for:

- naming;
- errors;
- pagination;
- security;
- versioning.

Do not make lint rules stylistic bureaucracy.

---

## Design review

Require review for consequential public/partner API changes.

Avoid central review for trivial internal implementation updates.

---

## Governance

Govern the contract and cross-cutting controls.

Do not centralize every domain decision.

---

## Standards

Platform standards may cover:

- auth;
- errors;
- pagination;
- naming;
- versioning;
- observability.

Keep the standard small and enforceable.

---

## Exceptions

API standards need exception handling.

Record:

- rationale;
- owner;
- scope;
- review/expiry.

---

## API maturity

You may classify APIs by lifecycle/maturity.

Do not create arbitrary numeric quality scores unless they drive useful action.

---

## Consumer support

Document:

- support channel;
- escalation;
- owner;
- incident responsibilities.

---

## Incident response

For API incidents, determine whether failure is:

- edge/gateway;
- identity;
- service;
- backend;
- network;
- consumer misuse.

Use correlation IDs and dependency telemetry.

---

## Dependency failures

Gateway/service should handle downstream failure with:

- finite timeout;
- bounded retry;
- clear errors;
- circuit break/load shed where appropriate.

---

## Degraded behavior

Some APIs can return partial/degraded data.

If so, make it explicit.

Do not silently omit fields without signaling.

---

## Partial success

For batch APIs, return per-item outcomes.

Do not return generic 200 when some items failed unless contract explicitly
models that.

---

## API reliability

Reliability includes:

- idempotency;
- timeout;
- retry;
- backpressure;
- consistency;
- failure semantics.

Not just gateway uptime.

---

## Consistency

Document whether reads are:

- strongly consistent;
- eventually consistent;
- read-your-writes.

Consumers need to know.

---

## Replication lag

If APIs read replicated data, surface or document lag-sensitive behavior.

---

## API state transitions

For stateful resources, define valid transitions.

Avoid hidden side effects triggered by ambiguous field combinations.

---

## State-machine APIs

If domain lifecycle is complex, model states and transitions explicitly.

Do not expose internal DB status codes accidentally.

---

## Webhook/API consistency

If both synchronous API and webhooks represent state, define ordering/finality.

---

## API security review triggers

Stronger review should apply to changes affecting:

- auth;
- authorization;
- admin APIs;
- tenant routing;
- rate limits;
- signed URLs;
- webhook trust;
- data exposure;
- gateway policy;
- protocol parsing.

---

## Threat model

At minimum consider:

- stolen token/API key;
- credential replay;
- route confusion;
- tenant confusion;
- broken object-level authorization;
- payload injection;
- SSRF;
- request smuggling;
- oversized payload;
- cache poisoning;
- cross-tenant cache leak;
- retry amplification;
- gateway compromise;
- backend trust of spoofed identity headers.

---

## BOLA / object-level authorization

Every resource-sensitive API should verify that the caller may access the
specific object.

Do not rely only on route-level auth.

---

## Mass assignment

Do not bind arbitrary request fields directly into privileged domain objects.

Use explicit allowed fields.

---

## Request smuggling

Normalize proxy/gateway behavior.

Keep edge and backend HTTP parsing compatible.

Avoid ambiguous transfer encoding/content length handling.

---

## Header injection

Validate/normalize security-sensitive headers.

Strip hop-by-hop or platform-owned headers.

---

## SSRF

APIs accepting URLs should validate outbound destinations.

Block metadata/internal networks where appropriate.

---

## Open redirects

Validate redirect targets.

Do not allow arbitrary attacker-controlled redirects from trusted domains.

---

## Injection

Use parameterized queries and safe APIs.

API gateways cannot compensate for backend injection vulnerabilities.

---

## Supply chain

API platform components are privileged infrastructure.

Compose with `software-supply-chain.md`.

Protect:

- gateway image;
- policy bundles;
- schemas;
- plugins;
- SDK generators;
- deployment pipeline.

---

## Gateway plugins

Plugins extend privileged traffic handling.

Treat them as trusted code.

Review:

- publisher;
- version;
- permissions;
- update process.

Avoid arbitrary dynamic plugins without governance.

---

## Schema toolchain

Pin code generators and schema compilers where reproducibility matters.

---

## API spec publishing

Published specs should correspond to released API versions.

Do not publish mutable "latest" only if consumers need historical versions.

---

## Deployment

Separate API contract deployment from backend implementation where possible.

A service can deploy frequently without changing public contract.

---

## Gateway rollout

Gateway changes can have broad blast radius.

Use:

- validation;
- canary;
- config versioning;
- rollback.

---

## Route canary

Canary routing should bind to:

- consumer;
- percentage;
- header;
- tenant;
- version.

Document behavior.

---

## Traffic shadowing

Shadow traffic can validate new backends.

Protect sensitive data and ensure shadow requests cannot create side effects.

---

## Blue/green

May simplify major gateway/backend changes.

Ensure state/session compatibility.

---

## Multi-cluster/multi-region

If APIs span clusters/regions:

- route based on health and policy;
- preserve identity;
- consider data locality;
- avoid global failure from one control plane.

---

## Gateway HA

Multiple replicas do not guarantee HA if they share one:

- config store;
- certificate authority;
- DNS dependency;
- stateful rate-limit backend.

Identify real failure domains.

---

## Rate-limit store

Distributed limits may require shared state.

Define behavior if the rate-limit backend fails.

Fail-open or fail-closed according to API risk.

---

## Quota accounting

Quota enforcement should be deterministic enough for policy needs.

Avoid double counting/replay surprises.

---

## API key management

Provide lifecycle:

- issue;
- rotate;
- revoke;
- scope;
- audit.

Do not display secrets repeatedly after creation unless required.

---

## Consumer identity lifecycle

Offboarding a consumer should revoke:

- credentials;
- keys;
- subscriptions;
- privileged access.

---

## API subscription model

If consumers subscribe to APIs:

- record owner;
- scopes;
- environment;
- quota;
- lifecycle.

Avoid indefinite orphan subscriptions.

---

## Developer approval

For sensitive APIs, access request may require approval.

Keep approval scoped to:

- API;
- environment;
- tenant;
- permission.

---

## Change communication

Breaking/deprecating changes need consumer communication.

Use:

- release notes;
- deprecation headers;
- catalog notifications;
- direct owner notifications.

---

## Deprecation headers

Where appropriate, expose standard deprecation/sunset metadata.

Do not rely solely on email.

---

## Usage analytics

Use actual traffic/consumer inventory to determine whether deprecated versions
are still used.

Respect privacy.

---

## Version retirement

Retire only after:

- migration path exists;
- consumers notified;
- critical users migrated;
- exception process handled.

---

## API testing strategy

### Unit tests

Test service/gateway policy logic.

### Contract tests

Validate schema and compatibility.

### Integration tests

Exercise gateway + backend + identity.

### Security tests

Test authn/authz, tenant isolation, abusive input.

### Performance tests

Measure gateway/service overhead and saturation.

### Migration tests

Test old/new API versions.

### End-to-end tests

Exercise real consumer flows.

---

## Schema validation tests

Validate:

- examples;
- required fields;
- enum changes;
- unknown field behavior;
- nullable/optional semantics.

---

## Contract compatibility tests

Detect:

- removed paths;
- removed fields;
- narrowed types;
- required-field additions;
- error-schema breakage.

Review semantic changes separately.

---

## Authorization tests

Test:

- anonymous;
- valid caller;
- wrong scope;
- wrong tenant;
- wrong object;
- admin-only endpoint.

---

## Rate-limit tests

Verify:

- sustained limit;
- burst;
- per-tenant isolation;
- retry-after behavior.

---

## Retry/idempotency tests

Force:

- timeout;
- duplicate request;
- gateway retry.

Confirm no duplicate side effect.

---

## Timeout tests

Verify end-to-end timeout budget.

Avoid zombie backend work continuing after client cancellation where cancellation
can propagate.

---

## Load tests

Test representative:

- request rate;
- payload size;
- concurrency;
- slow backend;
- hot tenant.

---

## Security fuzzing

Consider fuzzing:

- parsers;
- headers;
- content negotiation;
- schema validation;
- webhook verification.

---

## Failure injection

Simulate:

- identity provider down;
- gateway config store down;
- backend unavailable;
- rate-limit store down;
- DNS failure.

Confirm documented degraded behavior.

---

## Acceptance path

A useful reference acceptance path:

```text
consumer authenticates
    ->
requests API
    ->
gateway validates identity
    ->
quota/rate policy allows
    ->
route selected
    ->
service performs resource authorization
    ->
request succeeds
    ->
trace/audit correlated
```

Negative path:

```text
valid token
    +
wrong tenant/resource
    ->
authorization denied
    ->
no backend side effect
```

Compatibility path:

```text
existing supported client
    ->
new platform/service release
    ->
contract still works
```

---

## Documentation

The platform should document:

- API classes;
- ownership;
- contract standards;
- auth;
- authorization;
- errors;
- pagination;
- versioning;
- rate limits;
- quotas;
- deprecation;
- support.

---

## Architecture diagrams

Useful views include:

### Edge

```text
Consumer -> Gateway -> Service -> Backend
```

### Trust

```text
Identity Provider -> Gateway -> Service Authorization
```

### Control

```text
API Config -> Control Plane -> Gateway Fleet
```

### Lifecycle

```text
Design -> Publish -> Adopt -> Deprecate -> Retire
```

---

## Architecture decisions

Use ADRs for consequential choices such as:

- gateway placement;
- protocol;
- auth model;
- API versioning;
- policy enforcement;
- rate-limit architecture;
- internal vs external exposure;
- multi-region routing.

---

## Recommended repository shape

Follow existing conventions first.

A generic API-platform repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── architecture/
│   ├── context.md
│   ├── trust-boundaries.md
│   ├── traffic-flows.md
│   └── decisions/
├── contracts/
│   ├── openapi/
│   ├── protobuf/
│   └── schemas/
├── platform/
│   ├── gateway/
│   ├── auth/
│   ├── policy/
│   ├── routing/
│   ├── rate-limits/
│   └── observability/
├── sdk/
├── cli/
├── tests/
│   ├── contract/
│   ├── integration/
│   ├── security/
│   ├── compatibility/
│   ├── performance/
│   └── e2e/
├── docs/
│   ├── consumers/
│   ├── API-standards/
│   ├── migration/
│   └── runbooks/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

An API-platform repository should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-contract
test-compatibility
test-security
test-integration
test-performance
test-e2e
lint-api
verify
```

These names are illustrative.

Use only commands the repository can actually implement.

---

## Acceptance criteria

An API platform is not complete because traffic successfully passes through a
gateway.

Demonstrate the applicable subset of the following.

### Contract

- API schemas are explicit;
- ownership is defined;
- machine-readable documentation matches runtime;
- error model is stable;
- pagination/idempotency semantics are documented.

### Identity and access

- caller identity is validated;
- audience is enforced;
- resource-level authorization works;
- tenant boundaries are enforced;
- admin APIs have stronger controls.

### Traffic safety

- timeouts are finite;
- retries are bounded and safe;
- payload size limits exist;
- rate limits/quotas work;
- overload behavior is controlled.

### Versioning

- compatibility checks exist;
- breaking changes are detected;
- deprecation/migration process exists;
- consumer usage can be discovered where practical.

### Security

- spoofed identity headers are rejected;
- cross-tenant object access is denied;
- abusive payloads are bounded;
- SSRF/open redirect/header injection risks are addressed where applicable.

### Observability

- request IDs propagate;
- gateway/service latency is visible;
- auth/rate-limit denials are visible;
- traces correlate edge and service;
- sensitive payloads are not logged by default.

### Operations

- gateway config is versioned;
- rollout/rollback exists;
- dependency outages have defined behavior;
- multi-region/failover behavior is understood where applicable.

### Developer experience

- API discovery works;
- onboarding/auth is documented;
- examples are valid;
- SDK/CLI contracts align where provided.

---

## Optional composition

Common combinations:

```text
platform-architecture + api-platform
```

For broad API capability architecture and ownership.

```text
api-platform + identity-platform
```

For trusted caller identity, federation, workload auth, and delegated access.

```text
api-platform + control-plane
```

For centrally managed gateway policy, routes, quotas, and lifecycle.

```text
api-platform + zero-trust-service
```

For internal and external API trust boundaries and resource-aware authorization.

```text
api-platform + software-supply-chain
```

For trusted gateway images, plugins, schemas, and SDK generation.

```text
api-platform + internet-facing
```

For hardened public edge, abuse controls, rate limiting, and attack-surface
reduction.

```text
api-platform + high-assurance
```

For stronger contract verification, fault injection, and change-control
evidence.

---

## Anti-patterns

Avoid:

- treating a gateway product as the API platform;
- putting business workflows in gateway policy;
- route-level auth as the only authorization;
- trusting internal network location;
- accepting tokens without audience validation;
- one API key for an entire organization;
- mutable API contracts with no lifecycle;
- returning 200 with embedded errors;
- no stable machine-readable error model;
- scripts scraping human docs because schema is absent;
- unbounded payloads;
- unbounded retries;
- retries on non-idempotent writes;
- one timeout value at every hop;
- gateway config changes applied globally without staged validation;
- shared cache keys that ignore tenant/user context;
- client IP used as identity;
- CORS treated as authentication;
- WAF treated as application security;
- portal/catalog used as the only source of API truth;
- breaking APIs without consumer inventory or migration;
- API versioning by implementation release number;
- service mesh used as a substitute for domain authorization.

Do not mistake traffic management for API architecture.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. API-platform purpose;
2. API classes and consumer types;
3. ownership model;
4. protocol/schema strategy;
5. authentication model;
6. resource authorization model;
7. tenant isolation model;
8. routing/gateway boundary;
9. rate-limit/quota model;
10. timeout/retry/idempotency model;
11. error model;
12. versioning/deprecation/migration model;
13. observability/audit model;
14. multi-region/failover model where applicable;
15. contract/compatibility tests executed;
16. security/integration/performance tests executed;
17. failure/negative-path tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim an API platform is "secure", "governed", "highly available",
"zero trust", or "developer friendly" merely because it uses an API gateway,
service mesh, OpenAPI, mTLS, rate limiting, or a developer portal.

Describe the contract ownership, identity validation, authorization boundaries,
traffic controls, compatibility process, failure behavior, and actual evidence.

---

## Guiding principle

A good API platform should make contracts boring and failures predictable.

Consumers should know what the API means.

Owners should know what they are responsible for.

Identity should be validated.

Authorization should be resource-aware.

Gateways should handle cross-cutting policy without stealing application
semantics.

Versions should evolve deliberately.

Retries should be safe.

Rate limits should protect shared capacity.

Errors should be machine-usable.

And the platform should always be able to answer:

> Who called which API, under what identity and policy, against which resource,
> through which version and route, with what outcome, and what compatibility
> promise did the consumer rely on?
