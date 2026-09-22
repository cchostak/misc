# operator-controller.md

> Recipe for scaffolding or elevating a Kubernetes operator or controller.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with profiles such as `zero-trust-service.md`,
> `software-supply-chain.md`, `kubernetes-workload.md`, `high-assurance.md`,
> `hostile-input.md`, or `ai-security.md` where appropriate.
>
> This recipe defines API, reconciliation, lifecycle, safety, RBAC,
> observability, upgrade, testing, and operational expectations for software
> that extends or automates Kubernetes through controllers and custom
> resources.
>
> It does not require a specific operator framework, SDK, programming language,
> admission stack, GitOps product, or Kubernetes distribution.

## Purpose

Use this recipe when the repository owns a reconciliation loop over Kubernetes
or a Kubernetes-style control plane.

Typical examples:

- Kubernetes operators;
- custom-resource controllers;
- infrastructure controllers;
- policy controllers;
- configuration controllers;
- lifecycle managers;
- external-system reconcilers;
- cluster automation controllers;
- workload orchestration controllers.

Do **not** use this recipe for an ordinary application merely because it runs
inside Kubernetes.

The goal is not to produce a controller-shaped binary.

The goal is to build a control loop that is:

- idempotent;
- convergent;
- observable;
- least-privileged;
- safe under retries;
- safe under partial failure;
- explicit about ownership;
- resilient to drift;
- compatible across upgrades;
- understandable from API intent to observed state.

---

## Core principle

A controller should continuously reconcile:

```text
desired state
    |
    v
observed state
    |
    v
reconcile
    |
    v
actions
    |
    v
new observed state
```

The controller should not "run a workflow once".

It should repeatedly ask:

> Given the desired state and the current observed state, what is the smallest
> safe action required to move reality toward intent?

Reconciliation must remain correct when:

- invoked repeatedly;
- interrupted midway;
- invoked after external drift;
- invoked after partial success;
- invoked against stale cache state;
- invoked concurrently with users or other controllers.

---

## Architectural invariants

An operator/controller SHOULD satisfy these invariants unless the repository has
a documented reason not to.

### 1. Reconciliation is idempotent

Running the same reconcile function repeatedly against the same desired and
observed state should not create uncontrolled duplicate effects.

Do not design reconciliation as an append-only script.

### 2. Desired state is declarative

The API should describe what the user wants, not a sequence of imperative
steps.

Prefer:

```yaml
spec:
  replicas: 3
  version: v1.4.2
```

over:

```yaml
spec:
  actions:
    - create
    - scale
    - deploy
```

unless the domain is inherently workflow-oriented.

### 3. Status reflects observation

`status` should represent observed state.

Do not let users specify status.

Do not treat status as a hidden configuration channel.

### 4. Spec and status are distinct contracts

`spec` expresses intent.

`status` reports progress and reality.

Use conditions, observed generation, and explicit state where appropriate.

### 5. Reconciliation is restart-safe

The controller must tolerate restart at any point.

Do not rely on in-memory workflow state for correctness.

Durable state should live in:

- the Kubernetes API;
- external systems;
- explicit durable storage

as justified.

### 6. External side effects are safe under retry

If reconciliation calls cloud, SaaS, database, or infrastructure APIs:

- use idempotency;
- detect existing state;
- reconcile by identity;
- avoid blind recreate.

### 7. Ownership is explicit

Know which resources and fields the controller owns.

Do not mutate unrelated fields or resources merely because the controller can.

### 8. Finalization is deliberate

If external resources must be cleaned up, use finalizers intentionally.

Do not create resources that can never be deleted because the finalizer path is
fragile.

### 9. RBAC is least-privileged

Grant only the verbs and resources actually required.

Do not default to `cluster-admin`.

### 10. Errors affect requeue behavior intentionally

Distinguish:

- transient failure;
- permanent invalid input;
- external dependency outage;
- conflict;
- rate limit;
- terminal condition.

Do not hot-loop on permanent errors.

### 11. API evolution is a compatibility problem

CRD changes can break:

- clients;
- persisted objects;
- upgrades;
- conversions;
- other controllers.

Treat API versioning as a public contract.

### 12. The controller is not the source of truth for everything

Kubernetes may own desired state while external systems own actual state.

