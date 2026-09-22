# observability-platform.md

> Recipe for scaffolding or elevating a shared observability platform.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md`, `developer-platform.md`,
> `control-plane.md`, `api-platform.md`, `eventing-platform.md`,
> `kubernetes-workload.md`, `secure-service.md`, and profiles such as
> `zero-trust-service.md`, `software-supply-chain.md`, `high-assurance.md`, or
> `hostile-input.md` where appropriate.
>
> This recipe defines architecture expectations for platforms that collect,
> transport, process, retain, query, visualize, alert on, and govern telemetry
> across multiple workloads, teams, tenants, and environments.
>
> It does not require Prometheus, OpenTelemetry, Grafana, Loki, Elasticsearch,
> Tempo, Jaeger, Datadog, Splunk, a service mesh, a specific cloud, or any
> particular collector or storage engine.

## Purpose

Use this recipe when the platform provides shared observability capabilities.

Typical examples:

- metrics platforms;
- logging platforms;
- tracing platforms;
- telemetry pipelines;
- OpenTelemetry platforms;
- SRE observability foundations;
- centralized monitoring;
- multi-tenant telemetry platforms;
- security telemetry platforms;
- developer-platform observability capabilities;
- AI/ML observability layers.

The goal is not to centralize every log line.

The goal is to provide a coherent observability architecture where teams can
answer:

- What telemetry should this system emit?
- Where is it collected?
- Who owns the signal?
- What metadata identifies the resource?
- How is tenant/team separation enforced?
- How much telemetry is retained?
- What is sampled?
- What is dropped under overload?
- Which signals are suitable for alerting?
- Which data is sensitive?
- How do we correlate metrics, logs, traces, events, and audit?
- What happens when the observability backend is unavailable?
- How do we know telemetry itself is healthy?

---

## Core principle

Telemetry is production data, not exhaust.

A useful model is:

```text
Workloads / Infrastructure
          |
          v
    Instrumentation
          |
          v
 Collection / Ingestion
          |
          v
 Processing / Routing
          |
          +--> metrics store
          +--> log store
          +--> trace store
          +--> event store
          |
          v
 Query / Alert / SLO / Investigation
          |
          v
 Humans / Automation
