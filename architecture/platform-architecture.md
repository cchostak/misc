# platform-architecture.md

> Foundational recipe for defining or elevating a shared platform architecture.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with platform-specific recipes such as `control-plane.md`,
> `developer-platform.md`, `identity-platform.md`, `api-platform.md`,
> `eventing-platform.md`, `observability-platform.md`, `data-platform.md`, or
> `ai-platform.md`, and with profiles such as `zero-trust-service.md`,
> `software-supply-chain.md`, `high-assurance.md`, `hostile-input.md`, or
> `ai-security.md` where appropriate.
>
> This recipe defines architecture expectations for platforms that provide
> reusable capabilities to multiple consumers, teams, workloads, or systems.
>
> It does not require a specific cloud, orchestrator, portal, service mesh,
> workflow engine, registry, control-plane implementation, or vendor.

## Purpose

Use this recipe when the repository or design effort defines a platform rather
than a single application.

Typical examples:

- internal developer platforms;
- security platforms;
- identity platforms;
- API platforms;
- data platforms;
- AI platforms;
- observability platforms;
- deployment platforms;
- infrastructure platforms;
- integration platforms;
- shared control planes;
- multi-tenant enterprise services;
- reusable enterprise capability layers.

The goal is not to assemble a large set of infrastructure products.

The goal is to define a coherent platform whose consumers can understand:

- what capability the platform provides;
- what contract they depend on;
- what the platform owns;
- what the consumer owns;
- what is self-service;
- what is governed;
- how trust is established;
- how changes are introduced;
- how failures are contained;
- how adoption and migration work;
- how the platform itself is operated.

A platform is a product with consumers, contracts, lifecycle, and an operating
model.

It is not merely a collection of shared infrastructure.

---

## Core principle

Define the platform contract before selecting the platform implementation.

A useful architecture model is:

```text
Consumers
    |
    v
Platform Contract
    |
    +--> APIs
    +--> protocols
    +--> schemas
    +--> policies
    +--> self-service workflows
    +--> lifecycle guarantees
    |
    v
Platform Capabilities
    |
    +--> control plane
    +--> data/runtime plane
    +--> identity
    +--> policy
    +--> state
    +--> observability
    +--> automation
    |
    v
Underlying Infrastructure / Products
```

Consumers should depend primarily on the platform contract.

They should not need to understand every implementation product beneath it.

At the same time, the abstraction must not hide consequential behavior such as:

- availability boundaries;
- data location;
- security policy;
- irreversible actions;
- cost;
- latency;
- eventual consistency;
- failure modes.

---

## Architectural invariants

A platform architecture SHOULD satisfy these invariants unless there is a
documented reason not to.

### 1. The platform has explicit consumers

Identify who consumes the platform.

Examples:

- application teams;
- data teams;
- security teams;
- operators;
- developers;
- automated systems;
- external partners.

Do not design an abstract "enterprise platform" with no concrete consumer model.

### 2. The platform contract is explicit

The platform should expose clear contracts through one or more of:

- API;
- protocol;
- CLI;
- SDK;
- resource model;
- event schema;
- policy interface;
- declarative configuration;
- portal/workflow.

Consumers should know what is stable.

### 3. Platform ownership is bounded

Define what the platform owns and what it does not.

Do not let the platform gradually absorb every adjacent responsibility.

### 4. Control plane and runtime/data plane are distinguishable

Where applicable, separate:

```text
decision / orchestration / policy / desired state
```

from:

```text
workload execution / traffic / data movement / actual resources
```

This improves:

- failure isolation;
- authorization;
- scaling;
- operability;
- upgrade design.

### 5. Authoritative state is explicit

For every important resource or decision, identify the system of record.

Do not create multiple writable sources of truth without conflict semantics.

### 6. Trust boundaries are explicit

Identify:

- human identity;
- workload identity;
- external systems;
- control-plane identity;
- tenant boundaries;
- privilege transitions.

Do not let network location substitute silently for identity or authorization.

### 7. Self-service is policy-bound

Self-service should reduce coordination cost without bypassing governance.

A consumer may request capability.

The platform should still validate and authorize the request.

### 8. Platform abstractions are versioned

Platform APIs, resource models, templates, policies, and protocols evolve.

Treat them as compatibility-sensitive interfaces.

### 9. Failure domains are intentional

A platform should not create a larger blast radius than necessary.

Identify what fails together.

### 10. Escape hatches are explicit

Some consumers will eventually need behavior outside the paved road.

Decide:

- what is allowed;
- under what review;
- who owns resulting risk;
- whether the path remains supported.

Do not force unsupported workarounds into production.

### 11. Operability is part of architecture

A platform design is incomplete without:

- ownership;
- telemetry;
- capacity model;
- incident model;
- upgrade path;
- recovery path.

