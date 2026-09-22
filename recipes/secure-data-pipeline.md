# secure-data-pipeline.md

> Recipe for scaffolding or elevating a secure data pipeline.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with profiles such as `zero-trust-service.md`,
> `software-supply-chain.md`, `hostile-input.md`, `high-assurance.md`,
> `ai-security.md`, or `internet-facing.md` where appropriate.
>
> This recipe defines architectural, security, privacy, integrity,
> observability, lineage, reliability, and operational expectations for systems
> that ingest, transform, enrich, move, store, or publish data.
>
> It does not require a specific cloud, warehouse, lakehouse, stream processor,
> orchestrator, message broker, transformation framework, or storage engine.

## Purpose

Use this recipe when the repository primarily implements or owns a data path.

Typical examples:

- ETL/ELT pipelines;
- batch ingestion;
- streaming pipelines;
- event processing;
- data lake ingestion;
- warehouse transformations;
- CDC pipelines;
- telemetry pipelines;
- security-data pipelines;
- log pipelines;
- ML feature pipelines;
- document ingestion pipelines;
- compliance evidence pipelines;
- cross-system synchronization.

Do **not** automatically apply this recipe to a service merely because it
stores data.

The goal is not to maximize the number of data-platform components.

The goal is to produce a data pipeline where:

- source identity is explicit;
- data ownership is known;
- trust boundaries are visible;
- schemas are versioned;
- transformations are reproducible;
- sensitive data is minimized and protected;
- lineage is inspectable;
- bad data does not silently contaminate downstream systems;
- reprocessing is safe;
- failure and partial progress are understood;
- consumers can determine what data they received and why.

---

## Core principle

A data pipeline moves trust as well as bytes.

A useful mental model is:

```text
source
  |
  v
ingest
  |
  v
validate
  |
  v
classify / authorize
  |
  v
transform
  |
  v
store / publish
  |
  v
consumer
```

At every stage ask:

- Where did this data come from?
- Is the source trusted?
- Is the schema valid?
- Is the caller allowed to read or write it?
- Has the data been transformed?
- What sensitive fields exist?
- Can the transformation be replayed?
- Can the result be traced back to source?
- What happens when validation or processing fails?

---

## Architectural invariants

A secure data pipeline SHOULD satisfy these invariants unless the repository
has a documented reason not to.

### 1. Source identity is explicit

Every ingested record or batch should be attributable to a source.

Preserve enough metadata to identify:

- source system;
- source object/topic/file;
- source revision/version where applicable;
- ingestion time;
- event time where applicable;
- source tenant/security domain.

Do not merge unrelated source domains without retaining provenance.

### 2. Schema is a contract

Data shape must be explicit.

Use:

- versioned schema;
- validated fields;
- clear nullability;
- clear required/optional semantics;
- compatibility rules.

Do not treat arbitrary JSON blobs as a durable interface merely because they
parse.

### 3. Invalid data does not silently become trusted data

Malformed, incomplete, unauthorized, or policy-violating records must be:

- rejected;
- quarantined;
- dead-lettered;
- explicitly repaired;
- or otherwise handled visibly.

Do not silently coerce every bad record into a plausible shape.

### 4. Sensitive data is classified

Know which fields may contain:

- credentials;
- personal data;
- health data;
- financial data;
- secrets;
- security findings;
- regulated data;
- proprietary content.

Apply controls based on data class.

### 5. Transformations are attributable

A derived dataset should be traceable to:

- source;
- transformation code/version;
- configuration;
- execution/run identity;
- relevant input partitions.

### 6. Reprocessing is safe

A pipeline should define what happens when the same data is processed again.

Use idempotency, deduplication, versioned outputs, or explicit overwrite
semantics.

Do not assume retries are harmless.

### 7. Partial failure is explicit

For multi-stage pipelines, know which stages completed and which did not.

Do not report global success when only some partitions or records succeeded.

### 8. Access is least-privileged

Use separate identities for:

- read source;
- write staging;
- transform;
- publish;
- administer.

