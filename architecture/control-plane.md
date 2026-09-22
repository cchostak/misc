# control-plane.md

> Foundational recipe for scaffolding or elevating a generic control plane.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md` and implementation recipes such as
> `operator-controller.md`, `secure-service.md`, `kubernetes-workload.md`,
> `library-sdk.md`, or `cli.md`, plus profiles such as
> `zero-trust-service.md`, `software-supply-chain.md`, `high-assurance.md`,
> `hostile-input.md`, or `ai-security.md` where appropriate.
>
> This recipe defines architectural, state, reconciliation, policy,
> authorization, lifecycle, failure, scaling, and operability expectations for
> systems that manage desired state and coordinate changes across one or more
> data planes or external systems.
>
> It does not require Kubernetes, a specific workflow engine, a service mesh,
> a cloud, a particular database, a message broker, or any specific controller
> framework.

## Purpose

Use this recipe when the system primarily exists to decide, coordinate, or
reconcile what should happen elsewhere.

Typical examples:

- infrastructure control planes;
- deployment control planes;
- fleet-management systems;
- policy control planes;
- identity/control services;
- network control planes;
- storage control planes;
- AI/agent control planes;
- multi-cluster management;
- resource orchestration platforms;
- external-system reconcilers;
- enterprise automation platforms.

The goal is not to create a "controller service".

The goal is to create a control plane where:

- desired state is explicit;
- actual state is observable;
- authority is bounded;
- decisions are deterministic where required;
- reconciliation is repeatable;
- retries are safe;
- partial failure is survivable;
- the data plane can degrade intentionally;
- state ownership is clear;
- changes are attributable;
- recovery is possible from durable truth.

---

## Core principle

A control plane converts intent into controlled action.

A useful model is:

```text
                intent / desired state
                         |
                         v
                  CONTROL PLANE
             +-----------+-----------+
             |           |           |
          policy       state      orchestration
             |           |           |
             +-----------+-----------+
                         |
                         v
                 enforcement/action
                         |
                         v
                    DATA PLANE
                         |
                         v
                  observed reality
                         |
                         +-----------> control plane
```

The control plane should continuously answer:

> What should exist, what actually exists, what is the smallest safe action
> required to reduce the difference, and am I authorized to perform it?

---

## Architectural invariants

A control plane SHOULD satisfy these invariants unless there is a documented
reason not to.

### 1. Desired state is explicit

Intent should live in a durable, inspectable representation.

Examples:

- API resource;
- declarative configuration;
- database record;
- Git-backed desired state;
- policy object.

Do not hide desired state exclusively in transient workflow memory.

### 2. Observed state is distinct from desired state

The system must distinguish:

```text
what the user asked for
```

from:

```text
what currently exists
```

Do not overwrite desired state merely because reality drifted.

### 3. Reconciliation is idempotent

Repeated evaluation of the same desired and observed state should not create
uncontrolled duplicate effects.

Every external side effect should be safe under retry or guarded by stable
identity/idempotency.

### 4. Authority is narrower than capability

The control plane may technically be able to affect many resources.

Its effective authority should be scoped by:

- caller;
- tenant;
- resource;
- environment;
- action;
- policy;
- approval.

Do not let broad platform credentials silently become universal authorization.

### 5. The data plane does not trust control traffic implicitly

Authenticate and authorize control-plane actions where the architecture requires
it.

Do not assume internal networking makes control commands trustworthy.

### 6. Durable state survives control-plane restart

Correctness should not depend on process memory.

After restart, a fresh instance should be able to reconstruct enough state to
continue safely.

### 7. Partial progress is representable

Long-running operations may be only partly complete.

Persist enough information to resume, reconcile, or compensate.

Do not reduce every operation to "success/failure" when intermediate state
matters.

### 8. Control-plane failure is not automatically data-plane failure

Where possible, existing runtime/data-plane operation should continue when the
control plane is temporarily unavailable.

Do not place the control plane synchronously in every runtime request path
without a strong reason.

### 9. Destructive actions require stronger evidence

Deletion, replacement, revocation, global policy mutation, and similar actions
should require stronger ownership and authorization checks than reads or
reversible updates.

### 10. State has an authority

Every important state domain should have one defined source of truth.

Do not allow multiple writable control planes to manage the same state without
explicit conflict semantics.

### 11. Policy is enforced at a known boundary

A policy document is not a control until an enforcement point applies it.

### 12. Every consequential action is attributable

For important actions preserve:

- initiating actor;
- control-plane identity;
- policy decision;
- target;
- operation;
- result;
- observed effect.

---

## Control-plane boundary definition

Before implementation, identify:

- consumers;
- control-plane API;
- desired-state store;
- observed-state sources;
- policy engine or authorization logic;
- orchestration/reconciliation engine;
- data-plane endpoints;
- external systems;
- audit store;
- event bus if present;
- administrative surface;
- bootstrap dependencies.

A minimal architecture may look like:

```text
Client
  |
  v
Control API
  |
  +--> AuthN/AuthZ
  |
  +--> Desired State Store
  |
  +--> Reconciler / Orchestrator
          |
          +--> Data Plane API
          +--> External API
          +--> Policy
          +--> Status Store
  |
  +--> Audit / Telemetry
