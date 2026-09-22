# eventing-platform.md

> Recipe for scaffolding or elevating a shared eventing and messaging platform.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md`, `control-plane.md`,
> `developer-platform.md`, `api-platform.md`, `identity-platform.md`,
> `secure-data-pipeline.md`, `secure-service.md`, `library-sdk.md`,
> `operator-controller.md`, and profiles such as `zero-trust-service.md`,
> `software-supply-chain.md`, `high-assurance.md`, or `hostile-input.md` where
> appropriate.
>
> This recipe defines architecture expectations for platforms that publish,
> route, retain, deliver, replay, govern, secure, observe, and evolve events,
> commands, messages, streams, and queues across multiple producers and
> consumers.
>
> It does not require Kafka, Pulsar, NATS, RabbitMQ, SNS/SQS, EventBridge,
> Service Bus, Pub/Sub, a specific schema registry, a particular cloud, or a
> specific streaming framework.

## Purpose

Use this recipe when the platform provides shared asynchronous communication or
event-streaming capability.

Typical examples:

- enterprise eventing platforms;
- event buses;
- streaming platforms;
- queueing platforms;
- integration messaging platforms;
- domain-event platforms;
- cloud-event platforms;
- change-data-capture platforms;
- event-driven application platforms;
- workflow/event backbones;
- security-event platforms;
- telemetry event backbones;
- AI/agent event fabrics.

The goal is not to centralize every message in one broker.

The goal is to create a coherent eventing platform where teams can answer:

- Is this an event, command, or work item?
- Who owns the topic/stream/queue?
- What is the event contract?
- Who may publish?
- Who may consume?
- What ordering is guaranteed?
- What delivery semantics actually exist?
- How long is data retained?
- Can it be replayed?
- What happens to poison messages?
- What is the schema compatibility policy?
- How is sensitive data protected?
- How are duplicate deliveries handled?
- How is backpressure managed?
- How is a failed consumer recovered?
- How is historical replay distinguished from live traffic?

---

## Core principle

An eventing platform should make asynchronous contracts explicit.

A useful model is:

```text
Producer
   |
   v
Publish Contract
   |
   v
Eventing Platform
   |
   +--> authentication
   +--> authorization
   +--> routing
   +--> partitioning
   +--> retention
   +--> delivery
   +--> replay
   +--> schema
   +--> observability
   |
   v
Consumer
```

The platform provides transport and shared guarantees.

Producers and consumers still own domain semantics.

And:

> "Exactly once" is an end-to-end claim, not a broker feature.

---

## Architectural invariants

An eventing platform SHOULD satisfy these invariants unless there is a
documented reason not to.

### 1. Message type is explicit

Distinguish:

```text
event
command
work item
notification
stream record
```

Do not call every asynchronous message an event.

### 2. Ownership is explicit

Every topic, stream, queue, schema, or event family should have an owner.

Ownership includes:

- contract;
- lifecycle;
- retention;
- security;
- compatibility;
- support.

### 3. Event identity is stable

Every event should have a stable identity where deduplication, replay, audit, or
causal tracing matters.

### 4. Delivery semantics are stated precisely

Use terms such as:

- at-most-once;
- at-least-once;
- effectively-once under stated constraints.

Do not claim exactly-once unless the full producer → transport → consumer →
side-effect path supports it.

### 5. Consumers tolerate duplicate delivery where required

At-least-once delivery implies idempotent or deduplicating consumers.

Do not push duplicate-handling responsibility into undocumented folklore.

### 6. Ordering guarantees are scoped

Ordering may be:

- none;
- per partition;
- per key;
- per producer;
- global.

Do not imply global ordering from local partition ordering.

### 7. Schemas are contracts

Event payloads are public interfaces between independently deployed systems.

Treat them as versioned and compatibility-sensitive.

### 8. Replay is intentional

If events are replayable, consumers must be safe under replay.

If they are not replayable, say so.

### 9. Backpressure is designed

A slow or failed consumer must not destabilize producers or the platform.

### 10. Sensitive data is classified

Retention, replay, replication, and observability can multiply data exposure.

Do not treat messaging as a neutral pipe.

### 11. Failures remain visible

Dead letters, retries, poison records, and delivery failures need owners and
operational paths.

### 12. Messaging infrastructure is not business semantics

The platform may route and enforce shared policy.

It should not become a hidden business workflow engine by accident.

---

## Message taxonomy

Define message categories.

### Event

An event is a statement that something happened.

Example:

```text
OrderPlaced
UserRegistered
DeploymentPromoted
```

Events should generally use past-tense semantic naming.

### Command

A command asks a specific capability to perform an action.

Example:

```text
CreateInvoice
RebuildIndex
RotateCredential
```

Commands usually have an intended handler.

### Work item

A work item represents queued processing.

Example:

```text
RenderDocumentJob
ScanRepositoryJob
```

It may have operational retry/dead-letter semantics distinct from domain events.

### Notification

A notification informs interested consumers but may not represent durable
business fact.

### Stream record

A stream record may represent ordered state transitions, telemetry, or data
changes.

Be explicit which model applies.

---

## Event versus command

A useful distinction:

```text
event:
  "this happened"

command:
  "please do this"