Preserve that distinction.

Do not invent authoritative state in annotations or caches when a real system
of record exists.

---

## Controller boundary definition

Before implementation, identify:

- watched resources;
- owned resources;
- external systems;
- reconciliation triggers;
- cache/watch dependencies;
- finalizers;
- status fields;
- conditions;
- RBAC;
- leader election;
- webhook/admission dependencies;
- upgrade model.

A minimal architecture may look like:

```text
Custom Resource
    |
    v
Controller Watch
    |
    v
Reconcile
    |
    +--> Kubernetes resources
    |
    +--> External API
    |
    +--> Status / Conditions
```

Keep the control loop understandable.

---

## API design

The API is part of the product.

Before defining a CRD, ask whether an existing Kubernetes API already models
the requirement sufficiently.

Do not create a CRD merely to store arbitrary configuration.

A good API should be:

- declarative;
- small;
- composable;
- stable;
- versionable;
- understandable to users.

---

## Spec design

Fields in `spec` should represent desired state.

For each field define:

- type;
- required/optional;
- default;
- validation;
- mutability;
- meaning;
- lifecycle.

Avoid vague fields such as:

```yaml
config: {}
options: {}
extra: {}
```

unless extensibility truly requires them.

---

## Defaults

Use defaults carefully.

Defaults become API behavior.

Avoid defaults that:

- silently create expensive resources;
- grant broad privilege;
- expose services publicly;
- select production environments;
- enable destructive behavior.

Defaults should be safe and unsurprising.

---

## Validation

Use CRD schema validation where possible.

Validate:

- enum values;
- ranges;
- required fields;
- formats;
- cross-field constraints where supported.

Use admission validation when schema validation is insufficient.

Do not defer obviously invalid input until deep inside reconcile logic.

---

## Immutability

Some fields should be immutable after creation.

Examples may include:

- external account identity;
- storage class;
- tenancy boundary;
- resource ownership key.

If mutation is unsafe, reject it explicitly.

Do not pretend every field can be updated in place.

---

## API naming

Use clear Kubernetes-style naming.

Prefer domain terms over implementation terms.

Do not expose internal class names or vendor names in public API unless they
are intentionally part of the contract.

---

## API versions

Use API versions deliberately.

Typical progression may include:

```text
v1alpha1
v1beta1
v1
```

Do not promote to stable based only on time.

Stable means the API contract is intended to remain compatible.

---

## Conversion

If multiple API versions coexist, define conversion.

Prefer hub-and-spoke conversion where appropriate.

Test:

- round-trip conversion;
- defaulting;
- field preservation;
- lossy conversions.

Do not silently discard user intent during version conversion.

---

## Stored versions

Know which CRD version is used for persisted objects.

Upgrade plans must account for storage version migration where needed.

Do not remove a served version while persisted objects still depend on it.

---

## Status

Status should report meaningful observed state.

Potential fields:

- phase;
- conditions;
- observedGeneration;
- external resource identifiers;
- endpoints;
- last successful reconcile;
- version.

Avoid dumping large backend payloads into status.

---

## Conditions

Use conditions for important independently observable states.

Examples:

```text
Ready
Progressing
Degraded
DependenciesReady
```

A condition should include:

- type;
- status;
- reason;
- message;
- observed generation where appropriate.

Do not create dozens of noisy conditions.

---

## Observed generation

Use `observedGeneration` where it helps users know whether status reflects the
current spec.

This is especially important when reconciliation is asynchronous.

---

## Phases

Use a single phase field sparingly.

A phase can be useful for simple lifecycle summaries.

Do not encode the full state machine into one brittle enum when conditions
communicate state better.

---

## Reconciliation contract

A reconcile loop should conceptually:

1. fetch current object;
2. handle deletion/finalization;
3. validate or derive desired state;
4. observe owned/external state;
5. compute minimal delta;
6. apply changes;
7. update status;
8. requeue only when needed.

Keep side effects explicit.

---

## Idempotency

Each reconciliation action should answer:

> If this call succeeds but the controller crashes before recording success,
> what happens on the next reconcile?

Design for that case.

---

## Create-or-update behavior

Prefer reconciling by stable identity.

For managed resources:

- get existing;
- compare desired;
- patch/update only necessary fields;
- create if absent.

Avoid delete-and-recreate for ordinary updates unless lifecycle semantics
require replacement.

---

## Patch strategy

Use update/patch methods intentionally.

Consider:

- merge patch;
- strategic merge;
- server-side apply;
- field ownership.

Avoid overwriting fields owned by users or other controllers.

---

## Server-side apply

If using server-side apply:

- define field manager identity;
- understand conflict semantics;
- own only intended fields.

Do not force conflicts automatically unless policy justifies it.

---

## Resource ownership

Use owner references when lifecycle semantics match Kubernetes ownership.

Do not attach owner references across invalid namespace/scope boundaries.

Know when garbage collection will remove owned objects.

---

## External resources

For cloud/SaaS/external resources:

- store stable external identifiers;
- reconcile by observed state;
- tolerate eventual consistency;
- handle rate limits;
- avoid duplicate creation.

Do not assume API success means immediate convergence.

---

## Eventual consistency

External systems may lag.

Reconcile should tolerate:

- delayed visibility;
- asynchronous provisioning;
- transient not-found;
- intermediate states.

Do not classify every not-yet-visible resource as fatal.

---

## Finalizers

Use finalizers only when deletion requires controller-managed cleanup.

A finalizer should have:

- clear name;
- bounded cleanup logic;
- retry behavior;
- observability.

Do not block deletion for cleanup that can safely happen asynchronously without
the object.

---

## Finalizer safety

Finalization must handle:

- already-deleted external resource;
- partial cleanup;
- credential loss;
- external outage;
- permission loss.

Avoid immortal resources caused by unrecoverable finalizer state.

Provide documented break-glass removal procedure where appropriate.

---

## Deletion semantics

Define whether deleting the custom resource means:

- delete external resource;
- orphan external resource;
- retain data;
- revoke access;
- stop managing.

Make this explicit.

Do not surprise users by deleting durable external state unless the API clearly
says so.

---

## Drift detection

Controllers should detect drift between desired and observed state.

Decide which drift to:

- repair automatically;
- report;
- tolerate;
- preserve as user-managed.

Do not fight endlessly with another controller over the same field.

---

## Shared ownership

If multiple controllers manage one resource, define field boundaries.

Avoid overlapping ownership.

Controller fights often indicate a bad ownership model.

---

## Watches

Watch only resources that materially affect reconciliation.

Avoid watching broad cluster-wide resources unnecessarily.

Every watch affects:

- memory;
- API load;
- event volume.

---

## Predicates and event filtering

Filter irrelevant events where supported.

Avoid reconciling every object on unrelated metadata changes.

Do not over-filter and miss changes that affect desired state.

---

## Secondary resources

Watch owned/dependent resources when their changes should trigger reconcile.

Examples:

- Deployment;
- Secret;
- ConfigMap;
- Job;
- external resource proxy.

---

## Polling

Polling may be necessary for external systems that do not emit events.

Use bounded intervals.

Prefer adaptive/backoff behavior where useful.

Do not poll high-cost APIs aggressively without need.

---

## Requeue behavior

Requeue intentionally.

Possible patterns:

- immediate requeue for known next step;
- delayed requeue for eventual state;
- no requeue until event;
- exponential backoff after transient failure.

Do not continuously requeue successful steady-state objects.

---

## Error handling

Classify errors.

Examples:

```text
INVALID_SPEC
DEPENDENCY_UNAVAILABLE
RATE_LIMITED
CONFLICT
AUTHORIZATION_DENIED
TRANSIENT_EXTERNAL_ERROR
TERMINAL_EXTERNAL_ERROR
```

Map them to:

- condition;
- log level;
- requeue behavior;
- metrics.

---

## Permanent errors

If spec is invalid or action is impossible without user change:

- surface clear condition;
- avoid hot-looping;
- wait for spec/event change.

---

## Transient errors

For transient external failures:

- retry with backoff;
- preserve status;
- avoid duplicate side effects.

---

## Rate limits

Respect Kubernetes and external API rate limits.

Do not create thundering herds after restart.

Use client-side rate limiting where appropriate.

---

## Jitter

For periodic requeues, add jitter where useful.

This prevents all objects from reconciling simultaneously.