```

Keep the number of authoritative components small.

---

## Control plane versus data plane

Define the boundary explicitly.

### Control plane

Usually owns:

- intent;
- configuration;
- policy;
- orchestration;
- lifecycle;
- admission;
- promotion;
- metadata;
- status.

### Data plane

Usually owns:

- request execution;
- traffic;
- workload processing;
- data movement;
- model inference;
- storage I/O;
- message delivery.

The exact split varies.

Document it.

---

## Runtime independence

Ask:

> If the control plane disappears for fifteen minutes, what still works?

Potential answers:

- existing workloads continue;
- cached policy remains active;
- no new resources are created;
- configuration updates pause;
- runtime traffic continues;
- destructive operations fail closed.

This answer should be intentional.

---

## Synchronous versus asynchronous control

Prefer asynchronous control for operations that:

- take time;
- involve external systems;
- can partially succeed;
- need retries;
- depend on eventual consistency.

A typical pattern is:

```text
request
  ->
accepted
  ->
operation ID
  ->
background reconciliation
  ->
status
```

Avoid keeping client connections open for multi-minute provisioning when an
asynchronous lifecycle model is more appropriate.

---

## Desired-state model

Desired state should be:

- durable;
- versioned;
- authorized;
- inspectable;
- attributable.

Define whether desired state is:

- mutable object state;
- append-only intent;
- Git commit;
- event stream;
- transactional record.

Do not mix these models casually.

---

## Observed-state model

Observed state may come from:

- direct API reads;
- telemetry;
- caches;
- events;
- data-plane status;
- external provider state.

Know the freshness and consistency of each source.

Do not make destructive decisions on stale observations without verification.

---

## State ownership

For each resource/domain define:

- desired-state owner;
- observed-state authority;
- status writer;
- external system of record.

Example:

```text
desired deployment version -> control-plane database
actual replica state       -> runtime scheduler
artifact identity          -> registry
authorization policy       -> policy service
```

---

## Source-of-truth conflicts

If two systems can change the same state, define conflict semantics.

Possible models:

- control plane wins;
- external system wins;
- last write wins;
- conflict blocks convergence;
- explicit adoption required.

Do not let two controllers fight indefinitely.

---

## State versioning

Every mutable resource should support concurrency control.

Possible mechanisms:

- revision;
- generation;
- resource version;
- ETag;
- monotonic sequence;
- transaction version.

Avoid blind overwrite.

---

## Optimistic concurrency

Prefer optimistic concurrency for control-plane APIs where appropriate.

On conflict:

1. re-read;
2. recompute;
3. retry boundedly.

Do not overwrite another actor's change silently.

---

## Resource identity

Use stable immutable identity.

Human-readable names may change.

Avoid managing external resources solely by mutable display name.

---

## External identifiers

Persist stable provider/resource IDs where useful.

This supports:

- retries;
- drift detection;
- deletion;
- import/adoption;
- reconciliation.

---

## Spec and status

A useful resource model separates:

```text
spec   = desired configuration
status = observed condition
```

Status should not become a hidden input channel.

---

## Conditions

Represent important states independently.

Examples:

```text
Ready
Progressing
Degraded
PolicyDenied
DependencyUnavailable
DeletionBlocked
```

Conditions should explain why.

Avoid one opaque `phase` field for complex systems.

---

## Observed generation

Where desired state has a generation/revision, status should identify which
revision it reflects.

This prevents stale status from being interpreted as current.

---

## Reconciliation

Reconciliation should be small and repeatable.

A generic loop:

1. load desired state;
2. validate authority and lifecycle;
3. observe actual state;
4. compute delta;
5. perform one or more bounded actions;
6. record status;
7. emit audit/telemetry;
8. requeue only if required.

Avoid huge reconcile functions that perform entire workflows blindly.

---

## Minimal delta

Prefer the smallest safe action needed to converge.

Do not recreate entire resource graphs when one field changed.

This reduces:

- blast radius;
- downtime;
- retry risk;
- audit noise.

---

## Idempotency

Every mutation should have an idempotency story.

Possible techniques:

- stable resource ID;
- idempotency key;
- compare-before-write;
- create-if-absent;
- upsert;
- operation ID;
- conditional update.

---

## Duplicate invocation

Assume reconciliation may run:

- twice;
- concurrently;
- after timeout;
- after leader failover;
- after restart.

The result should remain safe.

---

## Unknown outcome

A timed-out write may have succeeded.

Represent:

```text
UNKNOWN_OUTCOME
```

or equivalent.

Before retrying:

- query actual state;
- reconcile;
- use operation ID;
- avoid duplicate create.

Do not assume timeout means no side effect occurred.

---

## Workflow orchestration

Some control-plane tasks are genuinely workflows.

Examples:

- certificate rotation;
- multi-stage migration;
- disaster-recovery failover;
- environment bootstrap.

Use an explicit durable state machine when sequence matters.

---

## Workflow state

Persist:

- current step;
- operation ID;
- completed steps;
- retry count;
- external IDs;
- failure reason.

Do not rely on in-memory task state.

---

## Reconciliation versus workflow

Prefer reconciliation when the system maintains continuing desired state.

Prefer workflow when the system executes a finite sequence.

Use both if required.

Do not disguise a brittle script as reconciliation.

---

## Compensation

For workflows without atomic rollback, define compensating actions.

Example:

```text
create A
create B
B fails
    ->