```

Do not publish commands disguised as events merely to appear loosely coupled.

---

## Domain events

Domain events should represent meaningful domain facts.

Avoid emitting low-level database implementation changes as public domain events
unless that is the intended contract.

---

## Integration events

Integration events may be derived from internal domain state and shaped for
external consumers.

This can decouple internal models from public event contracts.

---

## Change data capture

CDC streams represent data-store changes.

They are not automatically stable business contracts.

If consumers depend on CDC, define:

- schema ownership;
- retention;
- tombstones;
- ordering;
- snapshot behavior.

---

## Topic / stream / queue ownership

Every logical channel should have:

- owner;
- purpose;
- producer set;
- consumer class;
- schema;
- retention;
- security policy;
- lifecycle state.

Avoid anonymous shared topics.

---

## Naming

Use predictable naming.

Avoid encoding every implementation detail into topic names.

Possible dimensions:

- domain;
- event family;
- environment;
- version where needed.

Do not rely on names alone for authorization.

---

## Topic lifecycle

A useful lifecycle may be:

```text
experimental
supported
deprecated
retired
```

Do not leave abandoned topics forever.

---

## Dynamic topic creation

If self-service topic creation is allowed:

- authorize it;
- apply quotas;
- require ownership;
- enforce naming/policy.

Do not allow unlimited anonymous topic creation.

---

## Event envelope

A common event envelope may include:

```text
event_id
event_type
source
subject
time
schema_version
correlation_id
causation_id
tenant
trace context
payload
```

Include only fields with clear semantics.

---

## Event ID

Use globally or contextually unique identity.

The event ID should remain stable during retries/redelivery.

Do not mint a new event ID every time the same event is retried.

---

## Correlation ID

Use correlation to group related work.

It is not event identity.

Do not confuse correlation ID with deduplication key.

---

## Causation ID

Causation can identify the specific event/command that triggered another event.

This is useful for tracing asynchronous chains.

---

## Trace context

Propagate distributed tracing context where appropriate.

Do not require traceability to depend on broker-specific metadata only.

---

## Source identity

Identify who/what produced the message.

This may include:

- service;
- workload identity;
- source system;
- environment.

Do not trust a self-declared `source` field as authentication by itself.

---

## Tenant context

Tenant context should derive from trusted producer identity or authorized
routing.

Do not trust arbitrary tenant IDs inside payloads as the only isolation
boundary.

---

## Timestamp

Use clear event time semantics.

Distinguish:

- event occurrence time;
- publish time;
- broker ingest time;
- processing time.

Do not conflate them.

---

## Schema version

Include or associate an explicit schema version where needed.

Do not force consumers to infer payload shape from topic name alone.

---

## CloudEvents-style envelopes

Standard envelopes may improve interoperability.

Use open standards where they fit.

Do not adopt a standard envelope while leaving domain semantics undefined.

---

## Schema registry

A shared schema registry may provide:

- schema storage;
- compatibility checks;
- ownership;
- discoverability.

It is useful when many producers/consumers evolve independently.

Do not make the registry the only place humans can discover event semantics.

---

## Schema formats

Common formats include:

- JSON Schema;
- Avro;
- Protobuf;
- CloudEvents metadata plus payload schema.

Choose based on:

- ecosystem;
- evolution;
- tooling;
- performance;
- interoperability.

Do not select schema format by trend.

---

## Compatibility policy

Define supported compatibility mode.

Examples:

```text
backward
forward
full
none
```

Be precise about the operational meaning.

---

## Breaking schema changes

Potentially breaking changes include:

- deleting field;
- changing type;
- changing semantic meaning;
- reusing enum values;
- changing default interpretation;
- changing keying/partition semantics.

---

## Additive changes

Adding optional fields is often compatible.

But consumers using strict decoding may still break.

Test actual consumer behavior.

---

## Required fields

Adding required fields is usually dangerous.

Prefer optional/additive evolution unless coordinated migration exists.

---

## Field semantics

Do not repurpose an existing field.

If meaning changes materially, introduce a new field/version.

---

## Enum evolution

Adding enum values can break exhaustive consumer handling.

Document unknown-value behavior.

---

## Schema governance

Platform governance should enforce:

- ownership;
- compatibility;
- security classification;
- basic naming.

Avoid central committees reviewing every domain detail.

---

## Schema validation

Validate producer messages before publication where practical.

Do not accept malformed records and push parsing failure downstream silently.

---

## Producer contract

A producer should define:

- event types;
- schema;
- key/partition semantics;
- delivery expectation;
- ownership;
- retention requirements.

---

## Consumer contract

A consumer should define:

- subscription;
- replay behavior;
- retry behavior;
- idempotency;
- ordering assumptions;
- failure handling.

---

## Consumer groups

Consumer groups may provide load sharing.

Define:

- membership;
- partition assignment;
- rebalancing;
- offset semantics.

Do not assume consumer-group behavior is identical across technologies.

---

## Delivery semantics

State actual semantics.

### At-most-once

Potential message loss, no duplicates.

### At-least-once

Potential duplicates, eventual delivery while retention/retry permits.

### Exactly-once

Only claim if end-to-end observable side effects satisfy it under stated
constraints.

Do not use broker marketing terminology as architecture evidence.

---

## Effectively-once

A practical pattern is:

```text
at-least-once delivery
    +
idempotent consumer
    +
stable event ID
    +