---

## Backoff

Use exponential backoff for repeated transient failures.

Reset backoff after successful progress.

---

## Work queues

Bound queue behavior.

Monitor:

- queue depth;
- reconcile latency;
- retries;
- work-item age.

Do not allow one pathological object to monopolize workers.

---

## Concurrency

Configure reconcile concurrency based on:

- API limits;
- external backend limits;
- object independence;
- memory/CPU.

Do not assume more workers always means faster convergence.

---

## Per-resource serialization

If concurrent reconcile of the same resource can cause races, ensure the
framework/work queue prevents it or add appropriate locking.

Avoid process-global locks when per-key serialization suffices.

---

## Optimistic concurrency

Kubernetes resources use resource versions.

Handle conflict errors by re-fetching and recomputing.

Do not blindly overwrite stale objects.

---

## Caches

Controller-runtime caches are eventually consistent.

Know when direct API reads are needed.

Do not assume a just-written object is instantly visible through every cache
path.

---

## Stale reads

Reconcile logic must tolerate stale observed state.

Avoid irreversible actions based solely on possibly stale cache data without
verification.

---

## Leader election

Use leader election when multiple replicas must not actively reconcile
simultaneously.

Prefer standard Lease-based mechanisms.

Do not add leader election when all replicas can safely reconcile independently.

---

## Leader failover

Test that a new leader can continue safely after the old leader disappears.

Correctness must not depend on in-memory state of the former leader.

---

## High availability

Running multiple replicas improves controller availability only when:

- leader election works;
- state is durable;
- external calls are retry-safe.

Do not call a controller HA merely because replicas = 3.

---

## RBAC

Generate or define RBAC from actual controller behavior.

Grant only required:

- apiGroups;
- resources;
- verbs;
- namespaces.

Avoid wildcard verbs/resources.

---

## Cluster scope

Prefer namespace scope if cluster-wide visibility is unnecessary.

Cluster-scoped controllers have larger blast radius.

Do not make a controller cluster-wide by default.

---

## Service account

Use a dedicated service account.

Do not run under a broad shared platform service account.

---

## External credentials

If the controller manages external systems:

- use workload identity where available;
- scope credentials narrowly;
- separate environments/accounts;
- rotate credentials.

Do not expose raw external credentials in status or logs.

---

## Impersonation

If Kubernetes impersonation is used, scope it carefully.

Do not let user-controlled values become arbitrary impersonation targets.

---

## Admission webhooks

Use mutating/validating webhooks only when needed.

Webhooks add:

- availability dependency;
- certificate lifecycle;
- upgrade complexity;
- failure-policy decisions.

Prefer CRD schema validation when sufficient.

---

## Mutating webhooks

Mutation should be deterministic and minimal.

Do not surprise users with large hidden configuration changes.

Document defaults introduced by mutation.

---

## Validating webhooks

Use for cross-field or dynamic validation not expressible in schema.

Avoid external network calls in admission unless absolutely necessary.

Admission is latency- and availability-sensitive.

---

## Webhook failure policy

Choose failure policy intentionally.

For security-critical validation, fail-closed may be appropriate.

For non-critical mutation, failure behavior may differ.

Do not choose `Ignore` or `Fail` by habit.

---

## Webhook certificates

Manage serving certificates deliberately.

Use platform-native certificate mechanisms where available.

Test renewal.

Do not ship static long-lived certificates in the repository.

---

## Conversion webhooks

Conversion webhooks are API-critical dependencies.

Test upgrades carefully.

A broken conversion webhook can prevent object reads/writes.

---

## CRD installation

CRDs should be applied before controllers that require them.

Manage ordering through deployment/bootstrap tooling.

Do not assume the controller can start correctly without required CRDs.

---

## CRD upgrades

CRD changes require care.

Validate:

- schema compatibility;
- served versions;
- storage versions;
- conversion;
- existing objects.

Do not replace CRDs blindly during upgrades.

---

## Defaulting changes

Changing defaults can alter behavior for newly created objects while existing
objects remain unchanged.

Document this.

Do not assume default changes apply retroactively.

---

## Status updates

Status updates should not rewrite spec.

Use status subresource where supported.

Avoid high-frequency status churn.

---

## Status noise