delete A if safe
```

Compensation is not always possible.

Document irreversible boundaries.

---

## Saga-like behavior

For distributed operations:

- track step completion;
- define compensation;
- preserve operation identity;
- expose partial state.

Do not imply global atomicity where none exists.

---

## Transactions

Use real transactions where state is within one transactional boundary.

Do not build distributed transaction machinery when eventual reconciliation is
safer and simpler.

---

## Policy architecture

Separate:

```text
policy definition
policy decision
policy enforcement
```

where useful.

A control plane is often a natural policy enforcement point.

---

## Admission

Before accepting desired state, validate:

- schema;
- identity;
- authorization;
- policy;
- quota;
- environmental constraints.

Reject impossible or forbidden intent early.

---

## Policy decision

A policy decision should have:

- subject;
- action;
- resource;
- context;
- decision;
- reason;
- policy version.

Do not base security-critical decisions solely on free-form model output.

---

## Policy enforcement

Enforcement may occur:

- at control API;
- before reconciliation;
- before each external action;
- at data plane;
- during admission.

For high-impact actions, re-check policy close to execution.

---

## Policy drift

Policy may change while long-running work is underway.

Define whether an operation:

- continues under original approval;
- revalidates;
- pauses;
- aborts.

Sensitive operations should not ignore revoked authority indefinitely.

---

## Authorization

Authorization should be deterministic.

Questions should include:

> May actor A, through control plane C, perform action X on resource R in
> environment E under policy P?

---

## Delegated authority

If the control plane acts on behalf of a user, preserve:

- user identity;
- control-plane identity;
- delegated scope.

Do not let a broadly privileged service account silently erase the initiating
user's authorization boundary.

---

## Service identity

Each control-plane component should have its own identity where practical.

Examples:

- API;
- reconciler;
- policy service;
- worker;
- external provider adapter.

Use least privilege.

---

## Administrative identity

Separate ordinary tenant operations from:

- global policy change;
- tenant creation;
- root trust;
- credential rotation;
- emergency actions.

---

## Break-glass

Emergency control-plane access should be:

- explicit;
- audited;
- time-limited;
- revocable.

Do not make break-glass the normal operating model.

---

## Approval

Some operations should require explicit human or external approval.

Examples:

- production promotion;
- destructive delete;
- credential revocation;
- tenant-wide policy change;
- regional failover.

Approval should bind to:

- exact action;
- exact target;
- revision/artifact;
- expiry.

---

## Approval invalidation

Invalidate stale approval when material parameters change.

Do not let approval for version A authorize version B.

---

## Control API

The control-plane API is a public contract for its consumers.

Define:

- resource model;
- error model;
- asynchronous semantics;
- versioning;
- pagination;
- idempotency;
- status.

---

## API transport

Use standard transport/protocols unless the domain requires custom behavior.

Examples:

- HTTP/JSON;
- gRPC;
- event/API combination.

Do not invent a custom protocol for ordinary CRUD/reconciliation needs.

---

## API errors

Expose machine-usable errors.

Examples:

```text
INVALID_ARGUMENT
UNAUTHORIZED
FORBIDDEN
CONFLICT
QUOTA_EXCEEDED
NOT_FOUND
OPERATION_IN_PROGRESS
DEPENDENCY_UNAVAILABLE
```

Do not force clients to parse log strings.

---

## Asynchronous operations

Return operation identity for long-running work.

An operation resource may expose:

- ID;
- target;
- status;
- progress;
- error;
- start/end;
- initiating actor.

---

## Cancellation

Define whether operations can be cancelled.

Cancellation must clarify:

- what has already happened;
- what can still be stopped;
- what compensation occurs;
- what state remains.

Do not promise cancellation of irreversible external effects.

---

## Retry semantics for clients

Client retries should be safe.

Use:

- idempotency keys;
- conditional requests;
- stable operation IDs.

Document which operations are retryable.

---

## Data-plane command channel

Control commands to the data plane should be authenticated and integrity
protected.

Examples:

- signed desired state;
- mTLS-authenticated API;
- authorized message channel;
- mutually authenticated RPC.

Do not accept unauthenticated control messages.

---

## Push versus pull

Control planes may use:

```text
push model:
control plane -> data plane