```

The platform should preserve enough context to explain system behavior without
turning every byte emitted by every workload into permanent centralized data.

Observability is not the same as:

- monitoring;
- logging;
- tracing;
- audit;
- analytics.

Those may be components of the platform.

---

## Architectural invariants

An observability platform SHOULD satisfy these invariants unless there is a
documented reason not to.

### 1. Telemetry has explicit ownership

Every important signal should have an owner.

Do not collect metrics or logs no team understands or uses.

### 2. Signals have defined semantics

A metric name or log field should mean something stable.

Do not let the same metric name represent different semantics across teams.

### 3. Resource identity is consistent

Telemetry should identify the emitting resource consistently across:

- metrics;
- logs;
- traces;
- events.

Avoid incompatible naming per signal type.

### 4. Cardinality is bounded

Labels/tags/attributes must not grow without control.

Do not put unbounded user IDs, request IDs, trace IDs, raw URLs, or arbitrary
payload values into metric dimensions.

### 5. Sensitive data is minimized

Telemetry frequently leaks:

- credentials;
- personal data;
- tokens;
- payloads;
- internal topology.

Treat telemetry as sensitive data by default until classified.

### 6. Collection failure does not normally crash the workload

Applications should continue functioning when telemetry backends are degraded
unless telemetry is itself a required business control.

### 7. Telemetry loss is visible

Dropping telemetry may be acceptable under pressure.

Silent loss is not.

### 8. Alerting is tied to actionable conditions

Alerts should indicate conditions requiring human or automated action.

Do not page on every anomaly.

### 9. SLOs measure consumer-visible behavior

Platform and application SLOs should reflect outcomes, not internal implementation
metrics alone.

### 10. Retention is intentional

Retention should reflect:

- operational need;
- security;
- legal/privacy requirements;
- cost.

Do not retain everything forever.

### 11. Query isolation is enforced

One tenant/team should not read another tenant's sensitive telemetry without
authorization.

### 12. Observability does not become a runtime dependency by accident

Instrumentation and collectors should fail safely.

---

## Signal taxonomy

Define which signals the platform supports.

Common classes:

```text
metrics
logs
traces
events
profiles
synthetic probes
audit
```

Do not force all use cases into one telemetry type.

---

## Metrics

Metrics are useful for:

- rates;
- counters;
- gauges;
- distributions;
- saturation;
- SLO calculation.

They are poor fits for high-cardinality per-request detail.

---

## Logs

Logs are useful for:

- discrete diagnostic events;
- state transitions;
- error context;
- operator-readable narratives.

Do not use logs as a substitute for stable metrics where aggregation matters.

---

## Traces

Traces are useful for:

- distributed request paths;
- latency decomposition;
- dependency analysis;
- causal debugging.

Do not trace every operation at full fidelity without a sampling/cost model.

---

## Events

Operational events may represent:

- deployments;
- failovers;
- configuration changes;
- scaling;
- incidents.

They help correlate system change with telemetry.

---

## Profiles

Continuous or sampled profiles may help diagnose:

- CPU;
- memory;
- allocation;
- lock contention.

Use only where platform/runtime support and risk justify it.

---

## Audit

Audit is not ordinary logging.

Audit should capture security- and governance-relevant actions with stronger
integrity, access, and retention requirements.

Do not mix authoritative audit records into disposable debug logs and call them
equivalent.

---

## Resource model

Define a common resource identity.

Potential fields:

```text
service
component
environment
region
cluster
namespace
tenant
instance
version
```

Keep dimensions stable and bounded.

---

## Resource naming

Prefer names derived from authoritative platform metadata.

Do not rely on free-form application strings for core resource identity.

---

## Service identity

A service should have a stable identity across deployments.

Do not use pod/container IDs as the only service identifier.

---

## Version identity

Include artifact/release version where useful.

This helps correlate regressions with releases.

---

## Environment identity

Distinguish:

- development;
- test;
- staging;
- production.

Do not merge them into one telemetry namespace without strong filtering.

---

## Tenant identity

For multi-tenant observability, derive tenant from trusted platform context.

Do not trust a caller-supplied telemetry attribute as the only isolation
boundary.

---

## Instrumentation strategy

Prefer standardized instrumentation where practical.

Potential approaches:

- language SDK;
- auto-instrumentation;
- framework middleware;
- sidecar/agent;
- gateway instrumentation.

Do not require application teams to handcraft every signal.

---

## Native versus auto instrumentation

Auto-instrumentation can reduce adoption cost.

Native instrumentation often provides better domain semantics.

Use both where appropriate.

---

## Open standards

Use open telemetry protocols/formats where they improve interoperability and
portability.

Do not choose standards as an end in themselves.

---

## Instrumentation libraries

Keep instrumentation libraries:

- lightweight;
- versioned;
- safe under failure;
- minimally invasive.

Avoid libraries that take over application logging globally.

---

## Semantic conventions

Define stable semantic conventions.

Examples:

- service name;
- HTTP route;
- RPC method;
- database system;
- messaging destination.

Use ecosystem conventions where available.

---

## Domain metrics

Application teams should own domain-specific metrics.

The platform may provide:

- runtime;
- infrastructure;
- framework;
- request-level signals.

Do not make the platform team invent business metrics for applications.

---

## Default instrumentation

Golden paths may provide:

- request rate;
- error rate;
- latency;
- runtime metrics;
- basic traces;
- standard log fields.

This is a baseline, not complete observability.

---

## Correlation

Support correlation across telemetry.

Common correlation keys:

- trace ID;
- span ID;
- request ID;
- operation ID;
- deployment version.

Do not put high-cardinality IDs into metric labels.

---

## Request IDs

Request IDs are useful for logs and support workflows.

Do not use external request IDs without validation if they can create log
injection or collisions.

---

## Trace context

Propagate trace context across:

- HTTP;
- RPC;
- messaging;
- jobs.

Do not trust arbitrary incoming trace metadata as authorization context.

---

## Baggage

Trace baggage can propagate useful context.

Use sparingly.

Do not put secrets or large user-provided values into baggage.

---

## Collection architecture

Common models include:

```text
application -> agent -> gateway -> backend
```

or:

```text
application -> collector -> backend
```

or direct export for simple environments.

Choose according to:

- scale;
- trust;
- network topology;
- control;
- cost.

---

## Agent-based collection

Node/host agents can collect:

- logs;
- host metrics;
- container metadata.

They add privileged runtime footprint.

Use least privilege.

---

## Sidecars

Sidecars can isolate collection per workload but add:

- resource cost;
- lifecycle complexity;
- failure modes.

Do not require sidecars if node/daemon collectors are sufficient.

---

## Gateway collectors

Gateway collectors can centralize:

- sampling;
- transformation;
- routing;
- auth;
- export.

Design for horizontal scaling and backpressure.

---

## Direct export

Direct application-to-backend export may be appropriate for small systems.

Understand:

- credential distribution;
- backend coupling;
- failure impact.

---

## Push versus pull

Metrics may use:

- pull;
- push;
- hybrid.

Choose based on runtime and network model.

Do not insist on one collection model for every signal.

---

## Collector identity

Collectors are privileged aggregation points.

Authenticate them and scope their backend write permissions.

---

## Telemetry authentication

Use workload/platform identity where practical.

Do not distribute one global ingest token to every workload.

---

## Ingestion authorization

Constrain telemetry writes by:

- tenant;
- service;
- environment;
- signal type.

Prevent one workload from impersonating another service's telemetry.

---

## Ingestion endpoints

Separate public/untrusted ingestion from internal trusted ingestion where risk
requires.

---

## Schema validation

Validate telemetry shape.

Reject or quarantine malformed data.

Do not let arbitrary attributes create uncontrolled indexing/cardinality.

---

## Normalization

Normalize:

- resource metadata;
- field names;
- timestamps;
- severity;
- trace context.

Do not rewrite domain semantics unexpectedly.

---

## Enrichment

Collectors may enrich telemetry with trusted metadata:

- service owner;
- cluster;
- region;
- environment.

Avoid overwriting application fields without clear precedence.

---

## Attribute precedence

Define which source wins when both platform and workload set the same attribute.

Security-sensitive dimensions should come from trusted platform metadata.

---

## Filtering

Drop telemetry intentionally.

Possible filters:

- debug logs in production;
- known noisy endpoints;
- health checks;
- high-volume low-value spans.

Document filters.

---

## Redaction

Redact sensitive fields before long-term storage where possible.

Examples:

- authorization headers;
- passwords;
- tokens;
- cookies;
- personal identifiers.

Do not rely only on downstream query-time masking.

---

## Transformation

Telemetry processors may:

- rename fields;
- aggregate;
- sample;
- route;
- redact.

Keep transformations versioned and testable.

---

## Routing

Route telemetry based on:

- tenant;
- environment;
- classification;
- signal type;
- retention class.

Do not let user-controlled labels route data into privileged stores without
validation.

---

## Multi-backend routing

Some platforms may use different backends for:

- hot operational logs;
- security audit;
- archive;
- regional residency.

Document authority and duplication.

---

## Backpressure

Collectors must handle backend slowdown.

Options:

- bounded memory queue;
- disk buffer;
- sample/drop;
- reject upstream.

Avoid unbounded queues.

---

## Local buffering

If collectors spool to disk:

- bound size;
- protect permissions;
- encrypt where needed;
- handle disk pressure.

Do not let observability fill application disks.

---

## Drop policy

Under pressure, define what may be dropped first.

Example:

```text
debug logs
    ->