### 12. Adoption is an architectural concern

A platform with no viable migration path is not reusable architecture.

Design onboarding and exit paths.

---

## Platform intent

Start by writing a concise platform intent statement.

It should answer:

```text
For <consumer>,
the platform provides <capability>,
through <contract>,
while owning <responsibility>,
and deliberately not owning <non-goal>.
```

Example:

```text
For application teams, the platform provides repeatable service deployment
through a declarative deployment API while owning runtime integration,
policy enforcement, and promotion mechanics. Application architecture and
business logic remain owned by the service team.
```

This is more useful than a product inventory.

---

## Problem statement

Document the actual problems the platform exists to solve.

Examples:

- inconsistent delivery pipelines;
- duplicated identity integrations;
- unsafe cloud provisioning;
- fragmented observability;
- slow environment creation;
- inconsistent model access;
- uncontrolled API exposure;
- poor tenant isolation.

Avoid generic goals such as:

```text
improve developer experience
increase standardization
enable cloud native
```

unless tied to specific observable problems.

---

## Non-goals

Platform non-goals are essential.

Examples:

- platform does not own application business logic;
- platform does not replace application authorization;
- platform does not mandate one programming language;
- platform does not store all application secrets centrally;
- platform does not become a universal workflow engine.

Non-goals prevent architecture creep.

---

## Consumer model

Identify each consumer class.

For each consumer define:

- identity;
- needs;
- privileges;
- lifecycle;
- support expectations;
- expected scale;
- failure tolerance.

A developer, production workload, security automation, and platform operator
should not be treated as the same consumer.

---

## Consumer journeys

Describe the important platform journeys.

Examples:

```text
create capability
configure capability
use capability
observe capability
update capability
recover capability
delete capability
```

For a developer platform:

```text
create service
    ->
build
    ->
deploy
    ->
observe
    ->
promote
    ->
retire
```

Architecture should support real workflows, not only API objects.

---

## Platform contract

The platform contract may contain:

- resource/API schema;
- service API;
- event schema;
- SDK;
- CLI;
- declarative manifest;
- lifecycle semantics;
- SLO/SLA commitments;
- policy behavior;
- versioning rules.

Document which parts are stable.

Do not make implementation details part of the contract accidentally.

---

## Contract layers

Distinguish:

```text
consumer contract
platform-internal contract
vendor/product contract
```

Consumers should not depend directly on vendor-specific implementation details
unless that is intentional.

---

## Control plane

Where the platform has a control plane, define what it owns.

Typical responsibilities:

- desired state;
- policy decisions;
- orchestration;
- resource lifecycle;
- reconciliation;
- configuration;
- promotion;
- tenancy;
- audit.

Do not overload the control plane with runtime data-path responsibilities
without reason.

Compose with `control-plane.md` for deeper guidance.

---

## Data plane / runtime plane

Define the plane where actual work occurs.

Examples:

- application workloads;
- network traffic;
- data processing;
- model inference;
- message delivery;
- secret retrieval;
- build execution.

The runtime/data plane should continue to behave predictably if the control
plane is temporarily unavailable where the architecture permits.

---

## Control-plane failure

Ask:

> What happens to existing workloads if the control plane disappears for ten
> minutes?

Possible answers:

- existing workloads continue;
- new provisioning stops;
- policy changes stop;
- traffic continues using cached configuration;
- writes fail closed.

Document the intended behavior.

Do not accidentally make the control plane part of every runtime request path.

---

## Management plane

If there is a separate management/admin plane, define it.

Typical operations:

- tenant creation;
- policy management;
- platform upgrades;
- credential rotation;
- break-glass access.

Use stronger authorization than ordinary consumer operations.

---

## Authoritative state

For each major domain identify the source of truth.

Examples:

```text
desired workload state -> Git repository
runtime state          -> orchestrator API
identity state         -> identity provider
artifact state         -> registry
policy state           -> policy repository/service
billing state          -> billing system
```

Do not use caches or dashboards as authoritative state.

---

## State ownership

If multiple systems hold copies of state, define:

- authoritative owner;
- replication direction;
- conflict behavior;
- recovery path.

Avoid bidirectional synchronization without explicit conflict semantics.

---

## Reconciliation

Prefer reconciliation over imperative orchestration when the domain represents
desired state.

A reconciler should:

1. read desired state;
2. observe actual state;
3. calculate delta;
4. perform bounded actions;
5. record status;
6. repeat safely.

Use workflow/state-machine orchestration instead when the domain is genuinely
a sequence of one-time steps.

Do not force every problem into either model.

---

## Workflow versus reconciliation

Use reconciliation when:

- desired state persists;
- drift should be repaired;
- retries are expected;
- resources have lifecycle.

Use workflow when:

- sequence matters;
- state transitions are finite;
- steps should not repeat indefinitely.