pull model:
data plane -> control plane / desired-state source
```

Choose based on:

- connectivity;
- trust;
- scale;
- failure behavior;
- firewall boundaries.

---

## Pull-based reconciliation

Pull models can reduce inbound management access to data planes.

Examples:

- GitOps agents;
- node agents;
- cluster reconcilers.

But pull does not remove the need for authorization or artifact verification.

---

## Push-based control

Push can simplify low-latency control.

Protect credentials and reachability.

Avoid one global admin credential for every data-plane target.

---

## Command ordering

If actions depend on order, encode it.

Do not assume distributed delivery preserves global order.

Use:

- per-resource sequence;
- version;
- generation;
- operation dependency.

---

## Replay protection

For sensitive command channels, prevent stale/replayed commands from causing
unwanted effects.

Possible controls:

- monotonic revision;
- expiry;
- nonce;
- operation ID;
- state preconditions.

---

## Drift detection

Periodically or eventfully compare desired and observed state.

Classify drift:

```text
expected
repairable
policy violation
external ownership
conflict
```

Do not repair everything blindly.

---

## Drift correction

For controller-owned state, correct drift automatically where safe.

For shared ownership, surface conflict rather than fighting.

---

## Adoption

If the control plane can adopt pre-existing resources:

- require explicit intent;
- verify ownership;
- store stable identity;
- avoid accidental takeover.

---

## Orphaning

Define whether resources can be detached from management.

Orphaning should preserve ownership semantics.

Do not keep silently reconciling after a resource has been intentionally
detached.

---

## Deletion

Deletion is a lifecycle event, not a CRUD afterthought.

Define:

- what is deleted;
- what is retained;
- order;
- ownership checks;
- finalization;
- timeout.

---

## Finalization

If external cleanup must occur before desired-state removal, use a durable
finalization state.

Finalization should be:

- idempotent;
- observable;
- bounded;
- recoverable.

---

## Stuck deletion

Provide operational handling for:

- lost credentials;
- unavailable provider;
- partially deleted resource;
- unknown ownership.

A stuck finalizer must be diagnosable.

---

## Destructive safety

Before destructive action verify:

- desired deletion still current;
- target identity;
- ownership;
- authorization;
- approval if needed;
- observed state freshness.

---

## Replacement

Some updates require resource replacement.

Define:

- create-before-delete;
- delete-before-create;
- migration/cutover;
- state transfer.

Expose disruptive behavior.

---

## External systems

Treat each external provider/system as an independent failure and trust domain.

For each integration define:

- identity;
- permissions;
- timeout;
- retry;
- rate limit;
- consistency;
- idempotency;
- error mapping.

---

## Provider adapters

Use provider adapters where real portability or multiple backends are required.

Do not create speculative abstractions that reduce useful provider capability.

---

## Eventual consistency

Expect external systems to be eventually consistent.

Model states such as:

```text
REQUESTED
PROVISIONING
READY
DELETING
FAILED
UNKNOWN
```

Do not treat temporary not-found immediately after create as proof of failure.

---

## Rate limits

Respect provider/API limits.

Use:

- bounded concurrency;
- backoff;
- jitter;
- caching;
- batching.

---

## Thundering herd

After restart or outage, many resources may reconcile simultaneously.

Use:

- queueing;
- jitter;
- rate limits;
- prioritization.

Do not stampede external APIs.

---

## Work queue

A queue should expose:

- depth;
- oldest item age;
- throughput;
- retry count;
- dead/stuck items where relevant.

Bound queue size or persistence.

---

## Queue durability

If losing queued work would violate correctness, persist it or ensure desired
state naturally retriggers reconciliation.

Do not rely on an in-memory queue as sole durable intent.

---

## Event-driven triggers

Use events to reduce latency.

But correctness should not depend entirely on receiving every event when
durable desired state exists.

Periodic reconciliation can repair missed notifications.

---

## Polling

Use polling only where needed.

Bound interval and cost.

Avoid high-frequency polling of large external inventories.

---

## Watch streams

If using watch streams:

- handle disconnect;
- resume from revision where supported;
- recover from compaction/gap;
- fall back to relist/reconcile.

Do not assume watches are perfect.

---

## Caches

Caches improve scalability but introduce staleness.

Know which decisions can tolerate stale reads.

For destructive/sensitive actions, verify against authoritative state where
needed.

---

## Cache invalidation

Use version/revision-aware caches.

Do not keep authorization or resource state indefinitely without invalidation.

---

## Leader election

Use leader election if only one active control loop may operate on shared
state.

Use established lease mechanisms.

Do not add singleton assumptions without failover.

---

## Active-active

If multiple active reconcilers are safe:

- use per-resource serialization or optimistic concurrency;
- ensure side effects are idempotent;
- avoid shared mutable in-memory coordination.

---

## Sharding

Shard when scale or blast-radius requires it.

Possible shard keys:

- tenant;
- region;
- resource hash;
- account;
- cluster.

Define ownership transfer.

---

## Shard assignment

Shard assignment should be stable and observable.

Avoid split-brain ownership.

---

## Multi-region control plane

For multi-region control planes define:

- authoritative region;
- active/active or active/passive;
- replication;
- conflict handling;
- failover;
- data residency.

Do not rely on "multi-region database" as the entire architecture.

---

## Consistency model

Be explicit about consistency.

Possible models:

- strong;
- read-after-write;
- eventual;
- per-resource linearizable;
- leader-based.

Consumers need to know when state becomes visible.

---

## Control-plane database

The state store is critical.

Design for:

- backup;
- restore;
- migration;
- concurrency;
- audit where needed;
- tenant isolation.

Do not hide all platform correctness inside opaque DB triggers.

---

## Event store

An append-only event log may be appropriate when:

- audit/history;
- replay;
- temporal reconstruction

are core requirements.

Do not adopt event sourcing by default.

---

## Event sourcing

If used, define:

- event schema versioning;
- snapshot strategy;
- replay;
- idempotent projection;
- data retention.

A historical log is not automatically a convenient operational store.

---

## State-machine persistence

For durable workflows, store explicit transitions.

Do not infer operation state solely from log text.

---

## Data migration

Control-plane state migrations must preserve active resources.

Prefer:

- backward-compatible schema change;
- dual-read/write where needed;
- staged cutover.

---

## API upgrade

During control-plane rollout, old and new versions may coexist.

Ensure:

- state compatibility;
- API compatibility;
- worker compatibility;
- message schema compatibility.

---

## Version skew

Define supported version skew between:

- API service;
- workers;
- agents;
- data plane;
- provider adapters.

Do not assume atomic upgrade of every component.

---

## Feature gates

Use feature gates for staged high-risk changes where helpful.

Retire stale gates.

---

## Protocol versioning

If control/data-plane protocol is custom, define:

- negotiation;
- supported range;
- downgrade behavior;
- unknown fields;
- capability discovery.

Avoid silent incompatible changes.

---

## Capability negotiation

Data planes may support different capabilities.

Do not send unsupported commands blindly.

Represent:

- supported version;
- feature set;
- limits.

---

## Backward compatibility

New control planes should generally manage older supported data-plane agents
during upgrade windows if architecture requires rolling upgrades.

---

## Forward compatibility

Older control-plane components may need to tolerate additive fields/messages.

Use explicit compatibility rules.

---

## Bootstrap

Document how the first control plane is created.

Questions:

- Where does root trust originate?
- Who creates initial identity?
- Where does initial state live?
- How are initial policies loaded?
- What dependencies already exist?

---

## Circular bootstrap dependencies

Avoid cycles such as:

```text
control plane requires identity platform
identity platform requires control plane
```

If unavoidable, define a minimal bootstrap mode and later transition.

---

## Root trust

Root keys, initial administrators, or root certificates require special
handling.

Keep them outside ordinary application credentials.

---

## Bootstrap credentials

Use temporary bootstrap credentials where possible.

Rotate or remove them after initialization.

---

## Recovery

A control plane should be recoverable from durable state.

Document:

```text
restore state
    ->