Do not use one broad data-admin credential for the entire pipeline.

### 9. Lineage is operational evidence

Lineage should help answer:

> Which source data and transformation produced this output?

Do not reduce lineage to a pretty graph with no reliable identifiers.

### 10. Deletion and retention propagate intentionally

If data must be deleted, expired, or revoked, define how that affects:

- raw data;
- transformed data;
- caches;
- indexes;
- aggregates;
- backups where feasible;
- downstream exports.

### 11. External data is untrusted

Treat partner feeds, uploaded files, events, logs, and public datasets as
untrusted until validated.

### 12. Security and correctness are linked

A pipeline that leaks one tenant's data into another tenant's output is both a
security failure and a data-correctness failure.

---

## Pipeline boundary definition

Before implementation, identify:

- source systems;
- ingestion method;
- schema registry or schema location;
- validation stage;
- transformation stages;
- intermediate storage;
- target stores;
- downstream consumers;
- orchestration;
- credentials;
- data classification;
- lineage metadata;
- failure handling;
- retention/deletion path.

A simple pipeline may look like:

```text
Source
  |
  v
Ingest
  |
  v
Raw / Landing
  |
  v
Validate / Quarantine
  |
  v
Transform / Enrich
  |
  v
Curated Store
  |
  v
Consumers
```

Not every pipeline needs every layer.

Use the smallest architecture that preserves correctness and security.

---

## Source inventory

Document every source.

For each source identify:

- owner;
- trust level;
- protocol;
- authentication;
- schema;
- expected volume;
- update frequency;
- sensitivity;
- retention;
- tenant scope;
- replay capability.

Do not ingest a source you cannot identify operationally.

---

## Trust classification

Classify source trust.

A simple model may include:

```text
authoritative
internal
partner
user-generated
external
untrusted
```

Trust classification may influence:

- validation strictness;
- quarantine;
- downstream eligibility;
- human review.

Do not let source metadata self-declare trust level.

---

## Ingestion

Ingestion should be intentional.

For each ingestion path define:

- source identity;
- authentication;
- authorization;
- expected schema;
- size limits;
- retry behavior;
- deduplication;
- checkpointing;
- observability.

Do not ingest unlimited or malformed payloads blindly.

---

## Batch ingestion

For files/batches, define:

- file naming/identity;
- checksum;
- expected partition;
- size;
- completeness;
- manifest if needed;
- duplicate handling.

Do not mark a batch complete merely because a file appeared.

Where completeness matters, use an explicit completion signal or manifest.

---

## Streaming ingestion

For streams, define:

- topic/stream identity;
- partitioning;
- ordering guarantees;
- delivery semantics;
- checkpointing;
- replay;
- consumer-group ownership.

Do not assume exactly-once semantics without verifying what the platform
actually guarantees.

---

## Delivery semantics

Be explicit about whether the pipeline is:

```text
at-most-once
at-least-once
effectively-once
exactly-once
```

Use the strongest accurate description.

Do not claim exactly-once merely because duplicates are rare.

---

## Event identity

Where deduplication matters, each event should have a stable identity.

Possible fields:

- event ID;
- source offset;
- sequence;
- content hash;
- business key.

Do not rely only on processing timestamp.

---

## Ordering

Define whether event order matters.

If it does:

- identify ordering key;
- handle out-of-order arrivals;
- define watermark/lateness policy;
- test reordering.

Do not assume global ordering from a partitioned system.

---

## Late-arriving data

Define behavior for late data.

Possible strategies:

- accept within lateness window;
- recompute aggregates;
- quarantine;
- ignore with explicit metrics.

Do not silently drop late records if consumers expect completeness.

---

## Schema management

Use versioned schemas where the system has durable producers/consumers.

Possible formats:

- JSON Schema;
- Avro;
- Protobuf;
- database schema;
- typed code contracts;
- another formal schema.

The format matters less than explicit compatibility.

---

## Schema evolution

Define compatibility rules.

Examples:

- backward-compatible;
- forward-compatible;
- full-compatible;
- breaking migration.