Some platforms use both.

Make the boundary explicit.

---

## Platform API

The platform API should expose domain capability, not vendor primitives.

Prefer:

```text
create database capability
```

over:

```text
create cloud-product-specific database instance
```

unless vendor detail is intentionally exposed.

Avoid lowest-common-denominator APIs that eliminate useful capabilities merely
for portability.

---

## Resource model

For declarative platforms, define resource semantics.

A resource should have:

- identity;
- spec/desired state;
- status/observed state;
- ownership;
- lifecycle;
- version.

Keep resource models cohesive.

Do not create one "PlatformResource" object with hundreds of unrelated fields.

---

## API versioning

Treat platform APIs as long-lived contracts.

Define:

- compatible change;
- breaking change;
- deprecation;
- migration;
- removal.

Do not tie API version mechanically to implementation version.

---

## Schema evolution

Schemas should evolve safely.

Prefer additive changes.

For breaking changes:

- provide migration;
- support parallel versions where needed;
- communicate lifecycle.

Do not silently reinterpret existing fields.

---

## Protocol design

If platform components communicate through custom protocols, define:

- framing;
- versioning;
- identity;
- errors;
- retries;
- ordering;
- cancellation;
- compatibility;
- resource limits.

Compose with `protocol-design.md` when available.

---

## API ownership

Every API/resource should have an owning platform domain/team.

Ownership includes:

- schema changes;
- documentation;
- support;
- migration.

Do not create "shared APIs" nobody owns.

---

## Platform capability boundaries

Decompose the platform into capabilities.

Examples:

```text
identity
build
artifact
deployment
runtime
networking
secrets
observability
data
policy
AI/model access
```

Each capability should have:

- owner;
- contract;
- dependencies;
- lifecycle.

---

## Capability cohesion

A capability should represent one coherent responsibility.

Do not combine unrelated functions merely because one team operates them.

---

## Dependency graph

Document platform capability dependencies.

Prefer acyclic dependencies where possible.

Watch for loops such as:

```text
identity platform depends on developer platform
developer platform depends on identity platform
```

Bootstrap architecture must break such cycles deliberately.

---

## Bootstrap architecture

Every platform needs a bootstrap path.

Ask:

- What creates the platform itself?
- What exists before the platform?
- Which identities exist first?
- Where are initial secrets stored?
- How are foundational dependencies installed?
- How is bootstrap recovered?

Do not require the fully operational platform to create the platform from
nothing.

---

## Day 0 / Day 1 / Day 2

Separate lifecycle concerns.

### Day 0

- initial provisioning;
- root trust;
- foundational identities;
- bootstrap.

### Day 1

- consumer onboarding;
- resource creation;
- configuration.

### Day 2

- upgrades;
- incidents;
- rotation;
- migrations;
- capacity;
- decommissioning.

A design that only explains Day 1 is incomplete.

---

## Platform tenancy

Define tenancy explicitly.

Potential scopes:

- user;
- team;
- project;
- business unit;
- customer;
- environment.

For each tenant boundary define:

- identity;
- namespace/resource isolation;
- data isolation;
- quota;
- audit;
- billing/cost attribution;
- administrative delegation.

---

## Isolation model

Isolation may be:

- logical;
- process;
- namespace;
- account/project;
- cluster;
- physical.

Use isolation proportionate to risk.

Do not claim "multi-tenant" without defining what is isolated from what.

---

## Shared versus dedicated resources

Decide which platform components are shared.

Examples:

```text
shared control plane
dedicated runtime plane
shared metadata
tenant-isolated data
```

Document blast-radius consequences.

---

## Noisy-neighbor control

Shared platforms need:

- quotas;
- rate limits;
- concurrency limits;
- capacity fairness.

Do not let one tenant exhaust shared control-plane or runtime capacity.

---

## Identity model

Identify all principal classes.

Typical classes:

- human user;
- workload;
- automation;
- platform controller;
- platform operator;
- external service.

Do not reuse one identity model for fundamentally different actors without
reason.

---

## Authentication

Use established identity protocols/mechanisms.

Do not invent custom authentication if established federation works.

Authentication answers:

> Who or what is this?

It does not answer:

> May it perform this action?

---

## Authorization

Authorization should be resource- and action-aware.

Potential models:

- RBAC;
- ABAC;
- ReBAC;
- capability-based;
- policy-as-code.

Avoid using platform role alone when resource ownership matters.

---

## Delegation

Platforms often act on behalf of consumers.

Preserve:

- initiating identity;
- platform identity;
- delegated authority.

Do not let a broad platform credential become silent privilege escalation.

---

## Workload identity

Prefer short-lived workload identity where supported.

Avoid distributing static credentials to every workload.

---

## Break-glass

Define emergency access.

A break-glass path should be:

- explicit;
- rare;
- audited;
- time-bounded;
- revocable.

Do not make normal operation depend on break-glass privileges.

---

## Policy architecture

Separate:

```text
policy definition
policy decision
policy enforcement
```

where useful.

A central policy engine is not mandatory.

Use the simplest architecture that keeps enforcement reliable.

---

## Policy enforcement points

Identify where policy is actually enforced.

Examples:

- API gateway;
- admission;
- controller;
- runtime;
- database;
- artifact registry;
- CI;
- deployment system.

Do not claim a policy exists if no enforcement point applies it.

---

## Policy exceptions

Exceptions should have:

- owner;
- rationale;
- scope;
- expiration;
- review.

Avoid permanent invisible bypasses.

---

## Self-service

Self-service should operate within bounded policy.

A good self-service flow:

```text
consumer request
    ->
validation
    ->
authorization
    ->
policy
    ->
provision/reconcile
    ->
status/evidence
```

Do not equate self-service with unrestricted access.

---

## Golden paths

A golden path is a supported default workflow.

It should:

- reduce repeated decisions;
- encode good defaults;
- remain understandable;
- allow justified deviation.

Do not turn the golden path into an undocumented mandatory prison.

---

## Paved road versus mandated road

Define what is:

```text
RECOMMENDED
SUPPORTED
REQUIRED
EXCEPTION-ONLY
UNSUPPORTED
```

Consumers should understand consequences of leaving the paved road.

---

## Escape hatches

A platform should have explicit escape-hatch policy.

Questions:

- Can consumers bring custom runtime components?
- Can they bypass platform networking?
- Can they use external data stores?
- Who owns support afterward?
- Which security controls remain mandatory?

Do not make exceptions accidental.

---

## Extensibility

Design extension points only where real variation exists.

Potential extension mechanisms:

- plugins;
- adapters;
- policy hooks;
- provider interfaces;
- custom resources;
- webhooks;
- event subscribers.

Every extension point expands compatibility and security surface.

---

## Plugin trust

If third-party plugins/extensions exist:

- authenticate publisher;
- version interface;
- isolate execution where necessary;
- define privileges;
- define lifecycle.

Do not let plugins inherit full platform authority automatically.

---

## Platform portal

A portal may improve discoverability.

It should not become the architecture.

The underlying platform capability should remain accessible through a stable
contract independent of the UI where practical.

The platform API is more important than the platform portal.

---

## CLI and SDK

Provide CLI/SDKs when they materially improve adoption.

They should wrap the platform contract rather than become the only supported
interface.

Compose with `cli.md` or `library-sdk.md`.

---

## Templates

Templates are useful for common starting points.

Treat templates as versioned products.

Avoid templates that:

- embed stale dependencies;
- hard-code ownership;
- create unnecessary resources;
- diverge from supported platform APIs.

---

## Service catalog

A catalog can help discover:

- ownership;
- resources;
- dependencies;
- documentation.

Do not treat catalog metadata as automatically authoritative.

Prefer synchronization from real systems of record where possible.

---

## Metadata model

Define which metadata matters.

Examples:

- owner;
- tenant;
- environment;
- service tier;
- data classification;
- lifecycle state.

Avoid metadata fields nobody maintains.

---

## Platform events

Publish lifecycle events where useful.

Examples:

- resource created;
- deployment promoted;
- certificate expiring;
- policy denied;
- capability deprecated.

Use explicit event schemas.

Do not make consumers poll if reliable events already exist.

---

## Event semantics

For platform events define:

- identity;
- ordering;
- delivery semantics;
- retention;
- replay;
- schema version.

Compose with `eventing-platform.md` for deeper architecture.

---

## Data architecture

If the platform stores consumer data, define:

- source of truth;
- ownership;
- classification;
- retention;
- backup;
- deletion;
- replication.

Do not accumulate shadow copies of sensitive data without purpose.

---

## Secrets architecture

Define how platform and consumer secrets are handled.

Distinguish:

- platform secrets;
- tenant secrets;
- workload secrets;
- bootstrap secrets.

Avoid copying secrets across layers unnecessarily.

---

## Configuration architecture

Separate:

```text
platform configuration
tenant configuration
resource configuration
runtime configuration
```

Define precedence and ownership.

Do not let one global config file control unrelated tenant behavior.

---

## Environment model

Define environment semantics.

Examples:

- dev;
- test;
- staging;
- production.

Environment is a security and lifecycle attribute.

Do not rely only on naming conventions.

---

## Environment isolation

Define what differs across environments:

- identities;
- accounts/projects;
- network;
- data;
- credentials;
- policy.

Avoid accidental production reachability from development control paths.

---

## Promotion

For platforms managing artifacts/configuration, promotion should be explicit.

Prefer promoting immutable artifacts or desired-state references.