restore credentials/trust
    ->
start control plane
    ->
reconcile data plane
```

---

## Disaster recovery

Identify what must be recovered:

- desired state;
- policy;
- identities/credentials;
- operation state;
- audit;
- configuration.

Avoid backing up reconstructable caches unless recovery-time requirements
justify it.

---

## Reconciliation after restore

After restoring state, assume the data plane may have changed.

Re-observe before issuing mutations.

Do not replay old commands blindly.

---

## Split brain

Design against multiple control planes believing they own the same resource.

Use:

- leader lease;
- shard ownership;
- fencing token;
- generation/epoch.

---

## Fencing

For dangerous shared-resource operations, use fencing/epochs where stale leaders
could still act.

---

## Lease expiry

Treat lease expiry conservatively.

A new leader should not assume the old actor is incapable of side effects
unless the fencing mechanism guarantees it.

---

## Data-plane survivability

Where runtime continuity matters, data-plane components should cache enough
validated configuration to continue temporarily.

Define:

- cache duration;
- revocation behavior;
- fail-open/fail-closed.

---

## Cached policy

Caching authorization/policy decisions can improve availability but creates
revocation delay.

Make TTL and invalidation intentional.

---

## Fail-open versus fail-closed

Choose by control type.

Examples:

- revocation/security policy: often fail closed;
- telemetry reporting: often fail open;
- existing routing config: may continue from last known good.

Do not use one universal failure mode.

---

## Last-known-good state

For config-driven data planes, preserve validated last-known-good configuration.

Do not replace healthy config with malformed or unauthenticated updates.

---

## Configuration rollout

Config changes should have:

- version;
- validation;
- rollout;
- rollback.

Treat config as production behavior.

---

## Staged rollout

For high-risk control changes, support:

- canary tenant;
- subset of resources;
- region;
- percentage;
- feature gate.

Observe before global rollout.

---

## Rollback

Rollback should restore known-good:

- control-plane binary;
- config;
- policy;
- protocol compatibility;
- desired state where appropriate.

State schema changes may make binary rollback unsafe.

---

## Change safety

Before global policy or config change, estimate blast radius.

Require stronger review for broad changes.

---

## Multi-tenancy

Define tenant boundaries at:

- API;
- state store;
- queue;
- policy;
- credentials;
- data plane;
- audit.

Do not rely only on a `tenant_id` field in request payload.

---

## Tenant isolation

Use trusted identity to derive tenant.

Enforce it on every resource lookup/mutation.

Test cross-tenant denial.

---

## Tenant quotas

Control-plane quotas may bound:

- resources;
- concurrent operations;
- API rate;
- external spend;
- data-plane capacity.

Self-service must remain bounded.

---

## Administrative delegation

Allow tenant administrators only the scope they own.

Avoid global admin roles when tenant-scoped roles suffice.

---

## Resource hierarchy

If resources form hierarchy, define inheritance carefully.

Examples:

```text
organization
  -> project
     -> environment
        -> resource