Do not update status on every reconcile if nothing materially changed.

Unnecessary writes increase API load and trigger more events.

---

## Conditions stability

Keep condition reasons/messages stable enough for automation and operators.

Do not put unbounded dynamic text into reason fields.

---

## Events

Emit Kubernetes Events for meaningful lifecycle transitions and failures.

Do not spam events on every reconcile loop.

Events are supplementary, not durable audit storage.

---

## Logging

Use structured logs.

Useful fields:

- controller;
- resource namespace/name;
- reconcile ID;
- generation;
- external resource ID;
- error category.

Do not log secrets.

---

## Metrics

Track controller-specific metrics.

Examples:

- reconcile count;
- reconcile duration;
- reconcile errors;
- queue depth;
- requeue count;
- external API latency;
- rate-limit events;
- finalization failures;
- resource convergence time.

Avoid high-cardinality labels from arbitrary resource names unless carefully
bounded.

---

## Tracing

Tracing may help for complex controllers and external API interactions.

Do not make tracing mandatory for simple controllers.

Do not include secrets in spans.

---

## Reconcile correlation

Use a reconcile/request identifier where helpful.

This aids diagnosis across:

- logs;
- external calls;
- status transitions.

---

## SLOs

Define SLOs only when real operational requirements exist.

Potential controller objectives:

- reconciliation latency;
- convergence time;
- successful reconcile rate.

Do not invent SLO numbers.

---

## Readiness

Controller readiness should reflect whether it can perform useful
reconciliation.

Potential dependencies:

- Kubernetes API reachable;
- required caches synced;
- critical config loaded.

Do not make readiness depend on every external managed system being healthy.

---

## Liveness

Liveness should indicate whether restart may help.

Do not restart the controller continuously because one external API is down.

---

## Startup

Wait for informer/cache synchronization before declaring readiness where the
framework requires it.

---

## Graceful shutdown

On shutdown:

- stop accepting new queue work;
- allow in-flight reconcile to finish within bounds;
- release leader lease cleanly;
- flush telemetry where practical.

---

## Multiple replicas

If using multiple replicas:

- configure leader election if required;
- verify readiness behavior;
- avoid every replica performing singleton background jobs unless intended.

---

## Background loops

Avoid hidden goroutines/background loops that bypass controller lifecycle.

If background processes exist, make them:

- cancellable;
- observable;
- restart-safe.

---

## Periodic maintenance

If the controller performs periodic cleanup or reconciliation:

- make it bounded;
- make it idempotent;
- make it leader-aware if necessary.

---

## External system rate limits

Controllers can overwhelm SaaS/cloud APIs.

Use:

- concurrency limits;
- caching;
- batching;
- rate limiters;
- backoff.

---

## External resource identity

Persist stable external IDs in status or annotations when appropriate.

Avoid looking up external resources by mutable names alone if stronger IDs
exist.

---

## Import/adoption

If the controller can adopt existing external resources:

- require explicit intent;
- verify ownership;
- avoid accidental takeover.

Do not silently start managing pre-existing resources because names match.

---

## Orphan handling

Define what happens when an external resource exists but the Kubernetes object
is missing.

Possible behaviors:

- orphan intentionally;
- garbage collect through separate policy;
- import;
- alert.

Do not delete automatically without ownership proof.

---

## Ownership markers

For external resources, consider tagging/labeling with controller ownership
metadata.

Use stable controller/resource IDs.

Do not rely solely on human-readable name matching.

---

## Drift ownership

Decide whether external manual changes are:

- reverted;
- preserved;
- surfaced as conflict.

Document the policy.

---

## Destructive reconciliation

Deletion/replacement actions require care.

Before destroying an external resource, confirm:

- ownership;
- target identity;
- desired deletion state;
- finalizer context;
- policy.

Do not let stale cache data trigger destruction.

---

## Safe replacement

Some changes require replacement.

Plan:

- create-before-delete;
- delete-before-create;
- cutover;
- data migration.

Expose disruptive behavior in status/events.

---

## Long-running operations

External provisioning may take minutes or hours.

Do not block one reconcile invocation for the entire operation if the external
API is asynchronous.

Start operation, persist identifier, requeue, observe.

---

## Async operation IDs