high-volume low-value traces
    ->
noncritical metrics
```

Critical audit/security data may require stronger durability.

---

## Telemetry loss metrics

Track dropped:

- spans;
- log records;
- metric points;
- events.

Silent telemetry loss is an observability failure.

---

## Retry

Telemetry exporters may retry transient failures.

Use bounded backoff.

Do not let retry storms harm workloads.

---

## Fail-open telemetry

Ordinary application telemetry should normally fail open with respect to
business processing.

Do not make user requests fail because the metrics backend is down.

---

## Fail-closed telemetry

Some compliance/security audit controls may require fail-closed behavior.

If so, isolate and document that requirement explicitly.

---

## Metrics architecture

Metrics storage should support:

- aggregation;
- time windows;
- alerting;
- SLO calculation.

Control cardinality from ingestion onward.

---

## Metric naming

Use stable naming conventions.

Avoid product-specific prefixes leaking into consumer contracts unnecessarily.

---

## Counters

Counters should be monotonic where semantics require.

Reset behavior should be understood.

---

## Gauges

Use gauges for current state.

Do not use gauges for cumulative totals that need rate calculation.

---

## Histograms

Use histograms/distributions for latency and size.

Choose buckets or native histogram strategy according to expected ranges and
backend capabilities.

---

## Percentiles

Do not average percentiles across instances.

Prefer aggregatable distributions where global percentiles matter.

---

## Metric labels

Labels should be:

- bounded;
- low-cardinality;
- operationally useful.

Avoid:

- user ID;
- request ID;
- full URL;
- raw exception message.

---

## Route labels

Use normalized route templates.

Prefer:

```text
/orders/{id}
```

over:

```text
/orders/12345
```

---

## Error labels

Use stable error category.

Do not use raw error strings as metric labels.

---

## Log architecture

Logs should be structured where possible.

Useful fields:

- timestamp;
- severity;
- service;
- environment;
- trace ID;
- operation ID;
- error category;
- message.

---

## Log severity

Define consistent severity semantics.

Avoid every recoverable warning becoming ERROR.

---

## Structured logging

Prefer machine-parsable structure.

Human-readable message remains useful.

---

## Log ingestion

Support:

- stdout/stderr;
- files where necessary;
- event APIs.

Avoid requiring applications to write directly to centralized log storage APIs
unless needed.

---

## Log parsing

Prefer structured logs over regex parsing.

If parsing legacy logs, treat parsers as versioned transformations.

---

## Multiline logs

Handle stack traces safely.

Do not let malformed multiline data merge unrelated records.

---

## Sensitive logging

Never log:

- passwords;
- bearer tokens;
- private keys;
- secret material.

Be cautious with:

- request bodies;
- headers;
- SQL;
- prompts/model inputs;
- personal data.

---

## Debug logging

Debug logs should be controlled.

Do not enable globally in production indefinitely.

---

## Dynamic log level

If runtime log-level changes are supported:

- authorize;
- audit;
- expire where appropriate.

Do not let unauthenticated users enable verbose logging.

---

## Trace architecture

Tracing should represent causal work.

A trace may cross:

- API gateway;
- service;
- queue;
- worker;
- database;
- external API.

---

## Span naming

Use stable low-cardinality span names.

Avoid raw URLs or user input.

---

## Span attributes

Include useful context.

Do not attach full payloads or secrets.

---

## Sampling

Sampling is a product decision.

Potential strategies:

- head sampling;
- tail sampling;
- probabilistic;
- rate limiting;
- error-biased;
- latency-biased.

Document what is lost.

---

## Head sampling

Simple and cheap.

May discard interesting failures before they are known.

---

## Tail sampling

Can preserve:

- errors;
- slow traces;
- rare paths.

Requires buffering and more infrastructure.

---

## Adaptive sampling

Useful at scale.

Ensure it does not systematically hide rare but important behavior.

---

## Sampling and SLOs

Do not compute exact request-rate SLOs from sampled traces alone unless the
statistics support it.

Use unsampled counters for authoritative SLI calculations where appropriate.

---

## Trace retention

Full traces may need shorter retention than aggregate metrics.

Use tiered retention.

---

## Exemplars

Link metrics to traces where supported.

This can speed investigation.

---

## Synthetic monitoring

Synthetic checks can validate user-visible paths.

Examples:

- login;
- API request;
- deployment workflow;
- DNS/TLS.

Do not let synthetic success substitute for real-user telemetry.

---

## Black-box monitoring

Use black-box probes for external behavior.

Useful for:

- endpoint availability;
- TLS;
- DNS;
- latency.

---

## White-box monitoring

Use internal telemetry for:

- saturation;
- queue depth;
- dependency health.

Both are useful.

---

## Real user monitoring

For browser/mobile platforms, real-user monitoring may capture client
experience.

Handle privacy carefully.

---

## Profiling

Profiling should be opt-in or controlled according to cost/sensitivity.

Avoid collecting heap objects containing secrets.

---

## Telemetry metadata plane

A shared metadata service may map:

- service;
- owner;
- environment;
- tier;
- repository.

Prefer authoritative sources.

Do not require every application to duplicate metadata manually.

---

## Service catalog integration

A catalog can enrich telemetry with ownership.

Do not rely on stale catalog data as security authorization.

---

## Ownership lookup

Alert routing depends on correct ownership.

Have fallback ownership for unknown services.

---

## Alerting architecture

Alerts should represent:

- actionable failure;
- imminent risk;
- SLO burn;
- capacity exhaustion.

Avoid alerting on every individual error.

---

## Page versus ticket

Classify alerts.

Example:

```text
PAGE
  immediate human action required