Do not change meaning while keeping the same field name/version if consumers
cannot distinguish it.

---

## Unknown fields

Decide intentionally whether unknown fields are:

- rejected;
- preserved;
- ignored.

Security-sensitive systems may prefer strictness.

Forward-compatible systems may preserve or ignore them.

Do not let arbitrary unvalidated fields silently influence downstream policy.

---

## Type coercion

Avoid dangerous implicit coercion.

Examples:

```text
"false" -> true
"00123" -> 123
"1e309" -> infinity
```

Be explicit.

Data quality failures should be visible.

---

## Validation

Validate as early as practical.

Potential checks:

- schema;
- required fields;
- enum values;
- range;
- format;
- referential integrity;
- tenant ownership;
- timestamp sanity;
- duplicate keys;
- business invariants.

Do not wait until the final consumer to discover obvious corruption.

---

## Quarantine

Use quarantine/dead-letter storage when useful.

Quarantine records should preserve:

- original input;
- source;
- failure reason;
- timestamp;
- pipeline version;
- retry/repair state.

Protect quarantine data according to source sensitivity.

Do not turn quarantine into an unmanaged permanent dump.

---

## Dead-letter handling

A dead-letter queue/store needs an operational process.

Define:

- ownership;
- alerting;
- retry policy;
- expiration;
- repair workflow.

Do not create a DLQ with no one responsible for draining or reviewing it.

---

## Data quality

Define measurable quality dimensions appropriate to the dataset.

Examples:

- completeness;
- validity;
- uniqueness;
- timeliness;
- consistency;
- referential integrity;
- distribution drift.

Do not invent dozens of quality checks that no consumer needs.

---

## Data contracts

For important producer/consumer boundaries, consider explicit data contracts.

A contract may define:

- schema;
- semantic meaning;
- freshness;
- ownership;
- quality expectations;
- compatibility.

Avoid contractual language if there is no operational enforcement.

---

## Transformation design

Transformations should be:

- deterministic where practical;
- testable;
- versioned;
- observable.

Separate:

```text
raw data
    ->
business transformation
    ->
published result
```

when that improves traceability.

---

## Pure transformations

Prefer pure/stateless transformations where possible.

They are easier to:

- test;
- replay;
- reason about;
- parallelize.

Do not force stateful architectures unless the problem requires them.

---

## Stateful processing

If state is required:

- define key;
- state lifetime;
- checkpointing;
- recovery;
- schema/version;
- concurrency behavior.

State migration is part of deployment design.

---

## Idempotency

Reprocessing should not create uncontrolled duplicates.

Possible techniques:

- upsert by business key;
- dedup table;
- idempotency key;
- content hash;
- versioned partition replacement.

Document idempotency boundaries.

---

## Deterministic outputs

Where reproducibility matters, avoid transformations that depend invisibly on:

- current time;
- random values;
- unordered iteration;
- mutable lookup tables;
- external APIs.

If such dependencies are required, capture their version/state.

---

## Lookup/enrichment data

Enrichment sources are dependencies.

Track:

- dataset version;
- freshness;
- trust;
- availability;
- fallback behavior.

Do not enrich security-sensitive records with arbitrary mutable remote data
without recording what was used.

---

## External API enrichment

If the pipeline calls external APIs:

- authenticate;
- use timeouts;
- bound retries;
- rate limit;
- cache intentionally;
- validate responses;
- define failure behavior.

Do not let API outage create infinite backlog or uncontrolled retries.

---

## Sensitive fields

Identify sensitive fields explicitly.

Apply controls such as:

- masking;
- tokenization;
- encryption;
- access restrictions;
- field-level redaction.

Do not depend on column names alone to detect all sensitive data.

---

## Data minimization

Retain only data needed for the pipeline purpose.

Do not copy every available source field into every downstream layer.

Minimization reduces:

- privacy risk;
- breach impact;
- storage cost;
- compliance burden.

---

## Pseudonymization

Where useful, replace direct identifiers with stable pseudonyms.