Store operation IDs where needed so retries can observe existing work rather
than start duplicates.

---

## Cancellation

Define whether changing/deleting spec cancels in-flight external operations.

Do not assume cancellation is always possible.

---

## Rollback

Controller rollback may involve:

- reverting controller image;
- reverting CRD/API;
- reconciling older desired state.

CRD/schema rollback can be harder than binary rollback.

Plan upgrades accordingly.

---

## Upgrade compatibility

During rolling controller upgrade, old and new controller versions may overlap
briefly.

Ensure:

- shared leader-election compatibility;
- CRD compatibility;
- status compatibility;
- external operation compatibility.

---

## Controller version skew

Avoid designs where two controller versions managing the same resource can
produce conflicting state.

---

## Feature gates

Use feature gates for risky staged behavior where useful.

Do not create a permanent maze of stale flags.

---

## Migration controllers

For complex API/data migrations, a dedicated migration process may be safer
than embedding all logic into normal reconcile.

Use only when warranted.

---

## Secrets management

If the controller reads secrets:

- request only needed Secret resources;
- avoid broad list/watch if unnecessary;
- never copy raw secrets into status;
- redact logs.

---

## Secret fan-out

Avoid copying one secret into many derived resources unless required.

This increases exposure and rotation burden.

---

## ConfigMaps

Treat ConfigMap content as untrusted configuration unless validated.

Do not execute scripts or commands from ConfigMaps automatically without trust
controls.

---

## Cross-namespace references

Cross-namespace references increase trust complexity.

If allowed:

- authorize them;
- validate scope;
- document semantics.

Do not let arbitrary namespaced users reference privileged resources elsewhere.

---

## Reference grants

Where using APIs that support explicit cross-namespace grants, require them.

Do not infer authorization from object existence.

---

## Multi-tenancy

For multi-tenant controllers:

- scope watches;
- enforce tenant boundaries;
- isolate credentials;
- isolate external resources;
- test cross-tenant references.

Do not trust tenant IDs embedded in spec without validating caller/resource
scope.

---

## Namespaced versus cluster-scoped APIs

Prefer namespaced custom resources when tenancy and lifecycle fit.

Use cluster-scoped resources only when the domain truly spans the cluster.

---

## Controller sharding

At large scale, shard by:

- namespace;
- resource hash;
- region;
- cluster;
- tenant.

Do not introduce sharding before scale demands it.

---

## API server pressure

Controllers can overload the Kubernetes API through excessive:

- list/watch;
- status updates;
- polling;
- full-object updates.

Measure and minimize unnecessary traffic.

---

## Informer indexes

Use cache indexes for common lookups when scale justifies it.

Avoid repeatedly scanning all objects inside reconcile.

---

## Garbage collection

Use Kubernetes garbage collection when ownership semantics match.

For external resources, implement explicit cleanup.

---

## Backups

The controller binary itself typically has little state.

Back up:

- CRDs/custom resources;
- external system state where needed;
- configuration;
- secrets according to platform policy.

---

## Disaster recovery

For critical operators, know how to recover if:

- controller deployment is lost;
- CRDs remain;
- cluster is restored;
- external resources continue to exist.

A new controller instance should converge from durable state.

---

## Local development

Provide a fast local workflow.

Possible approaches:

- envtest/fake API server;
- kind/k3d;
- real test cluster.

Use the lightest environment that validates the behavior under test.

---

## Unit tests

Test pure logic separately from Kubernetes integration.

Examples:

- desired-state calculation;
- diff logic;
- condition transitions;
- validation helpers;
- external request construction.

---

## Reconcile tests

Test reconciliation against representative state.

Cases should include:

- create;
- update;
- steady state;
- drift;
- deletion;
- partial external success;
- transient failure;
- permanent invalid spec.

---

## Envtest / API integration tests

Where the ecosystem supports an API-server test environment, test:

- CRD validation;
- status updates;
- finalizers;
- owner references;
- RBAC assumptions where possible;
- webhook behavior.

Do not rely only on mocked client behavior.

---

## Fake client caveat

Fake clients often do not reproduce:

- server-side defaulting;
- admission;
- resourceVersion conflicts;
- watch behavior;
- field ownership.

Use them for unit tests, not as the only integration evidence.