TICKET
  action needed but not urgent

INFO
  context only
```

---

## Symptom versus cause

Prefer symptom-based paging.

Example:

```text
high user-visible error rate
```

may be more useful than:

```text
CPU 81%
```

unless CPU directly predicts imminent failure.

---

## Multi-window burn alerts

For SLO-driven systems, multi-window burn-rate alerts can balance speed and
noise.

Use where SLO maturity justifies it.

---

## Alert ownership

Every alert should have:

- owner;
- severity;
- runbook;
- rationale.

Do not create orphan alerts.

---

## Alert deduplication

Deduplicate repeated manifestations of the same incident where practical.

Do not suppress independent failures accidentally.

---

## Alert grouping

Group by meaningful failure domain.

Avoid one page per pod when the service is failing globally.

---

## Alert suppression

Maintenance/deployment suppression may reduce noise.

Keep suppression scoped and time-bounded.

---

## Alert testing

Test alert rules.

Do not discover syntax/logic errors during incidents.

---

## Runbooks

High-value alerts should link to a runbook.

A runbook should include:

- what the alert means;
- likely causes;
- diagnostics;
- safe remediation;
- escalation.

---

## SLO architecture

An SLO should describe a consumer-relevant capability.

Example:

```text
99.9% of valid API requests succeed within 500 ms over 30 days
```

Use actual requirements.

Do not invent numbers for architecture completeness.

---

## SLI

Define exactly how the signal is calculated.

Possible SLIs:

- request success;
- latency;
- freshness;
- availability;
- correctness.

---

## SLO ownership

Application teams generally own application SLOs.

The observability platform owns telemetry/SLO tooling and its own platform SLOs.

---

## Error budget

If using error budgets, define:

- window;
- calculation;
- burn policy;
- exclusions.

Avoid gaming exclusions.

---

## Availability measurement

Measure from the consumer perspective.

Internal process uptime is not sufficient.

---

## Latency measurement

Use suitable percentiles/distributions.

Do not rely only on averages.

---

## Freshness SLOs

For data/event systems, freshness may matter more than request latency.

---

## Observability platform SLOs

Possible platform SLOs include:

- telemetry ingestion availability;
- query availability;
- alert delivery delay;
- telemetry loss;
- trace ingestion latency.

---

## Query architecture

Users should be able to query telemetry by stable resource dimensions.

Avoid requiring deep backend-specific knowledge for common operations.

---

## Query isolation

Enforce tenant/team access.

Do not assume obscurity of index names provides isolation.

---

## Query quotas

Expensive queries can destabilize shared backends.

Use:

- query timeout;
- concurrency limits;
- scan limits;
- rate limits.

---

## Query cancellation

Support cancellation.

Do not let abandoned dashboard queries continue consuming large resources.

---

## Dashboard architecture

Dashboards should answer operational questions.

Avoid wall-of-graphs dashboards with no purpose.

---

## Golden dashboards

Golden signals often include:

- latency;
- traffic;
- errors;
- saturation.

Adapt to domain.

---

## Service dashboards

Standard service dashboards may be auto-generated.

Application teams should add domain-specific panels.

---

## Dashboard ownership

Every production dashboard should have owner/context.

Retire obsolete dashboards.

---

## Dashboard as code

Version dashboards where practical.

Avoid manual drift for critical views.

---

## Search experience

Log/trace search should support:

- service;
- environment;
- time;
- correlation ID;
- severity.

---

## Cross-signal navigation

Enable workflows such as:

```text
alert
  ->
metric
  ->
trace exemplar
  ->
logs
  ->