transactional/conditional side effect
```

Describe constraints precisely.

---

## Duplicate delivery

Assume duplicates unless the full architecture proves otherwise.

Consumers may deduplicate using:

- event ID;
- idempotency key;
- domain key plus version.

---

## Deduplication store

If deduplication requires state:

- bound retention;
- handle restart;
- avoid unbounded growth.

Do not keep infinite event IDs forever.

---

## Ordering

Define exact guarantee.

Examples:

```text
no order
per partition
per key
per producer
global
```

Consumers should depend only on documented scope.

---

## Partition key

Choose key based on domain ordering and load distribution.

Bad keys can create hot partitions.

Do not choose tenant ID automatically if one tenant dominates traffic.

---

## Hot partitions

Detect and mitigate skew.

Options:

- better key;
- subpartitioning;
- separate high-volume tenant;
- rebalancing.

---

## Global ordering

Global ordering reduces scalability and availability.

Use only where truly required.

---

## Event time ordering

Events may arrive late or out of order.

Consumers should handle this if the domain allows.

---

## Late data

For stream processing, define lateness/window policy.

Do not silently discard late events without documented behavior.

---

## Retention

Define retention per stream/topic.

Retention depends on:

- replay need;
- legal/privacy constraints;
- recovery window;
- cost.

Do not use one retention period everywhere.

---

## Retention versus deletion

Retention expiration and explicit data deletion are different.

Sensitive data may require deletion before retention naturally expires.

---

## Compaction

Log compaction may preserve latest value per key.

Define tombstone behavior.

Do not confuse compaction with complete deletion.

---

## Tombstones

If tombstones represent delete:

- document schema;
- preserve ordering;
- ensure consumers process them.

---

## Replay

Replay is a first-class capability if supported.

Define:

- who may replay;
- time range;
- destination;
- live/replay interaction;
- side-effect safety.

---

## Replay identity

Mark replayed events where consumers need to distinguish them.

Do not mutate original event identity merely because it is replayed.

---

## Live plus replay

If replay and live traffic overlap, define merge/ordering semantics.

Avoid duplicate side effects.

---

## Replay authorization

Historical data may be more sensitive than live data.

Do not let every consumer replay arbitrary tenant history.

---

## Backfill

Backfill may republish:

- historical events;
- repaired records;
- newly derived events.

Mark provenance and operation identity.

---

## Reprocessing

Reprocessing should be safe.

Consumers need:

- idempotency;
- checkpoint control;
- side-effect isolation.

---

## Snapshot plus stream

For state reconstruction, a common model is:

```text
snapshot
    +
changes after snapshot position
```

Define handoff exactly.

---

## Consumer offsets

Offsets/checkpoints are consumer state.

Persist them durably when loss would cause incorrect replay or skip.

---

## Offset reset

Resetting offsets is a powerful operation.

Require explicit authorization and audit where consequential.

---

## Poison messages

A poison message repeatedly fails processing.

Consumers/platform should prevent one record from blocking progress forever.

---

## Dead-letter queues

Use DLQs when they enable real recovery.

Every DLQ should have:

- owner;
- retention;
- alert;
- replay procedure.

Do not create ownerless message graveyards.

---

## Retry topics/queues

Retry channels may separate delayed retries from live traffic.

Keep event identity and attempt metadata.

---

## Retry policy

Classify failures:

```text
transient
permanent
poison input
authorization
dependency outage
unknown outcome
```

Do not retry permanent schema or policy errors endlessly.

---

## Backoff

Use exponential backoff with jitter where appropriate.

Avoid synchronized retry storms.

---

## Max attempts

Bound retries.

On exhaustion:

- dead-letter;
- quarantine;
- alert;
- terminal state.

Do not retry forever.

---

## Delayed retry

Use delayed retry where immediate repeat would only amplify failure.

---

## Unknown outcome

A consumer may time out after an external side effect.

Before retrying:

- reconcile external state;
- use idempotency;
- preserve operation ID.

Do not assume timeout means no side effect.

---

## Transactional outbox

Use an outbox when a service must atomically commit domain state and publish
event intent.

Pattern:

```text
business transaction
    +
outbox row
    ->
outbox publisher
    ->