Do not rebuild or mutate artifacts differently per environment without reason.

---

## Artifact model

If the platform handles software/model/data artifacts, define:

- identity;
- provenance;
- immutability;
- signing;
- retention;
- promotion.

Compose with `software-supply-chain.md`.

---

## Supply-chain boundary

The platform itself is privileged infrastructure.

Protect:

- source;
- build;
- dependencies;
- images/packages;
- plugins;
- policy bundles;
- templates.

A compromised platform artifact has broad blast radius.

---

## Network architecture

Define:

- north-south traffic;
- east-west traffic;
- control-plane traffic;
- management traffic;
- egress.

Network architecture should reflect trust boundaries.

Do not use network segmentation as the sole authorization mechanism.

---

## Service-to-service communication

Define:

- identity;
- encryption;
- discovery;
- retry;
- timeout;
- authorization.

Avoid implicit trust merely because traffic is internal.

---

## Public exposure

If parts of the platform are internet-facing:

- isolate edge concerns;
- use strong authentication;
- rate limit;
- protect admin paths;
- monitor abuse.

Compose with `internet-facing.md` where available.

---

## Dependency model

List critical dependencies.

For each dependency define:

- why required;
- availability impact;
- timeout;
- retry;
- fallback;
- owner.

Do not build a platform with hidden transitive operational dependencies.

---

## Dependency criticality

Classify:

```text
required for runtime
required for new provisioning
required for administration
optional
```

This helps design degraded behavior.

---

## Failure domains

Identify what fails together.

Examples:

- region;
- cluster;
- account;
- tenant shard;
- control-plane instance;
- shared database.

Do not assume replicas create independent failure domains.

---

## Blast radius

For each privileged component ask:

> If this component is compromised or misconfigured, how many consumers can it
> affect?

Reduce unnecessary shared privilege.

---

## Fault containment

Use boundaries such as:

- tenant partitioning;
- account separation;
- cluster separation;
- shard separation;
- scoped credentials;
- queue isolation.

Do not make every failure global.

---

## Availability

Define availability according to consumer need.

Do not invent "five nines".

Different platform functions may have different availability needs.

Example:

```text
runtime data plane      -> high availability
new resource creation   -> lower availability acceptable
reporting               -> eventual
```

---

## Degraded operation

Document behavior during partial outage.

Examples:

- existing services continue;
- new provisioning stops;
- read operations continue;
- policy updates pause;
- cached configuration remains active.

A platform should degrade intentionally.

---

## Dependency outage

For every critical dependency, define:

- timeout;
- retry;
- cached state;
- failure mode;
- alert.

Avoid cascading retry storms.

---

## Backpressure

Shared platform APIs need:

- queue bounds;
- rate limits;
- concurrency control.

Do not let overload become memory exhaustion.

---

## Capacity model

Define capacity units meaningful to the platform.

Examples:

- tenants;
- resources;
- API requests;
- builds;
- deployments;
- events/sec;
- telemetry volume;
- model tokens;
- storage.

Capacity is part of architecture.

---

## Scaling

Identify scaling axes.

Examples:

```text
horizontal controller scale
tenant sharding
regional partitioning
worker pools
read replicas
```

Do not introduce distributed complexity before scale requires it.

---

## Sharding

Use sharding when:

- blast radius;
- scale;
- locality;
- tenancy

justify it.

Define shard assignment and migration.

---

## Regional architecture

If multi-region:

- define active/active or active/passive;
- define data ownership;
- define failover;
- define residency;
- define control-plane behavior.

Do not draw two regions on a diagram and call it resilient.

---

## Disaster recovery

Define what must be recoverable:

- desired state;
- platform metadata;
- identities/config;
- artifacts;
- consumer data where owned.

Document recovery dependencies.

Do not invent RPO/RTO without requirements.

---

## Backup

Back up state that cannot be reconstructed.

Avoid backing up caches and derived data unless recovery time requires it.

Test restore for critical state.

---

## Rebuildability

For derived platform components ask:

> Can this be rebuilt from authoritative state?

Prefer rebuildable architecture where practical.

---

## Observability

The platform should expose enough telemetry to operate:

- APIs;
- controllers;
- queues;
- dependencies;
- tenant saturation;
- provisioning latency;
- failure rates;
- policy denials.

Do not collect telemetry without ownership or operational use.

---

## Platform health

Separate:

```text
platform process health
platform capability health
consumer outcome health
```

A green dashboard does not prove consumers can successfully use the platform.

---

## SLOs

Define SLOs around consumer-visible platform capabilities where appropriate.

Examples:

- API availability;
- provisioning latency;
- deployment convergence;
- authentication success;
- event delivery delay.

Avoid SLOs for metrics no consumer cares about.

---

## Audit

Audit security- and lifecycle-relevant platform actions.

Examples:

- privilege changes;
- tenant creation;
- policy changes;
- production promotion;
- break-glass use;
- destructive resource deletion.

Audit should identify actor and target.

---

## Logging

Use structured logs.

Avoid logging:

- secrets;
- tokens;
- full sensitive payloads.

Include stable identifiers useful for correlation.

---

## Tracing

Use tracing where distributed platform flows justify it.

Keep trace propagation consistent across capability boundaries.

---

## Cost architecture

Shared platforms create shared cost.

Define:

- cost drivers;
- quotas;
- chargeback/showback if required;
- expensive operations;
- capacity ownership.

Do not hide unbounded cost behind self-service.

---

## Cost attribution

Where useful, attribute cost to:

- tenant;
- team;
- environment;
- capability.

Avoid high-maintenance attribution if nobody acts on it.

---

## Abuse controls

Shared platforms need protection from accidental or malicious abuse.

Controls may include:

- quotas;
- rate limits;
- admission policy;
- concurrency bounds;
- budget limits.

Self-service should not mean unlimited.

---

## Platform security model

Document:

- trust boundaries;
- privileged components;
- identities;
- credentials;
- admin surfaces;
- tenant boundaries;
- external dependencies.

A platform's threat model should account for compromised consumers.

---

## Compromised consumer

Assume one consumer may be compromised.

Ask:

- Can they affect other tenants?
- Can they escalate through platform APIs?
- Can they poison shared metadata?
- Can they exhaust capacity?
- Can they access platform credentials?

Design for containment.

---

## Compromised platform component

Assume a controller or service may be compromised.

Minimize:

- privileges;
- credential scope;
- cross-tenant access;
- write authority.

Do not make every platform service an administrator.

---

## Administrative separation

Separate ordinary operator actions from high-risk administrative actions.

Use stronger controls for:

- root trust;
- identity configuration;
- global policy;
- tenant boundary changes;
- destructive global operations.

---

## Change management

Platform changes affect many consumers.

Classify changes by impact.

Examples:

```text
internal implementation
compatible contract change
consumer migration required
breaking platform change
security emergency
```

Use a process appropriate to impact.

---

## Compatibility window

For breaking changes, define a migration window.

Support:

- old and new APIs;
- feature flags;
- compatibility adapters;
- migration tooling

where justified.

Do not force synchronized enterprise-wide upgrades without necessity.

---

## Deprecation

A deprecation should state:

- what is deprecated;
- replacement;
- migration path;
- date/release boundary;
- operational impact.

Do not deprecate by quietly removing documentation.

---

## Migration

Every platform capability should have an onboarding and migration story.

Migration may include:

- discovery;
- assessment;
- dry-run;
- import;
- parallel operation;
- cutover;
- rollback.

---

## Exit strategy

Consumers should be able to leave the platform where reasonable.

Avoid unnecessary lock-in to hidden implementation.

Document:

- export formats;
- ownership transfer;
- data extraction;
- cleanup.

A platform that cannot be exited becomes organizational risk.

---

## Adoption model

Adoption should be intentional.

Possible strategies:

- opt-in paved road;
- default for new systems;
- mandatory for regulated workloads;
- migration campaign.

Architecture should support the chosen adoption model.

---

## Consumer support

Define:

- ownership;
- support channel;
- escalation;
- incident responsibility.

Do not create a platform that consumers cannot get help operating.

---

## Responsibility matrix

Document responsibilities.

A simple model:

```text
Platform owns:
- platform API
- shared control plane
- policy enforcement
- platform runtime integration

Consumer owns:
- business logic
- application data semantics
- service-level authorization
- application SLOs

Shared:
- incident response
- capacity planning
- migration
```

Avoid ambiguous ownership during incidents.

---

## Product management

Treat platform capabilities as products.

For each capability track:

- consumers;
- adoption;
- reliability;
- pain points;
- lifecycle;
- roadmap.

Do not judge platform success solely by infrastructure uptime.

---

## Developer experience

Developer experience includes:

- time to first success;
- clarity;
- error quality;
- self-service;
- documentation;
- local testing;
- migration burden.

Do not reduce developer experience to a portal UI.

---

## Feedback loops

Collect consumer feedback through:

- support issues;
- adoption metrics;
- migration failures;
- operational incidents.

Use evidence to evolve the platform.

---

## Documentation

A platform architecture should document:

- purpose;
- consumers;
- capabilities;
- boundaries;
- contracts;
- identity;
- tenancy;
- policy;
- failure domains;
- lifecycle;
- operating model;
- migration;
- non-goals.

Do not document only product installation.

---

## Architecture diagrams

Use diagrams that communicate decisions.

Useful views:

### Context

```text
Consumers -> Platform -> External Systems
```

### Capability

```text
API / identity / policy / state / runtime / observability
```

### Trust boundaries