deployment event
```

This is a major observability platform capability.

---

## Deployment correlation

Record deployments/releases as events or annotations.

This helps identify regressions.

---

## Change events

Useful operational changes include:

- config update;
- feature flag;
- scaling;
- policy change;
- failover.

---

## Audit versus observability

Keep audit semantics distinct.

Audit typically requires stronger:

- integrity;
- retention;
- access control;
- completeness.

Do not delete audit merely because log retention expired.

---

## Security telemetry

Security teams may need:

- auth failures;
- privilege changes;
- policy denials;
- suspicious access.

Do not assume normal application logs are adequate audit evidence.

---

## SIEM integration

If telemetry flows to a SIEM/security platform:

- classify;
- route;
- minimize duplicates;
- preserve identity/context.

Do not send every debug log to expensive security storage by default.

---

## Data classification

Classify telemetry by sensitivity.

Examples:

```text
public
internal
confidential
restricted
security-audit
```

Use classification to drive access and retention.

---

## Personal data

Logs/traces may contain personal data.

Minimize and mask.

Define retention.

---

## Secret scanning

Consider automated detection for accidentally logged secrets.

Treat detection as backstop, not permission to log carelessly.

---

## Prompt/AI telemetry

AI systems may log:

- prompts;
- model responses;
- retrieved context;
- tool arguments.

These can be highly sensitive.

Apply explicit policy before collection.

---

## Redaction policy

Define platform-level redaction for known sensitive fields.

Applications still own domain-specific redaction.

---

## Data residency

Telemetry may cross regions.

Define routing/retention by residency requirements.

---

## Regional ingestion

Local ingestion can reduce latency/residency risk.

Centralized query may require controlled aggregation.

---

## Cross-region query

If supported, define:

- authorization;
- data movement;
- latency;
- fallback.

---

## Encryption

Use transport encryption.

Use encryption at rest according to platform/security requirements.

---

## Key management

If tenants require separate keys, define lifecycle and operational cost.

Do not promise tenant-key isolation without verifying backend behavior.

---

## Retention

Define retention classes.

Example:

```text
debug logs       -> short
operational logs -> medium
metrics          -> longer aggregate
traces           -> shorter sampled
audit            -> policy-defined
```

Do not use one retention policy everywhere.

---

## Tiered storage

Older telemetry may move to lower-cost storage.

Define:

- query latency;
- retrieval process;
- retention.

---

## Downsampling

Metrics may be downsampled over time.

Document loss of granularity.

---

## Aggregation

Pre-aggregation can reduce cost.

Do not destroy dimensions needed for incident response without analysis.

---

## Deletion

Support deletion where required by privacy or contractual obligations.

Know how deletion propagates to:

- hot store;
- archive;
- replicas;
- indexes;
- caches.

---

## Legal hold

Where applicable, legal hold may override normal retention.

Keep scope explicit.

---

## Cost architecture

Observability cost often grows with:

- log bytes;
- metric cardinality;
- trace volume;
- retention;
- query scan.

Expose cost drivers.

---

## Cost attribution

Where useful, attribute telemetry cost to:

- team;
- service;
- environment;
- tenant.

Avoid chargeback complexity if nobody acts on it.

---

## Budgets

Set budgets or quotas for:

- log volume;
- metric series;
- trace rate;
- query usage.

Avoid surprise bills.

---

## High-cardinality controls

Detect and reject/sanitize explosive dimensions.

Do not let one deployment create millions of metric series unnoticed.

---

## Cardinality alerts

Alert on sudden series growth.

---

## Sampling budgets

Set trace/log sampling by service criticality and volume.

Avoid one global percentage.

---

## Debug burst mode

For incidents, temporarily increase telemetry.

Time-bound the increase.

Do not leave debug sampling enabled forever.

---

## Telemetry quality

Quality includes:

- completeness;
- correctness;
- freshness;
- consistency;
- attribution.

A metric that exists but lies is worse than no metric.

---

## Telemetry contracts

Critical telemetry may have contracts.

Examples:

- service must emit request count;
- request errors use standardized field;
- traces propagate context.

Keep required contracts minimal and testable.

---

## Contract testing

Validate instrumentation behavior in CI/integration environments where useful.

---

## Missing telemetry

Detect services with no expected metrics/logs/traces.

Do not assume silence means health.

---

## Timestamp quality

Clock skew affects correlation.

Ensure hosts/workloads have time synchronization.

---

## Late telemetry

Collectors/backends may receive delayed data.

Define acceptable freshness.

---

## Duplicate telemetry

Retries may duplicate logs/spans.

Understand backend deduplication.

Do not assume every telemetry pipeline is exactly once.

---

## Ordering

Logs/events may arrive out of order.

Queries should sort by timestamps with appropriate caveats.

---

## Telemetry provenance

Know:

- source;
- collector;
- transformation;
- destination.

This matters when debugging data discrepancies.

---

## Telemetry pipeline observability

Observe the observability pipeline itself.

Track:

- collector health;
- queue depth;
- dropped data;
- export errors;
- backend ingest delay.

---

## Meta-monitoring

Use an independent or sufficiently decoupled signal to detect platform-wide
observability failure.

Avoid relying solely on the failed backend to report its own outage.

---

## Synthetic telemetry canaries

Generate known telemetry periodically.

Verify it appears end to end.

This detects silent ingestion gaps.

---

## Alert-delivery verification

Test that alerts actually reach the expected channel.

Do not assume rule evaluation equals notification delivery.

---

## Notification routing

Route alerts by:

- service owner;
- severity;
- environment.

Use authoritative ownership metadata.

---

## Notification systems

Treat notification systems as dependencies.

Define behavior if:

- chat;
- pager;
- email

is unavailable.

---

## Incident integration

Observability can integrate with incident management.

Keep incident workflow separate from telemetry storage.

---

## Maintenance windows

Support scoped maintenance windows.

Avoid global silences.

---

## Silence ownership

Every silence should have:

- owner;
- reason;
- scope;
- expiry.

No permanent unexplained silences.

---

## Access control

Define roles such as:

- viewer;
- service owner;
- platform operator;
- auditor;
- security analyst.

Use least privilege.

---

## Read access

Telemetry may reveal sensitive production information.

Do not grant every developer unrestricted organization-wide access by default.

---

## Write access

Restrict:

- dashboards;
- alerts;
- recording rules;
- ingestion config;
- redaction policy.

Avoid unreviewed global rule changes.

---

## Administrative access

Separate platform administration from ordinary telemetry query access.

---

## Multi-tenancy

Tenant isolation should cover:

- ingestion;
- storage;
- query;
- alerting;
- dashboards;
- retention;
- metadata.

Do not rely solely on naming prefixes.

---

## Shared backend

If tenants share storage:

- enforce query filters server-side;
- protect metadata;
- partition quotas.

---

## Dedicated backend

Use dedicated storage only when isolation/compliance/scale requires.

Do not multiply operational burden without need.

---

## Noisy tenants

Protect against one tenant causing:

- ingest saturation;
- series explosion;
- expensive queries;
- retention pressure.

---

## Query fairness

Use concurrency/rate controls.

Do not let one broad regex query starve incident response.

---

## Platform admin blast radius

Observability administrators may have access to large amounts of sensitive data.

Use:

- MFA;
- JIT;
- audit;
- separate admin roles.

---

## Audit of observability admin

Audit:

- retention changes;
- redaction changes;
- tenant access;
- alert-rule changes;
- data exports.

---

## Data export

Exporting telemetry can bypass platform access controls.

Treat export as privileged.

---

## API access

If exposing observability APIs:

- authenticate;
- authorize;
- rate-limit;
- version.

Compose with `api-platform.md`.

---

## SDKs

Provide SDKs only when they materially improve instrumentation.

Compose with `library-sdk.md`.

---

## CLI

A CLI may help with:

- query;
- tail;
- trace lookup;
- dashboard validation;
- alert testing.

Compose with `cli.md`.

---

## Developer platform integration

A developer platform should make basic observability available by default.

Examples:

- service dashboard;
- logs;
- request metrics;
- traces;
- deployment annotations.

Do not require teams to file tickets for standard access.

---

## Golden path observability

A golden-path service should ship with:

- standard resource metadata;
- default telemetry;
- dashboard;
- basic alerts where appropriate.

Avoid generating dozens of meaningless default alerts.

---

## Service onboarding

Onboarding should establish:

- service identity;
- owner;
- environment;
- access;
- default telemetry route.

---

## Offboarding

When a service is retired:

- remove alerts;
- retire dashboards;
- expire access;
- retain telemetry according to policy.

Do not leave stale operational noise indefinitely.

---

## Kubernetes integration

Kubernetes telemetry may include:

- node;
- pod;
- workload;
- control-plane metrics;
- events.

Use `kubernetes-workload.md` for application runtime expectations.

Do not expose raw pod-centric views as the only application observability
experience.

---

## Infrastructure telemetry

Infrastructure teams may need lower-level signals.

Keep consumer-facing dashboards at service/capability level where possible.

---

## Service mesh telemetry

Mesh telemetry can provide consistent network signals.

Do not assume it captures business errors or application semantics.

---

## API platform integration

Correlate:

- edge request;
- gateway;
- service;
- backend.

Use shared request/trace context.

---

## Eventing integration

For asynchronous systems, observability should track:

- publish;
- lag;
- consume;
- retry;
- DLQ;
- replay.

Compose with `eventing-platform.md`.

---

## Data platform integration

For pipelines, observe:

- freshness;
- completeness;
- quality;
- processing lag;
- lineage operation.

Runtime success alone is not enough.

---

## AI platform integration

For AI systems, observe:

- model;
- prompt/version;
- latency;
- token/cost;
- refusal/error;
- tool calls;
- retrieval metadata.

Avoid logging sensitive model inputs/outputs without explicit policy.

---

## Supply chain

Observability platform components are privileged data systems.

Compose with `software-supply-chain.md`.

Protect:

- collectors;
- agents;
- plugins;
- dashboards/rules as code;
- backend images;
- query extensions.

---

## Collector plugins

Plugins/parsers can process hostile telemetry.

Treat them as code execution surface.

Review and pin versions.

---

## Parser security

Logs and telemetry are attacker-influenced.

Protect parsers against:

- malformed data;
- oversized fields;
- regex DoS;
- decompression bombs.

Compose with `hostile-input.md`.

---

## Query-language security

Observability query languages can be powerful.

Restrict dangerous administrative/export functions.

Do not concatenate untrusted input into queries.

---

## Dashboard injection

Treat labels/log values as untrusted display content.

Avoid unsafe HTML/script rendering.

---

## Log injection

Escape control characters where relevant.

Do not let attacker-controlled newlines forge structured audit lines.

---

## Threat model

At minimum consider:

- secret leakage into logs;
- cross-tenant query;
- telemetry spoofing;
- cardinality attack;
- ingest flood;
- expensive-query DoS;
- malicious parser input;
- compromised collector;
- audit deletion;
- sensitive trace payloads;
- telemetry pipeline outage hiding incidents.

---

## Telemetry spoofing

A compromised workload should not be able to impersonate arbitrary services or
tenants easily.

Bind trusted resource attributes at collector/platform layer.

---

## Cardinality attack

A malicious or buggy workload can generate unbounded series.

Enforce quotas and attribute limits.

---

## Ingestion flood

Rate-limit or isolate abusive producers.

Do not let observability become a cluster-wide denial-of-service vector.

---

## Query DoS

Bound:

- time range;
- regex complexity;
- concurrency;
- scanned bytes.

---

## Security review triggers

Require stronger review for changes to:

- tenant isolation;
- retention;
- audit;
- redaction;
- ingestion identity;
- query access;
- alert delivery;
- global sampling;
- collector privilege.

---

## Change management

Observability changes can hide failures globally.

Version and review:

- alert rules;
- recording rules;
- routing;
- redaction;
- retention;
- sampling.

---

## Dashboards as code

Version critical dashboards.

Review changes where operationally important.

---

## Alert rules as code

Prefer version-controlled rules.

Test before rollout.

---

## Recording rules

Recording rules/precomputed aggregates may improve query efficiency.

Document semantic meaning.

Do not create aggregates nobody owns.

---

## Global config rollout

Use staged rollout for changes with wide blast radius.

Examples:

- collector config;
- parsing;
- routing;
- redaction.

---

## Canary collectors

Validate new collector/transformation versions on a subset before global
deployment where risk justifies it.

---

## Rollback

Keep known-good:

- configs;
- parsers;
- collectors;
- rules.

Do not make emergency rollback depend on the failing observability UI alone.

---

## Compatibility

Telemetry schemas evolve.

Keep queries/dashboards resilient to additive fields.

Avoid renaming core resource dimensions without migration.

---

## Deprecation

Deprecate:

- metric names;
- log fields;
- dashboards;
- query APIs

with transition periods where consumers depend on them.

---

## Metric migration

For metric rename, consider dual emission temporarily.

Avoid permanent duplication.

---

## Dashboard migration

Update dashboards/alerts before removing signals.

---

## Query migration

Track critical consumers of saved queries/API integrations.

---

## Capacity planning

Model:

- ingest bytes/sec;
- metric series;
- trace spans/sec;
- log events/sec;
- retention;
- query concurrency;
- storage growth.

Do not size only by average ingestion.

---

## Peak behavior

Plan for:

- incident log storms;
- deployment spikes;
- traffic bursts;
- security events.

---

## Storage forecasting

Retention * ingest rate * compression/replication drives storage.

Use measured assumptions.

---

## Hot/warm/cold tiers

Use tiering where cost/query patterns justify it.

Do not overcomplicate small environments.

---

## Index strategy

Index only fields needed for query.

Over-indexing increases cost.

Under-indexing harms incident response.

---

## Compression

Use backend-appropriate compression.

Understand CPU/query tradeoffs.

---

## Query acceleration

Pre-aggregation, indexes, caching, or columnar storage may help.

Do not hide stale cached results as current.

---

## Backend sharding

Shard by dimensions that support scale and isolation.

Plan rebalancing.

---

## Multi-region

For multi-region observability define:

- local ingestion;
- replication;
- central query;
- residency;
- failover.

---

## Regional independence

A region should ideally retain enough local observability to diagnose regional
failures.

Do not make regional incident response depend entirely on another failed region.

---

## Disaster recovery

Recover:

- critical dashboards/rules;
- tenant config;
- retention config;
- auth mappings;
- audit configuration.

Historical telemetry may or may not require full DR depending on requirements.

---

## Backup

Back up configuration that cannot be rebuilt.

Avoid backing up ephemeral caches.

---

## Rebuildability

Prefer infrastructure/config as code for:

- dashboards;
- alerts;
- collectors;
- routing.

---

## Platform health dashboard

The observability platform needs its own health view.

Include:

- ingest;
- drop;
- storage;
- query;
- alert delivery;
- collector errors.

---

## Testing strategy

### Unit tests

Test:

- processors;
- parsers;
- redaction;
- routing;
- alert logic.

### Contract tests

Validate resource/attribute conventions.

### Integration tests

Emit telemetry through real collectors/backends.

### Security tests

Test tenant isolation and secret redaction.

### Load tests

Test peak ingest and query pressure.

### Failure tests

Test backend/collector outage.

### Alert tests

Verify rule evaluation and delivery.

### End-to-end tests

Emit known telemetry and verify it is queryable and alertable.

---

## Synthetic canary test

A useful E2E check:

```text
emit known metric/log/span
    ->