event platform
```

This avoids the classic dual-write gap.

---

## Inbox pattern

Consumers may use an inbox/deduplication record to make handling retry-safe.

Use when side-effect correctness justifies it.

---

## CDC outbox

CDC can publish outbox rows efficiently.

Keep domain event schema separate from raw database log format.

---

## Dual writes

Avoid:

```text
write database
publish event
```

as two independent uncoordinated operations when both must succeed together.

---

## Exactly-once caveat

Even if transport supports transactional consume/produce, external side effects
such as:

- email;
- HTTP calls;
- third-party writes

may still break exactly-once semantics.

State the actual boundary.

---

## Commands

Commands should usually have an intended receiver or capability.

Avoid broadcast commands unless semantics truly allow multiple handlers.

---

## Command idempotency

Commands that may redeliver should have operation identity.

Handlers should not create duplicate destructive effects.

---

## Command expiry

Some commands become unsafe/stale.

Support expiry/deadline where appropriate.

---

## Command cancellation

If commands can be cancelled, define the race between execution and
cancellation.

Do not imply guaranteed rollback after execution begins.

---

## Work queues

Work queues are operational constructs.

Define:

- visibility timeout;
- retry;
- ack;
- lease;
- poison handling;
- concurrency.

---

## Ack semantics

Acknowledge only after the intended durable processing point.

Do not ack before side effects are safely complete unless duplicate work is
acceptable.

---

## Visibility timeout

For leased work queues, timeout should exceed normal processing or support lease
extension.

Avoid premature redelivery.

---

## Long-running jobs

For long work:

- heartbeat/renew lease;
- checkpoint;
- persist operation state.

Do not hold invisible work indefinitely.

---

## Producer identity

Authenticate producers.

Authorization should constrain:

- which topics;
- which tenant;
- which event types;
- publish rate.

---

## Consumer identity

Authenticate consumers.

Authorization should constrain:

- subscriptions;
- replay;
- tenant scope;
- sensitive streams.

---

## Broker authentication

Use established:

- mTLS;
- OAuth/OIDC;
- workload identity;
- platform-native IAM.

Do not distribute broad shared passwords.

---

## Authorization

Grant only required:

- publish;
- consume;
- manage;
- replay;
- create topic.

Separate data-plane use from administrative control.

---

## Admin plane

Topic creation, retention changes, ACL changes, and global policy should use a
stronger admin path.

Do not let ordinary producers mutate platform-wide config.

---

## Workload identity

Prefer short-lived workload identity over static broker credentials.

Compose with `identity-platform.md`.

---

## Delegation

If a platform publishes on behalf of a user or tenant, preserve initiating
identity and tenant scope.

---

## Multi-tenancy

Define tenant isolation across:

- topics;
- partitions;
- credentials;
- quotas;
- logs;
- schema registry;
- replay.

Do not rely only on topic-name prefixes.

---

## Shared versus dedicated infrastructure

Decide when tenants share:

- brokers;
- clusters;
- namespaces;
- schema infrastructure.

Use dedicated isolation only where risk/scale justifies it.

---

## Quotas

Bound:

- publish rate;
- consume rate;
- storage;
- partitions;
- connections;
- replay load.

Self-service messaging should not mean unlimited infrastructure.

---

## Noisy neighbors

Detect tenants/consumers causing:

- hot partitions;
- excessive lag;
- connection storms;
- storage pressure.

---

## Backpressure

Backpressure is unavoidable in asynchronous systems.

Define where it appears:

- producer throttling;
- broker queue growth;
- consumer lag;
- rejected writes.

Do not hide overload until disks fill.

---

## Producer backpressure

Producers should handle:

- quota exceeded;
- broker unavailable;
- buffer full;
- timeout.

Avoid unbounded local buffering.

---

## Local producer buffers

If clients buffer locally:

- bound memory/disk;
- define loss behavior;
- expose metrics.

---

## Consumer lag

Lag is a first-class signal.

Track:

- messages;
- bytes;
- time behind head.

Use time-based lag where record rate varies.

---

## Lag SLOs

Some consumers may have freshness SLOs.

Do not assign one lag target to all consumers.

---

## Broker capacity

Model:

- messages/sec;
- bytes/sec;
- partitions;
- connections;
- storage;
- retention.

Do not size only by CPU.

---

## Partition count

Partitioning affects:

- parallelism;
- ordering;
- metadata load;
- cost.

Do not create huge partition counts by default.

---

## Broker scaling

Scale according to actual broker semantics.

Understand:

- rebalancing;
- replication;
- storage movement;
- controller metadata.

---

## Storage pressure

Define behavior as storage fills.

Possible actions:

- reject publishes;
- delete expired data;
- throttle;
- expand capacity.

Do not allow silent data loss.

---

## Replication

Replication improves availability but is not backup.

Define:

- replication factor;
- failure domains;
- durability expectations.

---

## Cross-region replication

Use when required for:

- DR;
- locality;
- global consumers.

Define:

- lag;
- conflict;
- active/active semantics;
- data residency.

---

## Active/active producers

If multiple regions publish same logical stream, define ordering and identity.

Do not imply a single global order.

---

## Failover

Define what happens if a region/broker cluster fails.

Consumers/producers should know:

- reconnect path;
- duplicate risk;
- data-loss window.

---

## Disaster recovery

Recover:

- broker metadata;
- schemas;
- ACLs;
- desired topic config;
- retained data where required.

Do not assume replication equals DR.

---

## Backup

Some messaging data may not need backup if producers can replay from source.

Document reconstruction strategy.

---

## Data durability

State actual durability guarantees.

Avoid vague "durable messaging" claims without failure assumptions.

---

## Acknowledgement level

Producer acknowledgements should map to durability requirements.

Do not choose weakest ack for critical business events solely for throughput.

---

## Producer retry

Producer retries must preserve event ID.

Do not create a new event identity for each transport attempt.

---

## Producer ordering

Retries can reorder messages.

Use idempotent/ordered producer features where required.

---

## Batching

Batching improves throughput.

Bound:

- batch size;
- wait time;
- memory.

Understand latency tradeoffs.

---

## Compression

Compression reduces bandwidth/storage.

Consider:

- CPU;
- payload sensitivity;
- decompression bombs for untrusted content.

---

## Serialization

Choose stable serialization.

Validate inputs.

Do not use unsafe object deserialization formats that can execute code.

---

## Unsafe deserialization

Never deserialize untrusted arbitrary language objects.

Prefer data-only formats.

---

## Payload size

Bound event/message size.

Large payloads can destabilize brokers.

For large objects, use:

```text
event with object reference
```

where appropriate.

---

## Claim check pattern

Store large payload elsewhere and publish a reference.

Then secure both:

- event;
- referenced object.

Do not publish long-lived public URLs for sensitive data.

---

## Sensitive data

Classify event fields.

Avoid propagating:

- secrets;
- tokens;
- full credentials;
- unnecessary personal data.

---

## Data minimization

An event should contain what consumers need.

Do not turn event streams into copies of entire source databases without
purpose.

---

## Encryption

Use transport encryption.

Use field/payload encryption where threat model requires.

Understand how encryption affects:

- routing;
- schema validation;
- observability.

---

## Key management

If payloads are encrypted:

- define key ownership;
- rotation;
- tenant isolation;
- access.

---

## Retention privacy

Long retention multiplies privacy risk.

Align retention with:

- purpose;
- legal requirements;
- recovery needs.

---

## Deletion requests

If data-subject or contractual deletion applies, define how deletion propagates
through retained logs, compacted topics, archives, and derived stores.

Event logs can complicate deletion.

Do not ignore this.

---

## Data residency

Regional replication may violate residency constraints.

Make routing and replication policy explicit.

---

## Schema privacy

Schema registries can leak domain metadata.

Restrict sensitive schema visibility where needed.

---

## Metadata leakage

Topic names and headers can reveal sensitive information.

Avoid embedding secrets or customer names unnecessarily.

---

## Observability

Track platform signals:

- publish rate;
- publish errors;
- broker latency;
- consumer lag;
- storage use;
- retry volume;
- DLQ rate;
- rebalance rate;
- replication lag;
- auth denials.

---

## Event tracing

Trace asynchronous flows using:

- trace context;
- correlation ID;
- causation ID.

Do not require a single request-span lifetime for long asynchronous workflows.

---

## Logs

Structured logs should include:

- topic/stream;
- event ID;
- producer identity;
- consumer group;
- error class.

Do not log full sensitive payloads by default.

---

## Metrics cardinality

Avoid per-event IDs as metric labels.

Use logs/traces for high-cardinality correlation.

---

## Audit

Audit:

- topic/schema creation;
- ACL changes;
- replay;
- retention changes;
- admin config;
- sensitive subscription changes.

---

## Replay audit

Replay can cause substantial side effects.

Record:

- actor;
- source range;
- destination;
- filters;
- operation ID.

---

## DLQ observability

A growing DLQ is a production signal.

Do not let failed events disappear from dashboards.

---

## Consumer health

A consumer process can be healthy while lag grows indefinitely.

Monitor business processing freshness.

---

## Broker health versus delivery health

Separate:

```text
broker up
```

from:

```text
events reaching consumers within expected time
```

---

## SLOs

Potential eventing SLOs:

- publish availability;
- end-to-end delivery latency;
- consumer freshness;
- durability.

Do not invent one global latency SLO for all topics.

---

## Event latency

Measure:

```text
event occurrence time
to
successful consumer processing time
```

when domain freshness matters.

---

## Schema compatibility metrics

Track rejected schema changes.

Do not treat high rejection count as inherently bad; it may mean governance is
working.

---

## Portal/catalog integration

Expose:

- topics;
- owners;
- schemas;
- consumers;
- lifecycle;
- retention.

Do not rely on portal metadata if broker/config state is authoritative.

---

## Self-service topic creation

A self-service flow may be:

```text
request channel
    ->