```text
tenant -> platform -> privileged systems
```

### Data/control flows

```text
request -> policy -> control plane -> runtime
```

Avoid diagrams that are only vendor logos connected by arrows.

---

## Decision records

Use ADRs for consequential decisions such as:

- control-plane model;
- API design;
- tenancy model;
- authoritative state;
- identity architecture;
- policy architecture;
- regional architecture;
- escape-hatch strategy;
- vendor dependence.

Do not create ADRs for trivial implementation details.

---

## Technology selection

Select technology after requirements and boundaries are understood.

Evaluate candidates against:

- capability fit;
- maturity;
- operability;
- failure behavior;
- integration;
- security;
- portability;
- support;
- cost.

Do not choose a product because it is fashionable.

---

## Build versus buy

For each platform capability ask:

```text
Is this differentiating?
Is this commodity?
Can we operate it?
Can we replace it?
What is the switching cost?
```

Do not build commodity infrastructure merely to avoid procurement.

Do not buy an opaque platform that prevents required control.

---

## CNCF/ecosystem use

Use ecosystem landscapes as maps, not shopping lists.

Adopt the minimum number of components needed to satisfy the architecture.

Avoid one product per box.

---

## Vendor boundaries

Encapsulate vendor-specific behavior where replacement or multi-provider
support is a real requirement.

Do not build speculative abstraction layers solely to claim portability.

---

## Portability

Be precise about what must be portable.

Possible goals:

- API contract portability;
- deployment portability;
- data portability;
- cloud portability;
- operational knowledge portability.

Full portability is expensive.

Do not promise it vaguely.

---

## Local development

Provide a development path for platform components.

Use:

- fakes;
- local emulators;
- lightweight clusters;
- test tenants;
- disposable environments.

Do not require production-scale infrastructure for every code change.

---

## Reference environment

A reference environment may demonstrate the architecture.

It should be:

- representative;
- reproducible;
- intentionally smaller than production.

Do not present a demo environment as proof of production scalability.

---

## Verification strategy

Platform verification should include several layers.

### Component tests

Validate individual services/controllers/libraries.

### Contract tests

Validate public APIs/protocols.

### Integration tests

Validate platform capability interactions.

### Policy tests

Validate security and governance controls.

### Failure tests

Validate dependency loss, retry, recovery, and partial outage.

### End-to-end tests

Validate real consumer journeys.

### Upgrade tests

Validate compatibility and migration.

---

## Consumer journey tests

At least one important journey should be executable end to end.

Examples:

```text
consumer authenticates
    ->
requests capability
    ->
policy allows
    ->
platform provisions
    ->
status becomes ready
    ->
consumer uses capability
    ->
consumer deletes capability
    ->
cleanup completes
```

Also test a denied path.

---

## Failure-path verification

Demonstrate failures such as:

- authorization denial;
- dependency outage;
- partial provisioning;
- stale state;
- retry;
- rollback;
- tenant quota exceeded.

A platform that is only tested when everything works is not ready.

---

## Upgrade verification

Test:

- platform component upgrades;
- API compatibility;
- resource migrations;
- old/new version coexistence where relevant;
- rollback constraints.

---

## Chaos/fault injection

For high-assurance or high-scale platforms, inject failures such as:

- controller loss;
- queue outage;
- data-store failover;
- external API timeout;
- region loss;
- credential revocation.

Use only where risk justifies complexity.

---

## Security verification

Test:

- tenant isolation;
- authn/authz;
- privilege escalation;
- policy bypass;
- secret exposure;
- admin separation.

Compose with security profiles for deeper requirements.

---

## Load and scale testing

Test the dimensions that matter.

Examples:

- tenants;
- resources;
- API requests;
- queued operations;
- event throughput.

Do not test only single-user functional behavior.

---

## Readiness review

Before calling the platform ready for production use, verify:

- architecture boundaries;
- contracts;
- ownership;
- operating model;
- security model;
- capacity;
- failure behavior;
- migration;
- support.

---

## Recommended repository shape

Follow the repository's existing conventions first.