collector receives
    ->
processor enriches
    ->
backend stores
    ->
query finds
    ->
alert rule evaluates if applicable
```

---

## Telemetry loss tests

Force backend unavailability.

Verify:

- bounded buffers;
- drop metrics;
- workload unaffected;
- recovery behavior.

---

## Cardinality tests

Inject high-cardinality labels.

Confirm limits/sanitization.

---

## Redaction tests

Inject:

- fake token;
- password;
- sensitive field.

Verify it does not reach long-term storage where policy requires.

---

## Tenant isolation tests

Attempt cross-tenant:

- query;
- dashboard access;
- alert access;
- log export.

Expect denial.

---

## Alert delivery tests

Trigger a controlled alert.

Verify it reaches intended notification target.

---

## Query-limit tests

Run expensive queries.

Verify timeout/quota/fairness controls.

---

## Retention tests

Verify data expires according to class.

Do not rely only on configured values.

---

## Failure injection

Simulate:

- collector crash;
- backend outage;
- network partition;
- storage pressure;
- notification outage.

Confirm documented behavior.

---

## Soak testing

Long-running tests can expose:

- memory leaks;
- queue growth;
- cardinality creep;
- storage skew;
- dropped telemetry.

---

## Acceptance path

A useful reference path:

```text
service emits telemetry
    ->