---

## End-to-end tests

Where practical, use a real disposable cluster.

Demonstrate:

```text
install CRD/controller
    ->
create custom resource
    ->
observe managed resource/external effect
    ->
observe status Ready
    ->
modify spec
    ->
observe convergence
    ->
delete
    ->
observe finalization
```

---

## Drift tests

Modify a managed child/external resource manually.

Confirm the controller:

- repairs;
- preserves;
- or reports

according to ownership policy.

---

## Retry tests

Inject transient failures.

Confirm:

- backoff;
- no duplication;
- eventual convergence.

---

## Finalizer tests

Test:

- normal deletion;
- already-missing external resource;
- transient cleanup failure;
- permanent cleanup failure behavior.

---

## Conflict tests

Simulate resourceVersion/update conflicts where possible.

Confirm reconcile retries safely.

---

## Leader-election tests

For HA controllers, verify only the intended active replica performs singleton
reconciliation/background work.

---

## Upgrade tests

For API/controller version changes, test:

- existing CRs;
- conversion;
- status preservation;
- rolling controller upgrade;
- rollback constraints.

---

## Fuzzing

Consider fuzzing:

- CRD conversion;
- parsers;
- external payload parsing;
- webhook request handling.

Crashes should become regressions.

---

## Property-based testing

Useful properties include:

- reconcile twice produces no additional effect at steady state;
- status does not change spec;
- invalid spec never creates external resources;
- cross-tenant references are always rejected.

---

## Negative-path tests

At least one meaningful failure path should be demonstrated.

Examples:

- invalid spec rejected;
- insufficient RBAC prevents action and surfaces clearly;
- external API outage causes bounded retry;
- stale approval/reference denied;
- finalizer cleanup failure remains visible;
- destructive action refused when ownership cannot be established.

---

## Security testing

Test:

- least-privilege RBAC;
- cross-namespace references;
- secret handling;
- webhook auth/TLS;
- external credential scope;
- malicious CR input.

Custom resources are untrusted input.

---

## Supply chain

Compose with `software-supply-chain.md`.

At minimum:

- pin controller dependencies;
- pin base image;
- use immutable image identity;
- scan/sign/verify where required.

Operators often hold broad privilege, so their own artifact integrity matters.

---

## Kubernetes workload hardening

The controller deployment itself should follow `kubernetes-workload.md`.

That includes:

- non-root where feasible;
- resource bounds;
- probes;
- graceful shutdown;
- service account;
- security context;
- immutable image.

---

## Documentation

The repository should document:

- custom resources;
- reconciliation behavior;
- ownership model;
- finalizer behavior;
- RBAC;
- external systems;
- conditions/status;
- upgrade path;
- failure modes.

Users should be able to answer:

> What will the controller create, update, delete, and own if I apply this
> object?

---

## API reference

Generate or maintain an API reference where practical.

Document:

- fields;
- defaults;
- validation;
- mutability;
- examples;
- status/conditions.

Do not leave users to infer API behavior from source structs alone.

---

## Architecture decisions

Use ADRs for consequential choices such as:

- CRD versus existing API;
- ownership semantics;
- finalization policy;
- server-side apply;
- external identity;
- conversion strategy;
- webhook adoption.

---

## Runbooks

For production operators, create runbooks for real operational failures.

Examples:

- controller crash loop;
- API rate limiting;
- stuck finalizers;
- external provider outage;
- webhook outage;
- reconciliation backlog;
- broken upgrade;
- RBAC regression.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic operator/controller repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── api/
│   └── <versioned API types>
├── controllers/
├── internal/
│   ├── reconcile/
│   ├── external/
│   ├── conditions/
│   └── policy/
├── config/
│   ├── crd/
│   ├── rbac/
│   ├── manager/
│   └── webhook/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── e2e/
│   ├── upgrade/
│   └── security/
├── docs/
│   ├── api/
│   ├── architecture/
│   └── adr/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

An operator/controller repository should expose obvious commands or equivalent
native interfaces for:

```text
check
test
test-integration
test-e2e
test-upgrade
test-security
generate
manifests
verify
run-local
```

These names are illustrative.

If code generation is required, generated output should be reproducible and
checked for drift.

---