validate ownership
    ->
apply policy
    ->
create
    ->
publish schema
    ->
issue access
    ->
observe
```

---

## Self-service consumers

Allow teams to subscribe within policy.

Avoid manual ticketing for routine low-risk subscriptions.

---

## Approval

Sensitive topics may require owner/data-owner approval.

Bind approval to:

- consumer identity;
- topic;
- environment;
- scope;
- retention/replay permission.

---

## Access review

Review privileged subscriptions periodically where required.

---

## API integration

Eventing and synchronous API platforms should coexist.

Use APIs for:

- immediate request/response;
- commands requiring direct feedback.

Use events for:

- asynchronous facts;
- decoupled reactions;
- replayable state changes.

Do not replace every API call with events.

---

## Request-reply messaging

Messaging can support request/reply, but understand:

- correlation;
- timeout;
- reply routing;
- duplicate replies.

Do not use a broker to emulate RPC without a reason.

---

## Event-carried state transfer

Events may carry enough state for consumers to avoid synchronous lookups.

Tradeoff:

- less runtime coupling;
- larger/staler data copies.

Use deliberately.

---

## Event notification

A small event may simply notify consumers to fetch authoritative data.

Tradeoff:

- more runtime coupling;
- fresher source data.

---

## Choreography

Event choreography allows autonomous reactions.

Good for loosely coupled domain events.

Risks:

- implicit workflows;
- difficult global reasoning;
- hidden cycles.

---

## Orchestration

Use explicit orchestration when:

- sequence matters;
- compensation matters;
- end-to-end status matters.

Do not force long business workflows into invisible event choreography.

---

## Event cycles

Detect loops such as:

```text
A emits X
B reacts and emits Y
A reacts and emits X again
```

Use causation and state guards.

---

## Storm prevention

Protect against amplification.

Examples:

- one event causing thousands of fan-out events;
- retry storms;
- recursive automation.

Use quotas and bounded fan-out.

---

## Fan-out

Understand expected number of consumers.

High fan-out affects:

- broker load;
- storage;
- downstream capacity.

---

## Broadcast

Broadcast events can be appropriate.

Do not use broadcast for commands that should have one owner.

---

## Stream processing

If platform provides stream processing:

- version jobs;
- checkpoint state;
- manage schema evolution;
- define replay/backfill.

Do not combine broker and processing platform responsibilities accidentally.

---

## Stateful stream processing

State stores need:

- checkpoint;
- restore;
- migration;
- scaling semantics.

---

## Windowing

Define:

- event time;
- processing time;
- watermark;
- late data.

---

## Materialized views

Derived views should preserve lineage to source streams.

Do not treat them as independent truth without provenance.

---

## CDC platform integration

CDC should define:

- source DB;
- table/schema ownership;
- snapshot;
- log position;
- delete/tombstone;
- sensitive fields.

---

## CDC security

Database log access is highly privileged.

Use least privilege and isolate connectors.

---

## Connector platform

Connectors to external systems expand trust.

For each connector define:

- credential;
- scope;
- network access;
- retry;
- schema mapping.

---

## Connector sandboxing

Third-party connectors may be high-risk.

Use isolation where appropriate.

---

## Plugin trust

If broker/platform plugins are supported:

- control publisher;
- pin version;
- test compatibility;
- limit privileges.

Do not load arbitrary unsigned plugins dynamically.

---

## Supply chain

Compose with `software-supply-chain.md`.

Protect:

- broker images;
- client libraries;
- schema tooling;
- connectors;
- plugins;
- deployment config.

---

## Client libraries

Provide thin client libraries only if they improve consistency.

Useful capabilities:

- envelope;
- tracing;
- schema validation;
- auth;
- retry defaults.

Do not hide broker semantics so completely that consumers cannot reason about
delivery.

---

## Language SDKs

Compose with `library-sdk.md`.

SDKs should expose:

- ack behavior;
- retry;
- timeout;
- consumer lifecycle.

---

## CLI

A platform CLI may support:

- topic discovery;
- ACL requests;
- replay;
- lag inspection;
- schema publish.

Compose with `cli.md`.

High-impact replay/delete commands should require explicit intent.

---

## Management API

The management plane may provide:

- topic lifecycle;
- schema lifecycle;
- access;
- quotas;
- retention.

Compose with `control-plane.md`.

---

## Declarative management

A declarative API can make event infrastructure reproducible.

Example:

```yaml
stream:
  owner: payments
  retention: 7d
  partitions: 12
  classification: confidential
  producers:
    - payment-service