collector authenticates/enriches
    ->
platform validates/routes
    ->
backend stores
    ->
service owner queries
    ->
dashboard shows signal
    ->
alert evaluates
```

Negative path:

```text
service emits secret-bearing telemetry
    ->
redaction/filtering applies
    ->
secret absent from retained store
```

Failure path:

```text
backend unavailable
    ->
bounded buffering/drop policy
    ->
application continues
    ->
telemetry-loss signal emitted
    ->
pipeline recovers
```

Isolation path:

```text
tenant A queries tenant B telemetry
    ->
authorization denies
```

---

## Documentation

The platform should document:

- supported signals;
- semantic conventions;
- resource identity;
- ingestion;
- sampling;
- retention;
- access;
- alerting;
- SLOs;
- cost controls;
- failure behavior.

---

## Consumer documentation

Application teams should know:

- how to instrument;
- required metadata;
- what not to log;
- how to query;
- how to create alerts;
- ownership responsibilities.

---

## Operator documentation

Platform operators should know:

- pipeline architecture;
- buffering/drop behavior;
- capacity;
- retention;
- backup;
- recovery;
- tenant isolation.

---

## Runbooks

Create runbooks for:

- ingestion outage;
- query outage;
- cardinality explosion;
- storage pressure;
- alert delivery failure;
- cross-tenant access incident;
- telemetry loss;
- collector crash loop.

---

## Architecture diagrams

Useful views include:

### Signal flow

```text
Workload -> Collector -> Processor -> Backend -> Query/Alert
```

### Trust

```text
Workload Identity -> Ingest Auth -> Tenant Store -> Authorized Query
```

### Failure

```text
Backend down
    ->
buffer/drop
    ->
loss metric
    ->