Document:

- reversibility;
- key management;
- collision properties;
- scope.

Do not call reversible tokenization "anonymization".

---

## Anonymization

Do not claim data is anonymous unless re-identification risk has been
meaningfully assessed.

Removing names alone is usually insufficient.

---

## Encryption in transit

Use authenticated encryption across relevant network boundaries.

Do not disable certificate verification for convenience.

---

## Encryption at rest

Use storage-platform encryption where appropriate.

Application-level encryption may be justified for highly sensitive fields.

Do not add bespoke encryption without a workable key lifecycle.

---

## Key management

If the pipeline encrypts data:

- identify key owner;
- rotate;
- separate key and data access where useful;
- audit access;
- handle recovery.

Do not store encryption keys next to encrypted data in the same unprotected
configuration.

---

## Tokenization

If using tokenization:

- define token scope;
- define lookup/reversal authority;
- avoid cross-tenant collisions/leakage;
- protect token vault.

---

## Secrets in data

Pipelines may accidentally ingest credentials.

Consider secret detection for sources such as:

- logs;
- source repositories;
- support tickets;
- uploads.

Do not replicate discovered secrets into more downstream systems unnecessarily.

---

## Data classification propagation

Where downstream controls depend on classification, propagate classification
metadata.

Do not allow a transform to strip sensitivity labels accidentally.

---

## Multi-tenancy

For multi-tenant pipelines:

- derive tenant from trusted source context;
- partition data;
- scope queries;
- scope caches;
- scope temporary storage;
- test cross-tenant denial.

Do not trust a user-supplied tenant field as the sole isolation boundary.

---

## Tenant joins

Cross-tenant joins should be denied by default unless explicitly required.

Validate join keys and tenant constraints.

A missing tenant predicate can create catastrophic leakage.

---

## Access control

Control access at:

- source;
- intermediate storage;
- transform execution;
- published dataset;
- admin plane.

Do not rely on one perimeter control.

---

## Workload identity

Prefer distinct workload identities for pipeline components.

Examples:

- ingest;
- transform;
- publish;
- reconcile;
- delete.

Use least privilege.

---

## Human access

Production data access by humans should be deliberate.

Consider:

- read-only access;
- audited queries;
- temporary elevation;
- masked views.

Do not make operational debugging depend on broad permanent warehouse-admin
rights.

---

## Service accounts

Avoid one shared service account for all pipelines.

Scope by:

- environment;
- pipeline;
- function.

---

## Environment separation

Separate non-production and production data.

Do not copy production-sensitive datasets into development automatically.

Use:

- synthetic data;
- masked data;
- sampled/redacted data.

---

## Temporary storage

Temporary files and staging tables can leak sensitive data.

Apply:

- restricted permissions;
- encryption;
- unique names;
- cleanup;
- TTLs;
- tenant separation.

Do not treat staging as exempt from security controls.

---

## Raw/landing zones

Raw data can be the most sensitive layer.

Restrict access.

Do not assume "raw" means temporary or harmless.

---

## Curated/published zones

Published data should have a clear contract.

Consumers should know:

- schema;
- freshness;
- owner;
- quality expectations;
- sensitivity;
- version.

---

## Data lineage

Preserve lineage sufficient to trace outputs.

Potential lineage metadata:

- source dataset;
- source partition;
- transformation job;
- code/version;
- run ID;
- target dataset;
- timestamp.

Lineage should support incident investigation and reprocessing.

---

## Column-level lineage

Use column-level lineage only when the value justifies the complexity.

It can be useful for:

- regulated data;
- sensitive-field tracking;
- complex transformations.

Do not introduce it by default for trivial pipelines.

---

## Provenance

For high-assurance pipelines, provenance may include:

- code digest;
- container/runtime digest;
- input dataset version;
- config;
- output artifact identity.

Compose with `software-supply-chain.md` where appropriate.

---

## Dataset versioning

Where consumers require reproducibility, give datasets or partitions stable
version identity.

Possible methods:

- snapshot ID;
- partition version;
- table version;
- commit ID;
- content hash.

Do not make historical reproduction depend on mutable live tables only.

---

## Immutability

Immutable raw snapshots can simplify:

- audit;
- replay;
- incident analysis.

But they increase retention/privacy burden.

Use intentionally.

---

## Mutable datasets

If datasets are mutable, define:

- update semantics;
- delete semantics;
- version history;
- auditability.

---

## CDC

For change-data-capture pipelines:

- track source offsets;
- preserve operation type;
- handle deletes;
- handle schema change;
- detect gaps;
- recover from lag.

Do not assume CDC is complete without monitoring source position.

---

## Tombstones and deletes

Deletes must be represented intentionally.

Do not drop delete events if downstream systems must remove data.

---

## Retention

Define retention by dataset purpose.

Avoid invented compliance periods.

Retention policy should cover:

- raw;
- intermediate;
- curated;
- quarantine;
- logs;
- backups where applicable.

---

## Deletion workflows

Support deletion when required.

A deletion request may need to propagate through:

```text
source
  ->
raw
  ->
transformed
  ->
indexes
  ->
caches
  ->
exports
```

Document what cannot be deleted immediately.

---

## Right-to-delete / privacy requests

If the system is subject to deletion obligations, ensure records can be found
reliably.

Do not promise full deletion if backups or downstream copies remain outside
the pipeline's control.

---

## Backfills

Backfills are production changes.

Before a backfill define:

- target range;
- expected volume;
- destination semantics;
- overwrite/merge behavior;
- cost;
- downstream impact.

Do not run unbounded historical backfills casually.

---

## Reprocessing

Reprocessing should use an explicit run identity.

Preserve:

- source range;
- pipeline version;
- config;
- output destination.

Do not mix reprocessed output silently with existing output unless merge
semantics are clear.

---

## Replay

For streams, replay should not bypass:

- validation;
- authorization;
- deduplication;
- rate limits.

Replay is not a privileged shortcut around normal controls.

---

## Checkpointing

Checkpoint state should be durable enough for the failure model.

Do not acknowledge progress before output durability if that would lose data.

---

## Transactions

Use transactions where they simplify correctness.

For distributed pipelines, document where atomicity ends.

Do not imply a global transaction if only per-stage atomicity exists.

---

## Exactly-once claims

Exactly-once is an end-to-end property.

It depends on:

- source;
- processing;
- checkpointing;
- sink semantics.

Do not claim exactly-once based solely on one framework feature.

---

## Failure modes

Define behavior for:

- source unavailable;
- malformed data;
- schema mismatch;
- sink unavailable;
- transformation bug;
- credential failure;
- quota exhaustion;
- partial partition failure;
- downstream timeout.

---

## Fail-open versus fail-closed

For data-quality/security validation, choose behavior intentionally.

Security-sensitive policy should generally fail closed.

For non-critical enrichment, degraded processing may be acceptable.

Do not use one global fallback.

---

## Retry policy

Retry transient failures only.

Bound retries.

Use backoff/jitter where appropriate.

Do not retry malformed records forever.

---

## Poison-pill records

One bad record should not permanently block an entire partition unless the
correctness model requires it.

Use quarantine/dead-letter patterns where appropriate.

---

## Backpressure

Streaming pipelines need backpressure.

Bound:

- queues;
- in-flight records;
- memory;
- concurrency.

Do not let downstream outage cause unbounded accumulation in process memory.

---

## Queue retention

If the broker is the recovery buffer, ensure retention exceeds realistic outage
windows.

Do not invent retention without capacity and recovery requirements.

---

## Load shedding

For non-critical telemetry pipelines, dropping low-priority data under
saturation may be acceptable.

If so:

- define what can be dropped;
- meter losses;
- expose it.

Do not silently lose records.

---

## Watermarks

For event-time processing, define watermark semantics.

Understand how lateness affects:

- windows;
- aggregates;
- finality.

---

## Time

Distinguish:

- event time;
- ingestion time;
- processing time.