```

Keep provider-specific knobs behind advanced escape hatches where practical.

---

## Reconciliation

If the platform reconciles desired messaging resources:

- stable identity;
- idempotent create/update;
- safe delete;
- external drift detection.

Compose with `operator-controller.md` if Kubernetes-native.

---

## Deletion

Deleting a topic/stream can destroy durable history.

Use strong safeguards:

- ownership;
- explicit intent;
- retention policy;
- confirmation/approval;
- backup/export where required.

---

## Topic rename

A rename is often create + migrate + retire.

Do not assume broker-level rename exists or preserves consumers.

---

## Partition changes

Changing partition count can alter key ordering/distribution.

Treat as compatibility-sensitive.

---

## Retention changes

Reducing retention may irreversibly delete data.

Require stronger review/approval where important.

---

## Replay tooling

Replay tooling should support:

- dry run/plan;
- filters;
- destination;
- rate limits;
- audit.

Do not replay directly into production side-effecting consumers without
safeguards.

---

## Replay sandbox

For validation, replay into isolated consumer groups/topics where practical.

---

## Consumer pause

Provide safe pause/resume semantics.

Understand retention implications.

---

## Lag recovery

If consumer falls far behind:

- scale;
- optimize;
- skip with approval;
- snapshot/reseed.

Do not silently reset offsets.

---

## Schema migration

For incompatible changes, common strategies:

- new versioned event type;
- dual publish;
- compatibility adapter;
- consumer migration then retirement.

Avoid indefinitely publishing every historical version.

---

## Dual publish

If dual publishing:

- preserve correlation;
- monitor consumers;
- define end date.

---

## Consumer migration

Track consumers before retiring schema/topic.

Do not depend only on documentation announcements.

---

## Publisher migration

When changing producers, avoid duplicate event emission unless consumers can
handle it.

---

## Broker migration

A platform may need to migrate broker technologies.

Protect consumer contract.

Possible strategy:

```text
dual publish
    ->
mirror
    ->
consumer cutover
    ->
producer cutover
    ->