recover
```

### Correlation

```text
Metric -> Trace -> Logs -> Deployment Event
```

---

## Architecture decisions

Use ADRs for consequential decisions such as:

- collection topology;
- multi-tenant isolation;
- sampling;
- retention;
- data residency;
- query architecture;
- alerting ownership;
- redaction model.

---

## Recommended repository shape

Follow existing repository conventions first.

A generic observability-platform repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── architecture/
│   ├── telemetry-model.md
│   ├── trust-boundaries.md
│   ├── data-flows.md
│   └── decisions/
├── contracts/
│   ├── resources/
│   ├── metrics/
│   ├── logs/
│   └── traces/
├── platform/
│   ├── collectors/
│   ├── processing/
│   ├── routing/
│   ├── storage/
│   ├── alerting/
│   └── access/
├── dashboards/
├── alerts/
├── tests/
│   ├── contract/
│   ├── integration/
│   ├── security/
│   ├── load/
│   ├── failure/
│   └── e2e/
├── docs/
│   ├── consumers/
│   ├── operators/
│   └── runbooks/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

An observability-platform repository should expose obvious commands or
equivalent native interfaces for:

```text
check
test
test-contract
test-integration
test-security
test-load
test-failure
test-alerts
test-e2e
verify
```

These names are illustrative.

Use only commands the repository can actually implement.

---

## Acceptance criteria

An observability platform is not complete because dashboards render or logs are
searchable.

Demonstrate the applicable subset of the following.

### Telemetry model

- supported signal types are explicit;
- resource identity is consistent;
- semantic conventions are documented;
- ownership is defined.

### Ingestion

- workloads authenticate;
- tenant/service identity is trusted;
- malformed telemetry is bounded;
- backpressure/drop behavior is defined.

### Security

- sensitive fields are minimized/redacted;
- cross-tenant queries are denied;
- administrative access is separated;
- telemetry export is controlled.

### Metrics

- cardinality is bounded;
- route/error labels are normalized;
- SLI metrics are unsampled where required.

### Logs

- logs are structured where appropriate;
- secrets are not retained;
- severity semantics are consistent.

### Traces

- context propagates;
- sampling is explicit;
- span names/attributes are bounded;
- sampled traces are not misused for exact counts.

### Alerting

- alerts are actionable;
- ownership/runbooks exist;
- alert delivery is tested;
- silences are scoped and expiring.

### SLOs

- SLIs are defined precisely;
- consumer-visible outcomes are measured;
- platform and application ownership are distinct.

### Operations

- collector/backend health is observable;
- telemetry loss is visible;
- capacity/cost drivers are known;
- retention actually works;
- degraded behavior is documented.

### Resilience

- backend outage does not normally break applications;
- buffers are bounded;
- failure recovery is tested;
- meta-monitoring detects pipeline failure.

---

## Optional composition

Common combinations:

```text
platform-architecture + observability-platform
```

For shared telemetry capability architecture and ownership.

```text
developer-platform + observability-platform
```

For default logs, metrics, traces, dashboards, and developer self-service.

```text
observability-platform + api-platform
```

For edge/service correlation, API SLOs, and request tracing.

```text
observability-platform + eventing-platform
```

For consumer lag, event-delivery freshness, retries, and replay observability.

```text
observability-platform + zero-trust-service
```

For workload identity, tenant-aware telemetry access, and privileged admin
boundaries.

```text
observability-platform + software-supply-chain
```

For trusted collectors, agents, plugins, dashboards/rules, and release
integrity.

```text
observability-platform + high-assurance
```

For stronger audit integrity, failure injection, telemetry-loss testing, and
independent alert verification.

---

## Anti-patterns

Avoid:

- calling centralized logging "observability";
- collecting everything forever;
- one global retention policy;
- one global ingestion credential;
- raw user/request IDs as metric labels;
- full URLs as labels;
- logging secrets and relying on future cleanup;
- debug logging enabled permanently in production;
- traces used as the only exact request counter;
- sampled telemetry used for authoritative SLI math without justification;
- platform dashboards with no owner;
- every alert paging;
- alerts with no runbook;
- permanent silences;
- logs treated as audit without integrity/retention controls;
- tenant isolation implemented only by index naming;
- observability backend outage causing application outage;
- unbounded collector buffers;
- silent telemetry drops;
- expensive queries with no limits;
- cardinality explosions discovered only through billing;
- service mesh telemetry treated as complete application observability;
- portal/dashboard availability used as the sole platform health measure;
- retention config assumed to equal deletion proof;
- cross-region telemetry replication ignoring data residency.

Do not mistake telemetry volume for observability quality.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. observability-platform purpose;
2. supported signal types;
3. resource identity/semantic convention model;
4. instrumentation model;
5. collection/ingestion topology;
6. authentication/tenant isolation model;
7. processing/redaction/routing model;
8. metrics/cardinality strategy;
9. logging/sensitive-data strategy;
10. tracing/sampling strategy;
11. alerting ownership and notification model;
12. SLI/SLO model;
13. retention/residency/deletion model;
14. capacity/cost/quota model;
15. failure/degraded-operation model;
16. contract/integration/alert tests executed;
17. security/load/failure tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim an observability platform is "complete", "secure", "multi-tenant",
"cost efficient", or "SRE ready" merely because metrics, logs, traces, and
dashboards exist.

Describe the telemetry contracts, resource identity, cardinality controls,
sensitive-data handling, access boundaries, failure behavior, retention, and
actual evidence.

---

## Guiding principle

A good observability platform should help engineers explain reality without
becoming another source of uncertainty.

Signals should mean something.

Resource identity should be consistent.

High-cardinality data should stay bounded.

Sensitive data should stay out unless intentionally collected.

Telemetry failures should be visible but should not usually break the product.

Alerts should be actionable.

SLOs should describe consumer outcomes.

Retention should reflect purpose.

And during an incident, the platform should make it possible to move from:

> Something is wrong.

to:

> This capability changed here, this dependency degraded there, these users
> were affected in this way, this release or condition correlates with the
> failure, and this is the evidence that supports that conclusion.