```

Authorization and policy may inherit.

Document exceptions.

---

## Policy inheritance

Avoid surprising precedence.

Define whether child policy can:

- tighten;
- loosen;
- override;
- only extend.

---

## Quotas and reservations

Capacity may be reserved or shared.

Define fairness and priority.

Do not let control-plane queue order become accidental policy.

---

## Prioritization

Use explicit priority classes for:

- emergency/security operations;
- production recovery;
- normal provisioning;
- bulk backfill.

Do not let caller-controlled integer priority bypass policy.

---

## Backpressure

Reject or queue when overloaded.

Do not accept unlimited provisioning intents into unbounded memory.

---

## Overload behavior

Define:

- HTTP/API response;
- retry guidance;
- queue semantics;
- fairness.

---

## Capacity model

Measure meaningful units such as:

- managed resources;
- reconcile/sec;
- external API calls/sec;
- queue depth;
- state-store write rate;
- events/sec.

Do not size only by CPU.

---

## Horizontal scaling

Control-plane stateless API layers should scale independently where possible.

Workers/reconcilers may require shard or leader models.

---

## Vertical scaling

Use where stateful or single-leader components require it, but do not let
vertical scaling substitute for a real architecture at known scale.

---

## Hot tenants

Detect tenants/resources that generate disproportionate load.

Isolate or rate-limit them.

---

## Noisy resource

A single bad resource should not permanently starve the queue.

Use per-key backoff and fairness.

---

## Dead-lettering

If an operation becomes permanently unrecoverable, surface it explicitly.

Do not silently drop failed work.

---

## Terminal state

Distinguish terminal failure from retryable failure.

Require user/operator change to restart terminal operations.

---

## Error taxonomy

Use stable categories.

Example:

```text
INVALID_INTENT
AUTHENTICATION_FAILED
AUTHORIZATION_DENIED
POLICY_DENIED
CONFLICT
DEPENDENCY_UNAVAILABLE
RATE_LIMITED
TRANSIENT_PROVIDER_ERROR
TERMINAL_PROVIDER_ERROR
UNKNOWN_OUTCOME
OPERATION_TIMEOUT
```

---

## Error visibility

Expose useful status to consumers without leaking sensitive backend internals.

---

## Audit model

Audit important control actions.

Useful fields:

- actor;
- delegated actor;
- control-plane component;
- resource;
- action;
- desired revision;
- policy decision;
- approval;
- operation ID;
- result.

---

## Audit integrity

For high-assurance systems, the control plane should not be able to silently
erase evidence of its own privileged actions.

---

## Observability

Track:

- API request rate;
- API latency;
- authorization denials;
- policy denials;
- reconciliation duration;
- reconcile errors;
- queue depth;
- oldest work age;
- external API latency;
- retry rate;
- unknown outcomes;
- operation completion;
- data-plane connectivity.

---

## Convergence metrics

A powerful control-plane metric is:

```text
time from desired-state change
to observed convergence
```

Measure by operation/resource class.

---

## Drift metrics

Track:

- resources out of sync;
- duration out of sync;
- repeated drift;
- ownership conflicts.

---

## Status freshness

Expose how current observed state is.

Do not present stale state as real time.

---

## Logging

Use structured logs with:

- resource ID;
- operation ID;
- tenant;
- reconcile ID;
- component;
- error category.

Avoid secrets and full sensitive payloads.

---

## Tracing

Use tracing for multi-stage orchestration when it materially helps.

Correlate:

```text
API request
    ->
operation
    ->
queue
    ->
reconcile
    ->
provider call
```

---

## Events

Emit lifecycle events such as:

- accepted;
- policy denied;
- provisioning;
- ready;
- degraded;
- deleting;
- failed.

Do not treat event delivery as the only durable state.

---

## SLOs

Define SLOs around consumer-visible capabilities.

Examples:

- control API availability;
- provisioning convergence time;
- update convergence;
- deletion completion.

Do not invent numbers without requirements.

---

## Security architecture

Threat-model:

- malicious tenant;
- compromised client;
- compromised control-plane component;
- compromised worker;
- stolen provider credential;
- malicious desired state;
- stale leader;
- replayed control command.

---

## Malicious desired state

Treat desired-state input as untrusted.

Validate:

- fields;
- size;
- references;
- URLs;
- templates;
- scripts;
- expressions.

Do not allow declarative config to become arbitrary code execution accidentally.

---

## Hostile templates

If templates can generate downstream resources:

- constrain language;
- sandbox evaluation where needed;
- validate output;
- prevent secret access.

---

## External references

References to secrets, namespaces, accounts, or resources should be authorized.

Object existence does not imply reference permission.

---

## Cross-tenant references

Deny by default unless explicitly granted.

---

## SSRF

If the control plane connects to caller-supplied endpoints:

- validate schemes;
- restrict private/metadata networks where appropriate;
- enforce allowlists;
- limit redirects.

---

## Secret handling

Keep:

- provider tokens;
- cloud credentials;
- signing keys;
- root trust

out of resource status and logs.

Prefer short-lived identity.

---

## Credential brokering

Where possible, mint scoped credentials per operation or target.

Do not distribute global provider credentials to every worker.

---

## Worker isolation

Workers performing privileged changes should be isolated according to risk.

Potential boundaries:

- per provider;
- per tenant;
- per environment;
- per privilege class.

---

## Administrative API isolation

Place global admin operations behind stronger authentication and authorization.

Do not expose them through the same weak client path as ordinary resource CRUD.

---

## Input size limits

Bound:

- request size;
- resource count;
- batch operation size;
- template size.

Control APIs can be denial-of-service targets.

---

## Batch operations

For bulk actions:

- bound batch size;
- expose per-item result;
- prevent one failure from hiding others;
- support dry-run where valuable.

---

## Dry run / plan

For consequential changes, a plan may show:

- resources affected;
- creates;
- updates;
- deletes;
- policy denials.

A plan is not authorization.

Revalidate at execution.

---

## Admission and execution race

State may change between plan and action.

Use preconditions/revision checks.

Do not execute an approved plan against materially changed targets silently.

---

## Model/AI involvement

If AI assists control decisions:

- treat model output as untrusted;
- require deterministic policy;
- bound tool/action authority;
- preserve human approval for high-impact operations.

Compose with `agentic-system.md` and `ai-security.md`.

Do not let a probabilistic model become the root control-plane authority.

---

## Supply chain

The control plane is highly privileged.

Compose with `software-supply-chain.md`.

Protect:

- source;
- build;
- dependencies;
- images;
- plugins;
- policy bundles;
- migration tools.

---

## Artifact promotion

Promote immutable control-plane artifacts.

Do not rebuild differently per environment without need.

---

## Signing and verification

If artifacts are signed, verify before deployment/admission.

Signing without verification is incomplete.

---

## Upgrade strategy

Control-plane upgrades should consider:

- database schema;
- API version;
- workers;
- protocol;
- policy;
- in-flight operations.

---

## In-flight operation compatibility

A new version must know how to handle operations started by an older version,
or explicitly drain them first.

---

## Rolling upgrade

If old and new versions overlap:

- share compatible state;
- understand leader election;
- avoid duplicate side effects;
- support protocol overlap.

---

## Blue/green control plane

May be appropriate for major migrations.

Ensure only one authoritative writer exists unless active-active semantics are
explicit.

---

## Schema migration

Prefer expand/migrate/contract.

Example:

```text
add compatible field/table
    ->