Do not mix them in metrics or business logic without intent.

---

## Clock assumptions

If ordering or windows depend on timestamps, consider clock skew and malformed
future/past timestamps.

---

## Observability

Track pipeline-specific signals.

Examples:

- records ingested;
- records processed;
- records rejected;
- records quarantined;
- lag;
- throughput;
- error rate;
- retry count;
- schema failures;
- checkpoint age;
- backfill progress;
- data freshness;
- late-arrival count;
- duplicate count.

---

## Data quality observability

Track important quality signals separately from runtime health.

A pipeline can be green operationally while producing bad data.

---

## Freshness

Measure freshness from the consumer's perspective.

Examples:

```text
now - latest successfully published event time
```

or:

```text
now - latest completed source partition
```

Do not use job success alone as proof of fresh data.

---

## Completeness

Where completeness matters, compare:

- expected partitions/files/records;
- received;
- processed;
- published.

---

## Reconciliation

For critical pipelines, periodically reconcile source and sink.

Examples:

- record counts;
- checksums;
- source offsets;
- aggregate totals.

Do not assume continuous processing guarantees no gaps.

---

## Drift

Data distribution drift may indicate:

- source change;
- parser bug;
- upstream incident;
- attack.

Use drift checks where the data and risk justify them.

---

## Security telemetry

Track:

- authn failures;
- authz denials;
- cross-tenant violations;
- unexpected source identities;
- sensitive-field detection;
- policy failures.

---

## Auditability

Important data-control actions should be attributable.

Examples:

- schema change;
- backfill;
- deletion;
- privilege change;
- production reprocess;
- dataset publication.

---

## Logging

Do not log entire sensitive records by default.

Prefer:

- record ID;
- partition;
- source;
- failure code;
- schema version.

Redact sensitive fields.

---

## Sampling logs

If payload samples are necessary for debugging:

- restrict access;
- redact;
- bound retention;
- avoid random uncontrolled sensitive-data exposure.

---

## Metrics cardinality

Avoid raw customer IDs, event IDs, paths, or payloads as metric labels.

---

## Run identity

Every pipeline execution should have a run/job identity.

Use it across:

- logs;
- metrics;
- lineage;
- output metadata;
- audit.

---

## Orchestration

Use orchestration only when needed.

The orchestrator should make dependencies explicit.

Avoid hidden cross-job coupling through undocumented shared state.

---

## DAG design

For DAG-based systems:

- keep stages coherent;
- avoid enormous monolithic DAGs;
- avoid one task per record;
- make retries stage-aware.

---

## Scheduling

Scheduling should reflect data availability.

Do not run on a fixed clock when event-based triggering is substantially safer
and simpler, unless platform constraints require time-based scheduling.

---

## Concurrency

Bound concurrent runs.

Avoid overlapping backfills or overlapping writes to the same partitions
unless supported.

---

## Locking

If only one writer is allowed for a target, enforce it.

Do not rely on operators remembering not to launch another run.

---

## Deployment

Pipeline code changes should be deployable independently from data where
possible.

Track:

- code version;
- config version;
- schema version.

---

## Migrations

Schema and transformation changes may require migration.

Prefer backward-compatible staged changes.

Avoid requiring all producers and consumers to upgrade simultaneously unless
necessary.

---

## Rollback

Rollback can be complicated because output data may already have changed.

Define whether rollback means:

- redeploy old code;
- rebuild prior dataset;
- restore snapshot;
- replay source;
- compensate changes.

Do not assume application-style rollback automatically repairs transformed
data.

---

## Canary / shadow processing

For risky transformation changes, consider:

- shadow runs;
- parallel output;
- sampled canary partitions.

Compare:

- row counts;
- quality metrics;
- business aggregates;
- schema;
- sensitive-field handling.

Do not publish experimental output to production consumers accidentally.

---

## Data diffing

For transformation changes, compare old and new outputs.

Useful checks:

- record count;
- changed rows;
- null distribution;
- aggregate totals;
- schema differences;
- sensitive-field movement.

---

## Testing strategy