A platform architecture repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── architecture/
│   ├── context.md
│   ├── capabilities.md
│   ├── trust-boundaries.md
│   ├── data-flows.md
│   └── decisions/
├── api/
├── contracts/
├── policies/
├── components/
├── tests/
│   ├── contract/
│   ├── integration/
│   ├── security/
│   ├── failure/
│   └── e2e/
├── docs/
│   ├── consumers/
│   ├── operations/
│   ├── migration/
│   └── runbooks/
└── <ecosystem build/dependency files>
```

Do not force this structure if the platform lives in multiple repositories.

Architecture may span repositories while contracts remain coherent.

---

## Verification interface

A platform architecture project should expose obvious commands or equivalent
native interfaces for the parts it can actually execute.

Examples:

```text
check
test
test-contract
test-policy
test-integration
test-security
test-failure
test-e2e
verify
```

The exact commands depend on implementation.

Do not invent commands that have no executable meaning.

---

## Acceptance criteria

A platform architecture is not complete because a diagram exists or a set of
products has been selected.

Demonstrate the applicable subset of the following.

### Intent

- platform purpose is explicit;
- consumers are identified;
- non-goals are explicit;
- platform/consumer responsibilities are separated.

### Contract

- consumer-facing contracts are defined;
- versioning/deprecation rules exist;
- implementation details are not accidentally exposed.

### Architecture

- control/runtime plane boundaries are explicit;
- authoritative state is identified;
- platform capabilities are decomposed coherently;
- bootstrap dependencies are understood.

### Security

- identities are explicit;
- authorization boundaries are explicit;
- tenant isolation is defined;
- privileged components are minimized;
- policy enforcement points are known.

### Reliability

- failure domains are identified;
- degraded operation is defined;
- dependency outage behavior is understood;
- recovery paths exist.

### Operability

- ownership is defined;
- telemetry exists;
- capacity units are known;
- administrative actions are auditable;
- support/runbooks exist where needed.

### Lifecycle

- onboarding path exists;
- migration path exists;
- deprecation path exists;
- upgrade compatibility is considered;
- consumer exit is possible where required.

### Verification

- at least one consumer journey is tested end to end;
- at least one deny/failure path is tested;
- contract compatibility is tested;
- tenant/security controls are tested where applicable.

---

## Optional composition

Common combinations:

```text
platform-architecture + control-plane
```

For platforms centered on declarative desired state, orchestration, policy, and
reconciliation.

```text
platform-architecture + developer-platform
```

For internal developer platforms and paved-road architectures.

```text
platform-architecture + identity-platform
```

For shared identity, federation, credential, and authorization capability.

```text
platform-architecture + api-platform
```

For shared API exposure, governance, routing, and lifecycle.

```text
platform-architecture + eventing-platform
```

For shared event, queue, schema, replay, and delivery capability.

```text
platform-architecture + observability-platform
```

For shared telemetry ingestion, processing, query, and SLO capabilities.

```text
platform-architecture + data-platform
```

For enterprise/shared ingestion, storage, processing, governance, and query.

```text
platform-architecture + ai-platform
```

For shared model, RAG, agent, tool, evaluation, policy, and AI runtime
capabilities.

```text
platform-architecture + zero-trust-service + software-supply-chain
```

For explicit trust boundaries and strong artifact/release integrity.

---

## Anti-patterns

Avoid:

- calling a collection of products a platform;
- vendor-logo architecture diagrams;
- portal-first design with no stable platform API;
- one giant platform service owning every concern;
- hidden system-of-record ambiguity;
- bidirectional sync without conflict semantics;
- consumer-facing vendor primitives by accident;
- self-service with no policy;
- mandatory golden paths with no exception model;
- unlimited shared-tenancy capacity;
- one platform credential with global admin authority;
- production reachable from development control paths without strong boundary;
- control plane placed in every runtime request path unnecessarily;
- global outages caused by one shared metadata store with no recovery plan;
- escape hatches that silently remove support/security;
- "multi-region" diagrams with no failover semantics;
- "HA" claims based solely on replica count;
- platform APIs with no compatibility policy;
- breaking consumer changes without migration;
- product adoption measured only by number of components installed;
- platform success measured only by platform uptime;
- architecture that has no consumer exit strategy;
- abstraction layers created solely to pretend every vendor is interchangeable.

Do not mistake centralized infrastructure for platform architecture.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. platform purpose;
2. consumer classes;
3. non-goals;
4. platform/consumer responsibility boundary;
5. platform contract;
6. capability decomposition;
7. control-plane/runtime-plane model;
8. authoritative state model;
9. identity and authorization model;
10. tenancy/isolation model;
11. policy enforcement model;
12. bootstrap model;
13. failure domains and degraded behavior;
14. capacity/scaling model;
15. observability/audit model;
16. lifecycle, deprecation, and migration model;
17. verification performed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a platform is "enterprise-ready", "secure", "self-service",
"multi-tenant", "highly available", or "cloud agnostic" merely because the
architecture contains products associated with those properties.

Describe the contracts, boundaries, enforcement, failure behavior, migration,
and evidence that make those claims true.

---

## Guiding principle

A platform should reduce repeated complexity without creating hidden complexity.

Consumers should understand the contract.

Platform teams should understand the ownership boundary.

Identity should be explicit.

Policy should be enforceable.

State should have an authority.

Failures should have a boundary.

Upgrades should preserve consumers.

Exceptions should be visible.

Migration should be possible.

And technology choices should remain implementation details until there is a
good architectural reason for them to become part of the platform contract.