deploy readers/writers
    ->
backfill/migrate
    ->
remove old field later
```

---

## Operation migration

If operation state schema changes, migrate or support both formats.

Do not strand in-flight workflows.

---

## API deprecation

Deprecate client-visible APIs with:

- replacement;
- migration guide;
- timeline;
- telemetry on remaining usage.

---

## Data-plane agent upgrades

If the control plane manages agents:

- define minimum supported version;
- capability negotiation;
- staged rollout;
- rollback.

Do not force simultaneous fleet-wide upgrade.

---

## Compatibility matrix

Maintain a compact compatibility statement for:

- control plane;
- agents;
- API versions;
- provider adapters.

---

## Testing strategy

### Unit tests

Test:

- desired-state calculations;
- policy logic;
- error classification;
- idempotency helpers;
- transition logic.

### State-store integration tests

Test:

- transactions;
- conflicts;
- migrations;
- recovery.

### Reconciliation tests

Test:

- create;
- steady state;
- update;
- drift;
- delete;
- duplicate invocation;
- partial failure.

### External integration tests

Test representative provider behavior.

### Security tests

Test:

- cross-tenant denial;
- privilege boundaries;
- admin separation;
- malicious resource input;
- replay/stale command.

### Upgrade tests

Test old state with new code.

### Disaster-recovery tests

Test restore and reconcile.

---

## Property-based testing

Useful properties include:

- applying reconciliation twice at steady state causes no new effect;
- a denied tenant can never mutate another tenant;
- desired state is not changed by observation;
- deletion never targets a resource without ownership evidence.

---

## Fault injection

Inject:

- provider timeout;
- state-store outage;
- lost queue message;
- stale cache;
- leader loss;
- partial create;
- partial delete;
- rate limit.

Observe convergence.

---

## Restart testing

Terminate the control plane mid-operation.

Restart it.

Confirm the system resumes safely.

This is a fundamental control-plane test.

---

## Unknown-outcome testing

Force timeout after external side effect.

Confirm reconciliation observes before retry.

---

## Split-brain testing

For systems with leaders/shards, test stale actor behavior.

Use fencing where necessary.

---

## Scale testing

Test realistic:

- resource count;
- tenants;
- update bursts;
- reconcile backlog;
- provider latency.

Do not test only empty systems.

---

## Soak testing

Long-running control planes may reveal:

- queue leaks;
- cache growth;
- retry churn;
- status write amplification;
- connection leaks.

Use where scale/risk warrants.

---

## End-to-end acceptance path

A useful reference path is:

```text
authenticate consumer
    ->
submit desired state
    ->
validate
    ->
authorize
    ->
admit
    ->
persist intent
    ->
reconcile
    ->
perform external action
    ->
observe actual state
    ->
publish status
    ->
consumer sees converged state
```

Also test:

```text
unauthorized intent
    ->
deny before external side effect
```

and:

```text
external partial failure
    ->
persist state
    ->
retry safely
    ->
converge
```

---

## Documentation

The repository should document:

- control-plane purpose;
- control/data-plane boundary;
- desired-state model;
- authoritative state;
- resource lifecycle;
- authorization;
- policy;
- retries;
- idempotency;
- deletion;
- failure behavior;
- upgrade;
- recovery.

---

## Architecture diagrams

Useful views include:

### Context

```text
Consumers -> Control Plane -> Data Planes / Providers
```

### State

```text
Desired State <-> Reconciler <-> Observed State
```

### Trust

```text
User -> API -> Policy -> Worker -> Provider
```

### Failure

```text
Control plane down
    |
    +--> data plane continues?
    +--> provisioning stops?
    +--> cached policy remains?