## Acceptance criteria

An operator/controller is not complete because it compiles and registers a
watch.

Demonstrate the applicable subset of the following.

### API

- CRD schema is valid;
- invalid objects are rejected;
- defaults are intentional;
- status/conditions are meaningful;
- API versioning strategy is documented.

### Reconciliation

- create converges;
- update converges;
- steady state is idempotent;
- drift behavior is correct;
- retries do not duplicate side effects.

### Deletion

- finalization works;
- external cleanup is idempotent;
- missing external resource is handled;
- stuck cleanup is observable.

### Ownership

- child resources are owned correctly;
- field ownership is explicit;
- external ownership can be established before destructive action.

### Security

- RBAC is least-privileged;
- controller service account is dedicated;
- secrets are not exposed in status/logs;
- cross-namespace/tenant references are constrained;
- malicious CR input is rejected or safely handled.

### Reliability

- transient external failures back off;
- permanent invalid spec does not hot-loop;
- queue/reconcile metrics exist;
- restart preserves correctness;
- leader failover works where used.

### Upgrade

- existing resources survive controller upgrade;
- CRD changes are compatible;
- conversion works where applicable;
- rollback limits are documented.

### Operations

- readiness/liveness semantics work;
- reconcile errors are observable;
- stuck finalizers can be diagnosed;
- API/backend rate-limit behavior is understood.

---

## Optional composition

Common combinations:

```text
operator-controller + kubernetes-workload
```

For hardening the controller's own Deployment and runtime behavior.

```text
operator-controller + zero-trust-service
```

For external API identity, scoped credentials, and stronger trust boundaries.

```text
operator-controller + software-supply-chain
```

For signed controller images, provenance, admission verification, and trusted
release artifacts.

```text
operator-controller + hostile-input
```

For controllers processing complex untrusted specs, files, templates, or
external payloads.

```text
operator-controller + high-assurance
```

For deeper fault injection, upgrade testing, independent verification, and
destructive-action controls.

```text
operator-controller + ai-security
```

For controllers that manage AI models, agents, prompts, or tool capabilities
where model-derived configuration is untrusted.

---

## Anti-patterns

Avoid:

- CRDs used as arbitrary config storage;
- imperative workflow scripts disguised as reconciliation;
- non-idempotent reconcile loops;
- cluster-admin by default;
- one giant controller owning unrelated domains;
- status used as hidden desired state;
- hot loops on permanent errors;
- finalizers with no recovery path;
- external resources identified only by mutable names;
- deleting external resources without ownership proof;
- status updated on every reconcile with no change;
- broad cluster-wide watches without need;
- fake-client-only test coverage;
- assuming cache reads are immediately consistent;
- assuming API success means external convergence;
- webhooks for validation that CRD schema could handle;
- CRD upgrades with no stored-version plan;
- overlapping field ownership with another controller;
- in-memory workflow state required for correctness;
- controller restarts that lose progress;
- mutable controller image tags in production;
- calling `replicas: 3` high availability without leader/failover correctness.

Do not mistake a reconcile loop for a robust control plane.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. controller purpose;
2. watched and owned resources;
3. API group/version/kind strategy;
4. spec/status/condition model;
5. reconciliation/idempotency model;
6. external-system ownership model;
7. finalizer/deletion semantics;
8. field/resource ownership strategy;
9. RBAC/service-account model;
10. leader-election/concurrency model;
11. retry/backoff/error model;
12. webhook/admission model where present;
13. CRD upgrade/conversion strategy;
14. observability signals;
15. unit/reconcile tests executed;
16. integration/e2e/upgrade tests executed;
17. security/negative-path tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a controller is "production-ready", "safe", "idempotent", or
"highly available" merely because reconciliation succeeds once.

Describe the repeated-reconcile, failure, deletion, ownership, upgrade, and
security behavior that was actually exercised.

---

## Guiding principle

A good controller should make convergence boring.

Desired state should be explicit.

Observed state should be honest.

Reconciliation should be repeatable.

Retries should be safe.

Ownership should be narrow.

Deletion should be intentional.

External effects should be attributable.

Upgrades should preserve the API contract.

And if the controller dies halfway through an operation, starting it again
should move the system toward the same desired result rather than create a new
problem.