### Unit tests

Test pure transformation logic.

### Schema tests

Test:

- compatibility;
- required fields;
- invalid payloads;
- unknown fields.

### Integration tests

Exercise:

- source connector;
- storage;
- transform runtime;
- sink.

### Security tests

Test:

- unauthorized source;
- unauthorized sink;
- cross-tenant data;
- path traversal;
- malicious files;
- secret handling.

### Recovery tests

Test:

- retry;
- checkpoint resume;
- duplicate delivery;
- partial failure;
- reprocessing.

### Data-quality tests

Use representative fixtures with:

- valid records;
- malformed records;
- edge values;
- duplicates;
- late data;
- missing fields.

---

## Property-based testing

Useful properties may include:

- reprocessing is idempotent;
- tenant ID never changes;
- row count relationships hold;
- normalization is deterministic;
- invalid schema never publishes.

---

## Fuzzing

Consider fuzzing parsers and decoders for hostile external data.

Especially relevant for:

- custom binary formats;
- archives;
- compressed inputs;
- custom protocol frames.

---

## Fixture hygiene

Do not use real secrets or unredacted production records in tests.

Use synthetic or sanitized fixtures.

---

## Acceptance test

Where practical, demonstrate:

```text
source fixture
  ->
ingest
  ->
validate
  ->
transform
  ->
publish
  ->
consumer-visible result
```

Also demonstrate at least one negative path.

Examples:

- malformed record quarantined;
- unauthorized tenant rejected;
- duplicate does not create duplicate output;
- delete propagates;
- failed sink resumes safely.

---

## Supply chain

Compose with `software-supply-chain.md` for release integrity.

Pin:

- transformation dependencies;
- runtime;
- connector versions;
- container images;
- schema tooling.

Do not use mutable runtime components in critical pipelines without control.

---

## Connector supply chain

Connectors often have powerful credentials.

Treat third-party connectors/plugins as privileged dependencies.

Review:

- publisher;
- permissions;
- version;
- update path;
- network behavior.

---

## Dependency minimization

Avoid adding large distributed systems to solve small local batch problems.

Use:

```text
simple file/database job
```

when that is sufficient.

Do not introduce:

- Kafka;
- Spark;
- Flink;
- Airflow;
- dbt;
- lakehouse formats;
- Kubernetes

without a concrete need.

---

## Documentation

The repository should document:

- source systems;
- target systems;
- data classifications;
- schema;
- transformation flow;
- lineage;
- retry/reprocessing;
- retention;
- deletion;
- operational ownership;
- failure modes.

A new engineer should be able to answer:

> Where did this dataset come from, what transformed it, and who is allowed to
> access it?

---

## Runbooks

For production pipelines, create runbooks for actual failure modes.

Examples:

- source stalled;
- schema broke;
- sink unavailable;
- DLQ growing;
- duplicate explosion;
- tenant leakage suspicion;
- backfill failed;
- deletion incomplete.

Do not create speculative runbooks for nonexistent components.

---

## Architecture decisions

Use ADRs for consequential choices such as:

- stream vs batch;
- broker;
- warehouse/lakehouse;
- delivery semantics;
- schema format;
- lineage system;
- tenancy model;
- replay strategy.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic secure data pipeline may resemble:

```text
.
├── AGENTS.md
├── README.md
├── src/
│   ├── ingest/
│   ├── validate/
│   ├── transform/
│   ├── publish/
│   ├── lineage/
│   └── security/
├── schemas/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── security/
│   ├── recovery/
│   └── fixtures/
├── docs/
│   ├── architecture/
│   ├── data-contracts/
│   └── adr/
└── <ecosystem-specific files>
```

Only create directories that contain meaningful content.

---

## Verification interface

A repository using this recipe should expose obvious commands or equivalent
native interfaces for:

```text
check
test
test-integration
test-security
test-recovery
validate-schema
run-local
backfill-dry-run
```

These names are illustrative.

Use the repository's native tooling.

---

## Acceptance criteria