```

Avoid diagrams that hide state ownership.

---

## Architecture decisions

Use ADRs for consequential choices such as:

- reconciliation vs workflow;
- state store;
- active-active vs leader;
- push vs pull;
- sharding;
- policy architecture;
- operation model;
- protocol versioning;
- deletion semantics.

---

## Recommended repository shape

Follow existing repository conventions first.

A generic control-plane repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── api/
│   ├── resources/
│   └── operations/
├── src/
│   ├── api/
│   ├── auth/
│   ├── policy/
│   ├── reconcile/
│   ├── workflow/
│   ├── state/
│   ├── providers/
│   ├── queue/
│   └── audit/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── reconciliation/
│   ├── security/
│   ├── upgrade/
│   ├── failure/
│   └── e2e/
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── operations/
│   ├── migration/
│   └── adr/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

A control-plane repository should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-reconcile
test-policy
test-security
test-integration
test-upgrade
test-failure
test-e2e
verify
```

These names are illustrative.

Do not invent a command surface that the project cannot actually support.

---

## Acceptance criteria

A control plane is not complete because its API accepts requests.

Demonstrate the applicable subset of the following.

### State model

- desired and observed state are distinct;
- authoritative state is defined;
- state versions prevent blind overwrite;
- status reflects current desired revision.

### Reconciliation

- create converges;
- steady state is idempotent;
- drift is handled correctly;
- duplicate invocation is safe;
- restart mid-operation is safe.

### Side effects

- external operations have stable identity;
- unknown outcomes are reconciled;
- retries are bounded;
- destructive operations verify ownership.

### Authorization

- caller identity is established;
- tenant/resource scope is enforced;
- delegated authority is preserved;
- administrative privileges are separated.

### Policy

- policy has explicit enforcement points;
- denied requests cause no external side effect;
- policy changes have defined behavior for in-flight operations.

### Failure behavior

- dependency outage behavior is defined;
- data-plane survivability is defined;
- retry storms are controlled;
- queue/backlog is observable.

### Lifecycle

- asynchronous operation state is durable;
- cancellation semantics are explicit;
- deletion/finalization works;
- replacement/migration behavior is known.

### Scale

- concurrency is bounded;
- rate limiting exists where needed;
- hot tenants/resources cannot starve the platform;
- shard/leader behavior is understood.

### Upgrade and recovery

- old state can be read by new versions;
- in-flight operations survive upgrade;
- rollback limits are documented;
- restore followed by reconciliation is tested where required.

### Observability

- operation ID exists;
- reconciliation latency/errors are visible;
- policy/authz denials are visible;
- consequential actions are auditable.

---

## Optional composition

Common combinations:

```text
platform-architecture + control-plane
```

For defining a broader shared platform with explicit control-plane semantics.

```text
control-plane + operator-controller
```

For Kubernetes-native or controller-style reconciliation implementations.

```text
control-plane + zero-trust-service
```

For identity-aware control APIs, delegated authority, and explicit trust
boundaries.

```text
control-plane + software-supply-chain
```

For strong provenance of control-plane code, policies, plugins, and releases.

```text
control-plane + high-assurance
```

For deeper restart, split-brain, fault-injection, destructive-action, and
recovery verification.

```text
control-plane + ai-security
```

For AI-assisted control planes where model output is advisory and deterministic
policy remains authoritative.

---

## Anti-patterns

Avoid:

- control plane as synchronous dependency for every data-plane request;
- desired state stored only in memory;
- one giant imperative workflow pretending to be reconciliation;
- retries without idempotency;
- treating timeout as proof of failure;
- deleting resources by mutable name only;
- multiple writable sources of truth;
- global admin credentials used by every worker;
- user-supplied tenant ID as sole authorization boundary;
- stale cached state driving destructive actions;
- no distinction between transient and permanent errors;
- hot-looping failed resources;
- unbounded work queues;
- active-active writers with no conflict/fencing semantics;
- relying on events as the sole durable source of intent;
- policy defined but not enforced;
- approval not bound to exact target/revision;
- control-plane restart losing workflow progress;
- data plane failing because telemetry/control metadata is unavailable;
- declaring HA because there are multiple replicas;
- incompatible agent/data-plane upgrades requiring global flag day;
- API/schema changes that strand in-flight operations.

Do not mistake orchestration for control-plane correctness.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. control-plane purpose;
2. consumer classes;
3. control-plane/data-plane boundary;
4. desired-state model;
5. observed-state model;
6. authoritative state sources;
7. resource identity/versioning model;
8. reconciliation/workflow model;
9. idempotency/unknown-outcome strategy;
10. authorization/delegation model;
11. policy enforcement model;
12. deletion/finalization semantics;
13. queue/concurrency/rate-limit model;
14. leader/shard/fencing model;
15. failure/degraded behavior;
16. upgrade/recovery model;
17. deterministic/reconciliation/security tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a control plane is "highly available", "idempotent", "self-healing",
"secure", or "eventually consistent" merely because it uses queues,
controllers, replicas, or retries.

Describe the actual state authority, reconciliation semantics, failure
boundaries, authorization, and recovery evidence.

---

## Guiding principle

A good control plane should make intent durable and side effects boring.

Desired state should be explicit.

Observed state should be honest.

Authority should be narrow.

Policy should be enforceable.

Reconciliation should be repeatable.

Retries should be safe.

Unknown outcomes should trigger observation, not blind repetition.

The data plane should survive control-plane disruption where possible.

And after any restart, failover, partial operation, or external drift, the
control plane should be able to reconstruct reality and continue moving the
system toward the same intended state without inventing a new one.