retire old
```

Do not promise seamless migration without measuring ordering/duplication
differences.

---

## Cross-broker mirroring

Mirroring may alter:

- ordering;
- offsets;
- timestamps;
- delivery duplicates.

Treat it as a new failure domain.

---

## Disaster migration

Emergency failover may sacrifice some guarantees.

Document acceptable data-loss/replay window.

---

## Environment separation

Separate:

- development;
- test;
- staging;
- production

through credentials, namespaces/clusters/accounts, or equivalent.

Do not let development consumers read production event streams by default.

---

## Test data

Avoid copying production sensitive events into non-production casually.

Use synthetic or minimized fixtures.

---

## Local development

Provide simple local development options where useful.

Examples:

- in-memory test broker;
- containerized lightweight broker;
- fake client;
- integration test environment.

Do not require a full enterprise cluster for unit tests.

---

## Test doubles

Fakes should model:

- duplicate delivery;
- ordering limits;
- ack/nack;
- retry

when those semantics matter.

Do not create a fake that makes messaging look more reliable than production.

---

## Testing strategy

### Unit tests

Test:

- envelope;
- schema validation;
- retry classification;
- idempotency logic;
- partition key logic.

### Contract tests

Validate schemas and compatibility.

### Integration tests

Use real or representative broker behavior.

### Consumer tests

Test duplicates, out-of-order, replay, poison messages.

### Producer tests

Test retry, backpressure, auth failure, schema rejection.

### Security tests

Test topic ACLs, tenant isolation, replay authorization.

### Failure tests

Test broker loss, partition loss, slow consumer, storage pressure.

### Migration tests

Test schema/topic/broker transitions.

---

## Duplicate-delivery tests

Deliver the same event twice.

Consumer side effects should match documented semantics.

---

## Out-of-order tests

Deliver events in unexpected order where platform does not guarantee order.

Consumers should handle or explicitly reject.

---

## Replay tests

Replay historical records.

Confirm consumers do not create unintended duplicate side effects.

---

## Poison-message tests

Inject malformed or permanently failing input.

Confirm:

- bounded retries;
- DLQ/quarantine;
- progress continues.

---

## Backpressure tests

Slow consumers/producers intentionally.

Observe:

- queue growth;
- throttling;
- memory/disk bounds.

---

## Retention tests

Verify records expire according to policy.

Do not claim retention based only on config text.

---

## Deletion/tombstone tests

Verify deletes propagate according to contract.

---

## Schema compatibility tests

Reject incompatible changes automatically where supported.

Review semantic compatibility separately.

---

## Authorization tests

Verify:

- unauthorized producer denied;
- unauthorized consumer denied;
- cross-tenant read denied;
- replay permission separate from live consume where required.

---

## Failover tests

For HA/DR architectures, fail nodes/regions.

Measure:

- data loss;
- duplicate delivery;
- recovery time;
- client behavior.

---

## Property-based testing

Useful properties:

- duplicate event never causes duplicate durable side effect where idempotency
  is promised;
- unauthorized tenant never reads another tenant's event;
- event ID remains stable across retries;
- schema decoder never crashes on malformed input.

---

## Fuzzing

Consider fuzzing:

- event parsers;
- schema decoders;
- connector input;
- webhook/event bridges.

---

## Soak testing

Long-running tests can expose:

- consumer leaks;
- offset drift;
- storage growth;
- rebalancing churn;
- retry loops.

---

## End-to-end acceptance path

A useful reference flow:

```text
producer authenticates
    ->
publishes schema-valid event
    ->
platform authorizes
    ->
event stored
    ->
consumer receives
    ->
consumer processes idempotently
    ->
offset/checkpoint advances
    ->
telemetry shows end-to-end success
```

Negative path:

```text
unauthorized producer
    ->
publish denied
    ->
no event visible
```

Failure path:

```text
consumer dependency fails
    ->
bounded retry
    ->
DLQ/quarantine if terminal
    ->
alert
    ->
safe replay after remediation
```

Replay path:

```text
authorized operator requests replay
    ->
range validated
    ->
rate bounded
    ->
events replayed with preserved identity/provenance
    ->
consumer remains safe
```

---

## Documentation

The platform should document:

- message taxonomy;
- ownership;
- event envelope;
- schema policy;
- delivery semantics;
- ordering;
- retry;
- DLQ;
- replay;
- retention;
- security;
- quotas;
- migration.

---

## Event catalog

An event catalog may expose:

- event type;
- owner;
- schema;
- producers;
- consumers;
- retention;
- classification;
- lifecycle.

Do not make catalog metadata a substitute for actual platform state.

---

## Runbooks

Create runbooks for:

- broker unavailable;
- consumer lag;
- hot partition;
- schema incompatibility;
- DLQ growth;
- replay;
- storage pressure;
- regional failover.

---

## Architecture diagrams

Useful views include:

### Publish/consume

```text
Producer -> Broker/Stream -> Consumer
```

### Trust

```text
Producer Identity -> ACL -> Topic -> Consumer Identity
```

### Replay

```text
Retained Log -> Replay Controller -> Isolated/Live Consumer
```

### Failure

```text
Consumer failure
    ->
retry
    ->
DLQ
    ->
operator remediation
    ->