A secure data pipeline is not complete because records moved from A to B.

Demonstrate the applicable subset of the following.

### Source and schema

- source identity is preserved;
- schema is explicit;
- invalid records are visible;
- schema changes are versioned.

### Security

- pipeline identities are least-privileged;
- sensitive fields are classified;
- secrets are not logged;
- cross-tenant isolation is tested;
- non-production does not receive production-sensitive data unintentionally.

### Integrity

- duplicates are handled;
- replay semantics are defined;
- partial failures are visible;
- output can be traced to source and transformation version.

### Reliability

- retries are bounded;
- poison records do not stall the system indefinitely;
- checkpoints/recovery work;
- sink/source outage behavior is understood.

### Privacy

- retention is explicit;
- deletion behavior is documented;
- temporary/quarantine data is protected;
- data minimization is applied where relevant.

### Observability

- throughput is visible;
- lag/freshness is visible;
- schema failures are visible;
- quality failures are visible;
- backfill/reprocessing progress is visible.

### Operations

- backfill has safe scope;
- rollback/recovery semantics are documented;
- lineage supports incident analysis;
- run identity is preserved.

---

## Optional composition

Common combinations:

```text
secure-data-pipeline + zero-trust-service
```

For strong workload identity and least-privilege access between pipeline
components.

```text
secure-data-pipeline + software-supply-chain
```

For provenance of transformation code, connectors, schemas, and runtime
artifacts.

```text
secure-data-pipeline + hostile-input
```

For pipelines ingesting arbitrary files, public feeds, user uploads, or
attacker-controlled telemetry.

```text
secure-data-pipeline + high-assurance
```

For stricter lineage, independent reconciliation, fault injection, and stronger
recovery evidence.

```text
secure-data-pipeline + ai-security
```

For data pipelines that feed AI systems, embeddings, training corpora, or RAG
indexes and therefore need poisoning and provenance controls.

---

## Anti-patterns

Avoid:

- treating schema-less JSON as a durable contract;
- one admin credential for every stage;
- silent coercion of malformed data;
- dead-letter queues nobody owns;
- claiming exactly-once without end-to-end evidence;
- missing delete/tombstone propagation;
- reprocessing without idempotency;
- mixing tenants without deterministic tenant constraints;
- copying production data into development by default;
- logging full sensitive records;
- storing secrets in raw data lakes indefinitely;
- mutable unversioned transformation logic;
- backfills without bounded scope;
- hidden manual repair steps;
- lineage that cannot identify source or transformation version;
- processing-time success presented as data-quality success;
- silent partial ingestion;
- unbounded queues;
- one bad record blocking a stream forever;
- vector/database/index copies surviving after source deletion with no policy.

Do not mistake movement for correctness.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. source systems;
2. target systems;
3. trust classification;
4. schema/versioning model;
5. workload identity model;
6. sensitive-data classification;
7. validation/quarantine model;
8. idempotency/deduplication model;
9. delivery semantics;
10. lineage/provenance model;
11. retention/deletion model;
12. retry/recovery model;
13. observability/quality metrics;
14. backfill/reprocessing model;
15. deterministic tests executed;
16. security/recovery tests executed;
17. commands actually run;
18. observed results;
19. known data-quality/security gaps;
20. controls deliberately not implemented and why.

Never claim a pipeline is "secure", "complete", "exactly once", "compliant",
or "reproducible" merely because data arrived at the destination.

Describe the actual identity, schema, lineage, integrity, privacy, and recovery
controls.

---

## Guiding principle

A secure data pipeline should make data movement explainable.

It should be able to answer:

> Where did this data come from?

> Who was allowed to move it?

> What schema did it satisfy?

> What transformed it?

> What sensitive fields did it contain?

> What happened when processing failed?

> Can we replay or delete it safely?

> Which exact source and transformation produced this output?

Move data deliberately.

Preserve provenance.

Reject ambiguity.

Minimize privilege.

Treat invalid data as invalid.

And never let a pipeline silently turn unknown or untrusted input into trusted
downstream truth.