replay
```

---

## Architecture decisions

Use ADRs for consequential decisions such as:

- event vs command model;
- schema format;
- delivery semantics;
- ordering scope;
- partition strategy;
- replay;
- retention;
- multi-region topology;
- outbox pattern.

---

## Recommended repository shape

Follow existing repository conventions first.

A generic eventing-platform repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── architecture/
│   ├── event-model.md
│   ├── trust-boundaries.md
│   ├── delivery-semantics.md
│   └── decisions/
├── schemas/
├── contracts/
├── platform/
│   ├── broker/
│   ├── access/
│   ├── registry/
│   ├── replay/
│   ├── connectors/
│   └── observability/
├── sdk/
├── cli/
├── tests/
│   ├── contract/
│   ├── integration/
│   ├── security/
│   ├── failure/
│   ├── replay/
│   └── e2e/
├── docs/
│   ├── producers/
│   ├── consumers/
│   ├── migration/
│   └── runbooks/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

An eventing-platform repository should expose obvious commands or equivalent
native interfaces for:

```text
check
test
test-contract
test-schema
test-integration
test-security
test-replay
test-failure
test-e2e
verify
```

These names are illustrative.

Use only commands the repository can actually implement.

---

## Acceptance criteria

An eventing platform is not complete because a producer can publish and a
consumer can receive.

Demonstrate the applicable subset of the following.

### Contract

- message/event taxonomy is explicit;
- event envelope is stable;
- schemas are authoritative;
- ownership is defined;
- compatibility is enforced.

### Delivery

- actual delivery semantics are documented;
- duplicate behavior is tested;
- ordering guarantees are scoped;
- acknowledgement semantics are understood.

### Replay and recovery

- retention is defined;
- replay is authorized;
- replay is safe for consumers;
- DLQs have owners and recovery procedures;
- offset reset is controlled.

### Security

- producer and consumer identities are authenticated;
- topic/stream authorization is enforced;
- tenant isolation is tested;
- sensitive payloads are minimized/protected;
- administrative operations are separate.

### Resilience

- retries are bounded;
- poison messages do not block progress indefinitely;
- consumer lag is observable;
- backpressure behavior is tested;
- broker/dependency outages have documented behavior.

### Data integrity

- event identity survives retries;
- idempotent consumer behavior exists where required;
- outbox/inbox or equivalent patterns address dual-write risk where needed;
- deletion/tombstone semantics are defined.

### Operations

- publish/consume health is visible;
- storage/retention pressure is visible;
- hot partitions/noisy tenants can be identified;
- schema and ACL changes are auditable.

### Lifecycle

- topic/schema deprecation has migration path;
- consumer inventory supports retirement;
- broker/platform migration preserves declared contracts.

---

## Optional composition

Common combinations:

```text
platform-architecture + eventing-platform
```

For shared eventing capability architecture and ownership.

```text
eventing-platform + control-plane
```

For declarative topic/schema/access/replay lifecycle management.

```text
eventing-platform + identity-platform
```

For producer/consumer workload identity and scoped authorization.

```text
eventing-platform + secure-data-pipeline
```

For event ingestion, transformation, lineage, and downstream data processing.

```text
eventing-platform + software-supply-chain
```

For trusted broker/client/connector artifacts and reproducible platform changes.

```text
eventing-platform + high-assurance
```

For deeper duplicate, replay, failover, fault-injection, and durability testing.

```text
eventing-platform + hostile-input
```

For untrusted payloads, connectors, parser hardening, and poison-message defense.

---

## Anti-patterns

Avoid:

- calling every message an event;
- commands disguised as domain events;
- anonymous topic ownership;
- raw database CDC exposed as a stable business contract accidentally;
- "exactly once" claimed from broker configuration alone;
- consumers that are not duplicate-safe under at-least-once delivery;
- global ordering assumptions without proof;
- one retention period for everything;
- unbounded replay into production;
- ownerless DLQs;
- retry forever;
- poison messages blocking a partition indefinitely;
- silent offset reset;
- topic names used as the only tenant boundary;
- shared static broker credentials;
- payloads containing secrets;
- production event dumps copied to development;
- schema changes with no compatibility checks;
- event replay with new IDs that defeat deduplication;
- dual writes without outbox/reconciliation;
- unbounded producer buffers;
- one hot tenant exhausting a shared cluster;
- choreography used for workflows nobody can reason about;
- replication described as backup;
- broker uptime used as the only measure of event delivery health.

Do not mistake a message broker for an eventing architecture.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. eventing-platform purpose;
2. message taxonomy;
3. ownership model;
4. event envelope and identity model;
5. schema/compatibility model;
6. producer/consumer identity model;
7. authorization/tenant isolation model;
8. delivery semantics;
9. ordering/partitioning model;
10. retry/DLQ/poison-message model;
11. idempotency/deduplication model;
12. retention/replay model;
13. backpressure/capacity model;
14. observability/audit model;
15. multi-region/DR model where applicable;
16. contract/schema/replay tests executed;
17. failure/security/idempotency tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim an eventing platform is "exactly once", "durable", "ordered",
"replayable", "multi-tenant", or "highly available" merely because the chosen
broker exposes features with those names.

Describe the actual end-to-end semantics, failure assumptions, consumer
behavior, authorization boundaries, retention, replay, and evidence.

---

## Guiding principle

A good eventing platform should make asynchronous behavior explainable.

Producers should know what they publish.

Consumers should know what they may assume.

Events should have identity.

Schemas should evolve deliberately.

Delivery guarantees should be stated honestly.

Duplicates should be survivable.

Ordering should be scoped.

Retries should be bounded.

Poison messages should remain visible.

Replay should be safe and authorized.

Sensitive data should not become immortal by accident.

And every important asynchronous flow should be explainable as:

> This producer, under this identity and schema, published this event with this
> stable identity into this retention and ordering boundary; these consumers
> were authorized to receive it under these delivery semantics, and this is
> what happens when delivery, processing, replay, or recovery fails.
