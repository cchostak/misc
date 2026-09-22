# data-platform.md

> Recipe for scaffolding or elevating a shared data platform.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md`, `control-plane.md`,
> `developer-platform.md`, `eventing-platform.md`,
> `observability-platform.md`, `secure-data-pipeline.md`,
> `identity-platform.md`, `api-platform.md`, and profiles such as
> `zero-trust-service.md`, `software-supply-chain.md`, `high-assurance.md`,
> `hostile-input.md`, or `ai-security.md` where appropriate.
>
> This recipe defines architecture expectations for platforms that provide
> shared ingestion, storage, processing, governance, metadata, lineage,
> quality, access, serving, and lifecycle capabilities for data producers and
> consumers.
>
> It does not require a specific warehouse, lake, lakehouse, stream processor,
> catalog, query engine, orchestration tool, cloud, storage format, or data
> governance product.

## Purpose

Use this recipe when the architecture provides shared data capabilities to
multiple producers, consumers, domains, or teams.

Typical examples:

- enterprise data platforms;
- analytical data platforms;
- data lakes;
- warehouses;
- lakehouses;
- streaming data platforms;
- security data platforms;
- ML feature/data platforms;
- operational analytics platforms;
- domain-oriented data platforms;
- regulated data platforms;
- metadata/catalog platforms;
- cross-system data sharing platforms.

The goal is not to centralize every dataset into one technology.

The goal is to create a coherent data platform where teams can answer:

- What is the authoritative source?
- Who owns this dataset?
- What is its schema?
- How did this data get here?
- Who may access it?
- How fresh is it?
- How complete is it?
- What transformations produced it?
- How is sensitive data classified?
- How are retention and deletion enforced?
- Can the dataset be rebuilt?
- How are schema changes introduced?
- How are consumers notified?
- What happens when a pipeline partially fails?
- What is the difference between operational, derived, and analytical truth?

---

## Core principle

A data platform moves trust as well as bytes.

A useful architecture model is:

```text
Sources
   |
   v
Ingestion Plane
   |
   v
Validation / Classification
   |
   v
Storage / Processing Plane
   |
   +--> metadata
   +--> lineage
   +--> quality
   +--> governance
   +--> access policy
   |
   v
Serving / Query Plane
   |
   v
Consumers
```

The platform should preserve enough information to answer:

> Which source produced this data, under which schema and policy, through which
> transformations, at what time, with what quality evidence, and who is
> authorized to use it?

---

## Architectural invariants

A data platform SHOULD satisfy these invariants unless there is a documented
reason not to.

### 1. Dataset ownership is explicit

Every durable dataset should have an accountable owner.

Ownership includes:

- semantic meaning;
- schema;
- lifecycle;
- quality;
- access;
- support.

Do not create ownerless enterprise data.

### 2. Source authority is explicit

Every dataset should distinguish:

- authoritative source;
- derived copy;
- cache;
- projection;
- aggregate.

Do not let downstream replicas silently become competing sources of truth.

### 3. Schema is a contract

Durable data should have explicit shape and evolution rules.

Do not treat "JSON" as a sufficient schema strategy.

### 4. Provenance is preserved

Data should retain enough lineage to identify:

- source;
- transformation;
- version;
- processing run;
- time.

### 5. Access is identity- and resource-aware

Authorization should be based on trusted identity and dataset/resource scope.

Do not rely on obscurity, naming, or network location.

### 6. Sensitive data is classified

Classification should influence:

- access;
- encryption;
- masking;
- retention;
- replication;
- observability.

### 7. Invalid data remains invalid

Malformed, unauthorized, or semantically invalid data should not silently
become trusted downstream truth.

### 8. Reprocessing is safe

Backfills, replays, and rebuilds should not corrupt or duplicate downstream
state.

### 9. Quality is observable

Pipeline success does not imply data correctness.

Track:

- freshness;
- completeness;
- validity;
- reconciliation.

### 10. Deletion and retention propagate intentionally

Data lifecycle must include:

- source;
- derived datasets;
- indexes;
- caches;
- backups;
- exports.

### 11. Platform abstractions are versioned

Schemas, contracts, transformation logic, and serving interfaces evolve.

Treat them as compatibility-sensitive.

### 12. Derived data is reproducible where practical

A consumer should know whether a dataset can be rebuilt from durable source
and versioned transformations.

---

## Data platform intent

Start with a concise platform statement.

Example:

```text
For engineering and analytics teams, the data platform provides governed
ingestion, storage, transformation, discovery, and serving through versioned
dataset contracts while preserving lineage, access boundaries, data quality,
and lifecycle controls.
```

Also define non-goals.

Examples:

- platform does not own business meaning for every domain;
- platform does not replace operational databases;
- platform does not centralize all transformations into one team;
- platform does not make every dataset globally discoverable;
- platform does not imply all data is suitable for AI/ML use.

---

## Data domains

Identify logical data domains.

Examples:

- customer;
- payments;
- security;
- product;
- telemetry;
- identity;
- finance.

Domains should reflect ownership and semantics, not arbitrary storage layout.

---

## Domain ownership

A domain should own:

- definitions;
- quality;
- schema;
- access intent;
- lifecycle.

Platform teams should provide common capability.

Do not make the central platform team the semantic owner of every dataset.

---

## Data products

A data product may expose:

- dataset;
- schema;
- quality guarantees;
- freshness;
- access policy;
- documentation;
- lineage.

Use the term only when the organization can support the ownership model.

Do not relabel unmanaged tables as "data products".

---

## Dataset contract

A dataset contract should define:

- owner;
- purpose;
- schema;
- keys;
- update model;
- freshness;
- retention;
- classification;
- access;
- compatibility;
- quality expectations.

---

## Dataset identity

Use stable dataset identity.

Do not identify a dataset solely by mutable storage path.

A logical dataset may move between technologies while keeping the same
consumer-facing identity.

---

## Physical versus logical identity

Separate:

```text
logical dataset
```

from:

```text
table / bucket / file path / topic / index
```

when the platform must support storage evolution.

---

## Source systems

Inventory source systems.

For each source define:

- owner;
- trust level;
- schema;
- ingestion method;
- update semantics;
- deletion semantics;
- data classification.

---

## Authoritative source

Document which system is authoritative.

Examples:

```text
customer profile -> operational customer service
billing state    -> billing ledger
security events  -> source event stream
```

Do not let analytical copies become write-back sources without explicit design.

---

## Source trust

Classify source trust.

Examples:

```text
authoritative internal
derived internal
partner-provided
user-generated
internet/untrusted
```

Validation and review should reflect trust.

---

## Ingestion plane

The ingestion plane may support:

- batch;
- streaming;
- CDC;
- file/object ingest;
- API ingest;
- event ingest.

Use the simplest mode that matches source semantics.

---

## Batch ingestion

Batch ingestion should define:

- schedule;
- source snapshot/window;
- idempotency;
- partial failure;
- re-run behavior.

Do not use current wall-clock time as the only batch identity.

---

## Stream ingestion

Stream ingestion should define:

- ordering;
- delivery semantics;
- checkpoint;
- replay;
- schema;
- late data.

Compose with `eventing-platform.md`.

---

## CDC ingestion

CDC should define:

- snapshot;
- log position;
- update/delete semantics;
- schema change behavior.

Raw CDC is not automatically a stable consumer contract.

---

## File ingestion

For file/object sources:

- verify size;
- format;
- checksum where useful;
- decompression limits;
- path/key safety;
- duplicate identity.

Compose with `hostile-input.md` for untrusted sources.

---

## API ingestion

For API-based sources:

- finite timeouts;
- rate limits;
- pagination;
- retries;
- checkpoint;
- partial result handling.

---

## Source credentials

Use least-privilege source credentials.

Prefer short-lived workload identity where possible.

Do not give every ingestion job broad source-system admin access.

---

## Ingestion identity

Every ingest run or stream should have a stable operation identity.

Useful fields:

- source;
- run ID;
- source revision/window;
- started;
- completed;
- status.

---

## Ingestion manifest

For batch/file ingest, consider a manifest containing:

- inputs;
- checksums;
- schema;
- expected count/size;
- output identity.

This improves reproducibility and audit.

---

## Duplicate ingest

Re-running the same input should not duplicate durable state unexpectedly.

Use:

- run identity;
- source watermark;
- idempotent merge;
- deduplication key.

---

## Partial ingestion

Represent partial success explicitly.

Do not mark a run successful if required partitions/files failed.

---

## Quarantine

Invalid or suspicious records may go to quarantine.

Quarantine should have:

- owner;
- reason;
- retention;
- remediation path.

Do not create an unreviewed data graveyard.

---

## Dead-letter handling

For streaming/batch records that repeatedly fail:

- bound retries;
- quarantine/DLQ;
- preserve error reason;
- allow safe replay.

---

## Validation

Validate at ingestion boundaries.

Potential checks:

- schema;
- type;
- size;
- required fields;
- enum;
- referential expectation;
- classification.

---

## Syntactic versus semantic validation

Schema validation answers:

```text
is this structurally valid?
```

Semantic validation asks:

```text
does this value make sense?
```

Both may be needed.

---

## Unknown fields

Choose behavior intentionally.

Possible modes:

- preserve;
- ignore;
- reject.

Do not silently discard important new source fields without awareness.

---

## Coercion

Avoid silent lossy coercion.

Examples:

- string `"abc"` to null;
- float to integer truncation;
- invalid timestamp to current time.

Invalid data should remain visible.

---

## Schema registry

A registry/catalog may provide:

- schema storage;
- versioning;
- compatibility;
- ownership;
- discovery.

Do not make registration equivalent to semantic review.

---

## Schema evolution

Prefer additive compatible changes where possible.

Potential breaking changes include:

- field removal;
- type change;
- semantic change;
- key change;
- nullability change.

---

## Column semantics

A column name is not sufficient documentation.

Document:

- meaning;
- unit;
- timezone;
- encoding;
- allowed values.

---

## Time semantics

Distinguish:

- event time;
- source update time;
- ingest time;
- processing time;
- load time.

Do not collapse all timestamps into `timestamp`.

---

## Timezones

Use explicit timezone-aware timestamps.

Prefer UTC internally where appropriate.

Do not rely on machine-local timezone.

---

## Keys

Define:

- primary key;
- natural key;
- surrogate key;
- version key.

Avoid relying on row position or unstable ordering.

---

## Identity resolution

If multiple systems represent the same real-world entity, define identity
resolution rules.

Do not join records based solely on mutable display fields without evidence.

---

## Join semantics

Joins can change trust and sensitivity.

For important joins define:

- key;
- cardinality;
- expected unmatched rate;
- tenant scope.

---

## Cross-tenant joins

Deny by default unless explicit use case and policy permit.

---

## Transformation plane

Transformations should be:

- versioned;
- reviewable;
- attributable;
- testable.

Do not allow untracked ad hoc production transformations to become durable
truth.

---

## Pure transformations

Prefer deterministic transformations where possible.

A pure transform:

```text
same inputs + same version -> same outputs
```

This improves rebuildability.

---

## Stateful transformations

If transformation depends on state:

- version state;
- checkpoint;
- consistency;
- recovery.

---

## External enrichment

If transformations call external APIs:

- cache where appropriate;
- record version/time;
- handle unavailable dependency;
- preserve provenance.

External enrichment reduces reproducibility unless captured.

---

## Transformation identity

Record:

- code/version;
- input dataset versions;
- parameters;
- execution environment;
- output dataset version.

---

## Data lineage

Lineage should answer:

```text
where did this data come from?
what changed it?
what depends on it?
```

Capture lineage automatically where practical.

---

## Column-level lineage

Use when required for:

- sensitive fields;
- compliance;
- complex analytics.

Do not require full column lineage if operational cost exceeds value.

---

## Provenance

Provenance should be durable enough for important datasets.

Possible elements:

- source version;
- transform revision;
- runtime;
- execution ID;
- timestamp.

---

## Dataset versioning

Version datasets when consumers need reproducible snapshots.

Possible identities:

- immutable snapshot ID;
- partition/version;
- table snapshot;
- commit-like version.

---

## Mutable datasets

Mutable tables are common.

Define:

- update model;
- consistency;
- time travel if available;
- snapshot semantics.

---

## Immutable datasets

Immutable append/snapshot models simplify provenance.

Use when compatible with domain needs.

---

## Append-only logs

Append-only does not mean data can never be corrected or deleted.

Define correction/tombstone semantics.

---

## Late-arriving data

Late data may change derived results.

Define:

- lateness window;
- reprocessing;
- correction notification.

---

## Backfills

Backfills should be first-class operations.

A backfill should define:

- input range;
- transformation version;
- output target;
- rate/capacity;
- dry-run where possible.

---

## Backfill safety

Protect production systems from unbounded backfills.

Use:

- quotas;
- concurrency;
- isolated compute;
- read limits.

---

## Reprocessing

Reprocessing should not silently duplicate output.

Use idempotent partition replacement/merge semantics.

---

## Replay

Streaming data replay should preserve original event identity and timing context
where useful.

Compose with `eventing-platform.md`.

---

## Storage plane

Choose storage based on workload.

Potential categories:

- object storage;
- relational warehouse;
- columnar store;
- key-value;
- search/index;
- time-series;
- feature store.

Do not force every dataset into one storage engine.

---

## Raw zone

A raw/landing layer may preserve source fidelity.

Protect it strongly.

Raw data often contains the most sensitive and least-clean content.

---

## Curated zone

Curated data should have:

- validated schema;
- ownership;
- quality;
- documentation.

---

## Serving zone

Serving datasets should be optimized for consumer access patterns.

Do not expose internal intermediate tables as stable public contracts without
intent.

---

## Bronze/silver/gold naming

Layered naming can be useful.

Do not rely on color names without explicit semantics.

---

## Storage format

Choose formats that support:

- schema evolution;
- compression;
- partitioning;
- tooling.

Do not optimize format before understanding access patterns.

---

## Partitioning

Partition based on common filters and data volume.

Avoid over-partitioning into millions of tiny files/objects.

---

## Small-file problem

Batch/stream outputs can create too many small files.

Use compaction where needed.

---

## Compaction

Compaction should preserve:

- data semantics;
- versioning;
- deletion/tombstones.

---

## Indexing

Indexes improve query performance.

Treat indexes as derived state.

They should be rebuildable from authoritative data where practical.

---

## Search indexes

Search systems often lag source data.

Expose freshness semantics.

Do not treat index absence as authoritative deletion unless contract says so.

---

## Caches

Caches should have:

- TTL/invalidation;
- tenant scope;
- rebuild strategy.

Do not let stale cache become source of truth.

---

## Serving plane

Consumers may access data through:

- SQL/query;
- API;
- file/object access;
- stream;
- feature serving.

Define stable access contracts.

---

## Query platform

A shared query engine should enforce:

- identity;
- dataset authorization;
- quotas;
- query limits.

Do not give all analysts unrestricted access to every dataset.

---

## Query isolation

Protect against:

- expensive full scans;
- cross-tenant access;
- resource starvation.

Use workload groups/quotas where available.

---

## Query timeouts

Bound long-running queries.

Do not let abandoned queries consume resources indefinitely.

---

## Query cancellation

Support cancellation.

---

## Query concurrency

Limit concurrency according to tenant/workload class.

---

## Materialized views

Materialized views are derived state.

Track:

- source;
- refresh;
- freshness;
- owner.

---

## Semantic layer

A semantic layer can standardize business measures.

Use only when ownership and governance are clear.

Do not centralize every analytical expression prematurely.

---

## Metrics definitions

Shared business metrics should have:

- owner;
- definition;
- unit;
- time window;
- source.

Avoid two incompatible "revenue" metrics with the same name.

---

## Data APIs

For programmatic serving APIs, compose with `api-platform.md`.

Keep domain contracts separate from storage technology.

---

## Bulk export

For large exports:

- asynchronous execution;
- authorization;
- expiry;
- signed access;
- audit.

Do not expose unrestricted full-table download by default.

---

## Data sharing

Shared datasets should have explicit consumer agreements.

Define:

- allowed purpose;
- access;
- retention;
- redistribution.

---

## External sharing

Partner/customer sharing needs stronger controls.

Use separate identities and auditable grants.

---

## Data marketplace

A catalog/marketplace can improve discovery.

Do not make discoverability equivalent to authorization.

---

## Metadata platform

Metadata may include:

- owner;
- schema;
- classification;
- lineage;
- quality;
- freshness;
- lifecycle.

Keep metadata authoritative where possible.

---

## Catalog

A catalog should help users discover datasets.

It should not become a second manually maintained truth.

Prefer synchronization from actual systems.

---

## Ownership metadata

Ownership should map to stable teams/groups.

Avoid personal owner values for long-lived datasets where team ownership is
more appropriate.

---

## Classification

Classify datasets.

Example:

```text
public
internal
confidential
restricted
regulated
```

Use the organization's actual taxonomy.

---

## Classification inheritance

Derived data may inherit sensitivity from sources.

Do not automatically downgrade classification because fields were transformed.

---

## Reclassification

If data is masked/aggregated sufficiently, classification may change.

Require explicit evidence/policy.

---

## Governance

Govern:

- ownership;
- schema;
- access;
- lifecycle;
- classification;
- quality.

Do not centralize domain semantics unnecessarily.

---

## Data stewardship

Stewards may support domain owners.

Avoid unclear dual ownership.

---

## Access model

Authorization should consider:

```text
subject
dataset
action
purpose/context where required
environment
tenant
```

---

## Authentication

Use enterprise/workload identity.

Compose with `identity-platform.md`.

Avoid shared database passwords.

---

## Authorization

Use resource-aware controls.

Potential models:

- RBAC;
- ABAC;
- policy-as-code;
- row/column policies.

---

## Row-level security

Use when multiple principals share storage and rows need isolation.

Test thoroughly.

Do not rely only on application filtering.

---

## Column-level security

Useful for sensitive fields.

Ensure exports and derived datasets preserve restrictions.

---

## Dynamic masking

Masking can reduce accidental exposure.

Do not treat reversible masking as anonymization.

---

## Tokenization

Tokenization/pseudonymization can reduce direct identifier exposure.

Protect re-identification capability.

---

## Anonymization

True anonymization is difficult.

Do not label pseudonymized data as anonymous casually.

---

## Purpose limitation

Some sensitive datasets may be restricted by use purpose.

If enforced, make policy and evidence explicit.

---

## Consent

Where consent affects processing, preserve consent state and changes.

Do not copy data into derived stores that cannot honor consent changes.

---

## Least privilege

Consumers should receive only required datasets/columns/actions.

Avoid broad warehouse admin roles.

---

## Service accounts

Machine consumers should use dedicated identities.

Avoid shared analyst/service credentials.

---

## JIT access

Sensitive datasets may use time-bound access.

Record:

- requester;
- approver;
- scope;
- expiry.

---

## Break-glass

Emergency access should be rare and auditable.

---

## Environment separation

Separate production and non-production data access.

Do not copy sensitive production data into development by default.

---

## Test data

Prefer:

- synthetic;
- masked;
- sampled;
- generated

test datasets.

Document limitations of synthetic data.

---

## Data egress

Control exports to:

- local machines;
- SaaS;
- external cloud;
- AI providers.

Data egress is a security boundary.

---

## Egress policy

Use allowlists/policy where sensitive data must remain within approved systems.

---

## Remote model/AI use

If data feeds AI/ML services, classify whether external model providers may
receive it.

Compose with `ai-security.md`.

---

## Encryption in transit

Use encryption across relevant trust boundaries.

---

## Encryption at rest

Use platform/storage-native encryption according to policy.

Do not claim strong tenant isolation solely from shared at-rest encryption.

---

## Key management

For high-sensitivity data, define:

- key owner;
- rotation;
- tenant separation;
- revocation.

---

## Bring-your-own-key

Use only when actual requirements justify operational complexity.

---

## Secrets

Do not store credentials in datasets, metadata, logs, or notebooks.

---

## Data quality

Data quality is part of platform correctness.

Potential dimensions:

- freshness;
- completeness;
- validity;
- uniqueness;
- consistency;
- reconciliation.

---

## Freshness

Define expected update interval.

Example:

```text
dataset refreshed within 30 minutes of source availability
```

Use actual requirements.

---

## Completeness

Measure whether expected records/partitions arrived.

Do not infer completeness from job exit code.

---

## Validity

Validate values against domain rules.

---

## Uniqueness

Check keys where uniqueness is required.

---

## Reconciliation

Compare counts/sums/checksums against source or ledger where appropriate.

This can catch silent partial ingest.

---

## Quality contracts

Critical datasets may have explicit quality SLOs.

Do not assign arbitrary thresholds without owner agreement.

---

## Quality ownership

Domain owners should own semantic quality.

Platform owns quality tooling and enforcement primitives.

---

## Quality failures

Decide whether quality failure:

- blocks publish;
- quarantines;
- marks degraded;
- alerts only.

Use based on downstream risk.

---

## Data drift

Track distribution/schema drift where it matters.

Do not flag every statistical change as an incident.

---

## AI/ML drift

For model features, drift may affect model performance.

Compose with AI evaluation/ML-specific controls.

---

## Freshness observability

Expose:

- last source event;
- last successful ingest;
- last complete partition;
- current lag.

---

## Pipeline health

Separate:

```text
pipeline process succeeded
```

from:

```text
expected data is present and valid
```

---

## Orchestration

Pipelines may use DAG/workflow orchestration.

Define:

- dependencies;
- retries;
- backfills;
- concurrency;
- failure state.

Do not hide data semantics inside scheduler configuration.

---

## Scheduler

A scheduler should not become the data source of truth.

It coordinates execution.

---

## DAG ownership

Every production DAG/workflow should have owner.

---

## Dependency cycles

Avoid circular pipeline dependencies.

Detect where practical.

---

## Retry

Retry transient failures.

Do not retry semantic invalidity forever.

---

## Partial failure

Represent which partitions/tasks failed.

Do not rerun entire huge pipelines if safe partial retry exists.

---

## Checkpointing

For long jobs/streams, checkpoint durable progress.

---

## Idempotency

Re-running a task should be safe.

Prefer partition-replace or merge by stable keys.

---

## Exactly-once claims

End-to-end exactly-once across source, transform, storage, and consumers is rare.

State actual guarantees.

---

## Transaction boundaries

Use transactions where one storage boundary supports them.

Across distributed systems, use reconciliation/idempotency rather than claiming
global atomicity.

---

## Publish boundary

Define when a dataset version becomes visible.

A useful pattern:

```text
write staging
    ->
validate
    ->
atomically publish pointer/version
```

This prevents consumers seeing partial data.

---

## Atomic publish

Where supported, publish complete snapshots atomically.

---

## Staging data

Staging areas should have retention and cleanup.

Do not accumulate abandoned partial outputs indefinitely.

---

## Version promotion

Promote validated dataset versions.

Do not mutate a published immutable version.

---

## Rollback

Rollback may point consumers to a prior known-good dataset version.

Understand whether downstream state must also be reconciled.

---

## Dataset release

For high-value data products, a release may include:

- version;
- schema;
- quality report;
- lineage;
- freshness;
- classification.

---

## Change management

Classify changes:

```text
compatible
behavior-changing
schema-breaking
quality-affecting
security-affecting
```

---

## Consumer impact

Before breaking changes, identify downstream consumers.

Use lineage/catalog/query telemetry where practical.

---

## Consumer notification

Notify affected owners.

Do not rely only on a wiki update.

---

## Deprecation

Deprecate:

- datasets;
- columns;
- APIs;
- schemas;
- transformations.

Include replacement and timeline.

---

## Migration

Support:

- dual publish;
- view compatibility;
- adapter;
- consumer cutover.

Avoid indefinite dual maintenance.

---

## Data lifecycle

Every dataset should have lifecycle.

Example:

```text
experimental
supported
deprecated
retired
```

---

## Retention

Define retention per dataset/classification.

Do not choose storage defaults as policy.

---

## Expiry

Temporary datasets should expire automatically where safe.

---

## Deletion

Deletion should propagate intentionally.

Targets may include:

- source copy;
- derived tables;
- search indexes;
- feature stores;
- caches;
- exports;
- snapshots;
- backups where policy permits.

---

## Tombstones

For append/stream models, tombstones may communicate delete.

Consumers must process them.

---

## Right-to-delete workflows

If regulatory/privacy deletion applies, track:

- request;
- identity;
- datasets affected;
- completion;
- exceptions.

---

## Backup retention

Backups may retain deleted data temporarily.

Document restoration/deletion implications.

---

## Legal hold

Legal hold may override ordinary deletion.

Keep scope and authorization explicit.

---

## Archival

Archive data when long-term retention is required but frequent query is not.

Define restore process.

---

## Cost architecture

Data-platform cost often grows with:

- storage;
- scans;
- compute;
- shuffle;
- network egress;
- replication;
- streaming throughput.

Expose cost drivers.

---

## Cost attribution

Where useful, attribute cost to:

- domain;
- team;
- dataset;
- environment;
- workload.

---

## Compute quotas

Bound interactive and batch workloads.

Avoid one analyst query consuming entire cluster capacity.

---

## Storage quotas

Set where self-service can create large datasets.

---

## Egress cost

Cross-region/cloud data movement may be expensive.

Make it visible.

---

## Query optimization

Provide tooling/guardrails for expensive queries.

Do not require every user to be a storage-engine expert.

---

## Workload classes

Separate:

- interactive;
- batch;
- streaming;
- critical;
- experimentation.

Use different resource policies.

---

## Priority

Critical production pipelines may need higher scheduling priority.

Do not let users self-assign highest priority without policy.

---

## Capacity planning

Model:

- ingest throughput;
- daily growth;
- query concurrency;
- storage;
- transformation compute;
- streaming lag.

---

## Small tenant / large tenant

Protect shared systems from hot domains.

Use quotas, workload groups, or dedicated resources where justified.

---

## Multi-tenancy

Define tenant isolation across:

- storage;
- compute;
- metadata;
- catalog;
- query;
- credentials;
- caches.

Do not rely only on database naming.

---

## Shared storage

Shared storage may be efficient.

Use row/column/catalog controls as needed.

---

## Dedicated storage

Use dedicated physical isolation for high-risk tenants when required.

Do not multiply systems without clear value.

---

## Shared compute

Use workload isolation and quotas.

---

## Compute impersonation

Query engines should preserve caller identity where possible.

Do not execute every query as one superuser and rely only on UI filtering.

---

## Metadata isolation

Catalogs can leak sensitive dataset names/columns.

Authorize metadata visibility.

---

## Lineage privacy

Lineage can reveal sensitive system relationships.

Control access.

---

## Notebook environments

If platform provides notebooks:

- identity;
- secrets;
- network;
- package install;
- data access;
- idle timeout;
- cleanup.

Notebooks are powerful execution environments.

Do not treat them as harmless query UIs.

---

## Arbitrary code execution

Data processing often runs arbitrary user code.

Use isolation appropriate to risk.

Avoid running untrusted jobs with broad platform credentials.

---

## Package dependencies

User-defined transformations introduce supply-chain risk.

Compose with `software-supply-chain.md`.

---

## Sandboxing

Use container/process/VM isolation as appropriate.

Do not rely solely on language-level restrictions for hostile code.

---

## Network egress from jobs

Restrict when sensitive data should not be exfiltrated.

---

## Temporary storage

Jobs may spill sensitive data.

Control:

- location;
- encryption;
- cleanup.

---

## Secrets in jobs

Use workload identity/secret references.

Do not embed database passwords in notebooks or DAG files.

---

## Data sharing with SaaS

Treat external SaaS destinations as independent trust domains.

Review:

- data class;
- retention;
- training/reuse;
- region;
- deletion.

---

## Metadata/catalog ingestion

Catalog crawlers may need broad read access.

Scope and isolate them.

---

## Automatic discovery

Discovery is useful but can expose sensitive metadata.

Apply authorization.

---

## Lineage collection

Collect lineage from:

- query engine;
- orchestrator;
- transformation framework;
- event metadata.

Avoid manually maintained lineage where automation is feasible.

---

## Lineage completeness

Be honest about gaps.

Do not display partial lineage as complete truth.

---

## Data quality platform

A shared quality capability may provide:

- assertions;
- profiling;
- anomaly detection;
- reports.

Do not centralize domain-specific quality ownership.

---

## Validation libraries

Provide reusable validation where helpful.

Compose with `library-sdk.md`.

---

## Data contracts in CI

Validate schema/contract changes before merge where possible.

Do not wait for production consumers to fail.

---

## Query compatibility tests

Test critical views/datasets against consumer queries where practical.

---

## Pipeline contract tests

Validate:

- input schema;
- output schema;
- quality invariants;
- partition completeness.

---

## Test data management

Use representative non-sensitive test data.

Avoid tiny fixtures that hide scale/skew issues.

---

## Synthetic data

Synthetic data is useful but may miss real-world edge cases.

Document limitations.

---

## Production sampling for tests

If using sampled production data:

- minimize;
- mask;
- authorize;
- expire.

---

## Testing strategy

### Unit tests

Test transformation logic.

### Schema tests

Validate input/output contracts.

### Integration tests

Exercise storage/query/orchestration.

### Data-quality tests

Test semantic invariants.

### Security tests

Test tenant/access boundaries.

### Recovery tests

Test replay/backfill/rebuild.

### Scale tests

Test representative data volume/skew.

### End-to-end tests

Exercise source to consumer.

---

## Golden datasets

Use small versioned golden datasets for deterministic transformations.

---

## Property-based testing

Useful properties include:

- transform is deterministic;
- deduplication is idempotent;
- forbidden tenant rows never appear;
- round-trip serialization preserves value.

---

## Fuzzing

Consider fuzzing:

- parsers;
- codecs;
- schema validators;
- file readers.

---

## Failure injection

Simulate:

- source unavailable;
- storage unavailable;
- partial partition;
- schema mismatch;
- bad record;
- network timeout.

Confirm safe behavior.

---

## Backfill tests

Run bounded backfill.

Verify no duplicate/corrupt output.

---

## Restore tests

For critical datasets, restore snapshots/backups.

Validate downstream consumption.

---

## Rebuild tests

Where derived datasets are rebuildable:

```text
authoritative source
    +
transformation version
    ->
same semantic output
```

within defined nondeterminism limits.

---

## Security tests

Test:

- unauthorized dataset query;
- cross-tenant row access;
- restricted column access;
- export denial;
- masked/unmasked views.

---

## Data deletion tests

Delete a test subject/key.

Verify propagation through downstream derived stores where required.

---

## Observability

Track platform signals:

- ingest success;
- freshness;
- completeness;
- quality failures;
- query latency;
- query failures;
- storage growth;
- streaming lag;
- backfill activity;
- deletion workflow.

---

## Data quality observability

Keep process and data health separate.

Example:

```text
pipeline completed successfully
but
source partition missing
```

should not appear green.

---

## Freshness dashboards

Expose last successful/complete data time.

---

## Data-volume anomalies

Unexpected zero, drop, or spike can indicate ingest issues.

Use context-aware thresholds.

---

## Query telemetry

Track:

- expensive scans;
- failures;
- concurrency;
- tenant cost.

Respect privacy.

---

## Audit

Audit sensitive actions:

- access grants;
- exports;
- unmask;
- retention change;
- dataset deletion;
- admin policy.

---

## Access logs

For high-value datasets, record who accessed what.

Balance with privacy and scale.

---

## Data lineage as evidence

Lineage can support incident and compliance investigations.

Do not claim full compliance merely because lineage exists.

---

## Platform SLOs

Potential platform SLOs:

- ingest availability;
- query availability;
- metadata availability;
- freshness;
- pipeline completion.

Use actual consumer requirements.

---

## Dataset SLOs

Critical data products may define:

- freshness;
- completeness;
- quality.

Owners should agree to them.

---

## Incident response

Data incidents may include:

- stale data;
- wrong data;
- missing data;
- cross-tenant exposure;
- unauthorized export;
- deletion failure.

These differ from ordinary service outages.

---

## Data incident containment

Provide mechanisms to:

- quarantine;
- stop publish;
- revoke access;
- roll back dataset version;
- halt downstream propagation.

---

## Bad data rollback

If possible, repoint consumers to known-good snapshot/view.

Do not overwrite evidence of the incident.

---

## Contaminated derived data

Track which downstream datasets consumed bad input.

Lineage should support impact analysis.

---

## Platform control plane

A data platform may have control-plane resources for:

- dataset registration;
- access;
- schemas;
- jobs;
- policies;
- lifecycle.

Compose with `control-plane.md`.

---

## Declarative dataset resources

A platform resource may describe:

```yaml
dataset:
  owner: payments
  classification: confidential
  schema: ...
  retention: 90d
  freshness: 15m
```

Keep logical contract separate from physical provider implementation.

---

## Reconciliation

If the platform reconciles data resources:

- create/update/delete idempotently;
- verify ownership;
- preserve state;
- avoid destructive drift correction without safeguards.

---

## Data deletion as control-plane action

Deletion can be high-impact.

Require:

- exact target;
- scope;
- authorization;
- audit;
- confirmation/approval where needed.

---

## Data migration

Platform migrations may move datasets between:

- storage engines;
- schemas;
- regions;
- accounts.

Preserve contract and lineage.

---

## Dual read/write

Use temporarily for migrations where required.

Bound duration.

Do not leave indefinite dual-write complexity.

---

## Cutover

Define:

- source freeze or live sync;
- validation;
- consumer switch;
- rollback.

---

## Cross-region replication

Define:

- source of truth;
- lag;
- failover;
- residency;
- delete propagation.

---

## Multi-region query

Avoid accidental cross-region sensitive data movement.

---

## Disaster recovery

Recover:

- authoritative data;
- metadata;
- schemas;
- lineage where required;
- policies;
- credentials.

---

## Backup

Back up data that cannot be reconstructed.

Derived datasets may be rebuilt instead.

---

## Restore ordering

Restore dependencies in order.

Example:

```text
metadata/schema
    ->
authoritative data
    ->
derived data
    ->
indexes
```

---

## DR tests

Test representative recovery.

Do not call backups a DR strategy without restore evidence.

---

## Security threat model

At minimum consider:

- malicious producer;
- malicious consumer;
- cross-tenant query;
- compromised connector;
- poisoned input;
- secret leakage;
- unauthorized export;
- stale access grant;
- lineage spoofing;
- data exfiltration through query;
- untrusted notebook/job code.

---

## Malicious input

Treat external data as untrusted.

Validate:

- parser limits;
- file paths;
- compression;
- encodings;
- schema.

---

## Data poisoning

For analytical/AI consumers, poisoned data may alter decisions or models.

Preserve provenance and trust classification.

---

## Exfiltration through queries

Restrict query capabilities where sensitive datasets are exposed.

Use result limits/export controls.

---

## Inference risk

Aggregates may reveal sensitive individuals through small-group queries.

Use privacy thresholds where relevant.

---

## SQL injection

Platform-generated SQL must use safe parameterization.

Do not concatenate untrusted identifiers/expressions without validation.

---

## UDFs

User-defined functions expand code execution.

Sandbox and govern them.

---

## Serialization safety

Avoid unsafe deserialization formats.

Use data-only encodings.

---

## Supply chain

Compose with `software-supply-chain.md`.

Protect:

- transformation code;
- connectors;
- query engines;
- orchestration plugins;
- base images;
- schema tooling.

---

## Connector trust

Connectors often have broad source/destination access.

Use least privilege and isolate them.

---

## Plugin trust

Review and pin plugins.

Do not allow arbitrary unmanaged plugins in shared execution engines.

---

## Change review triggers

Require stronger review for changes to:

- classification;
- access policy;
- retention;
- deletion;
- cross-region replication;
- schema breaking changes;
- shared transformation libraries;
- public datasets.

---

## Developer platform integration

A developer platform may expose data capabilities such as:

- database;
- object store;
- stream;
- warehouse dataset;
- analytics environment.

Keep data lifecycle explicit.

---

## Eventing platform integration

Streams/events may be ingestion sources.

Compose with `eventing-platform.md`.

---

## API platform integration

Data products may expose APIs.

Compose with `api-platform.md`.

---

## Observability platform integration

Observe data freshness/quality/lineage operations.

Do not send sensitive payloads to observability by default.

---

## AI platform integration

AI systems may consume curated datasets.

Require:

- provenance;
- classification;
- access;
- deletion semantics;
- eval/training boundaries.

---

## Feature platform

If serving ML features, define:

- offline/online consistency;
- freshness;
- point-in-time correctness;
- lineage.

Do not train on future information accidentally.

---

## Point-in-time correctness

For ML/analytics, joins must avoid leakage from future data.

---

## Dataset discoverability

Discovery should show enough to assess suitability.

Include:

- owner;
- description;
- schema;
- freshness;
- classification;
- quality;
- lineage.

---

## Documentation

Every supported dataset should have useful documentation.

Do not leave consumers to infer meaning from column names.

---

## Runbooks

Create runbooks for:

- stale dataset;
- failed ingest;
- schema break;
- bad backfill;
- cross-tenant exposure;
- deletion failure;
- query outage;
- storage pressure.

---

## Architecture diagrams

Useful views include:

### Flow

```text
Source -> Ingest -> Validate -> Transform -> Store -> Serve -> Consumer
```

### Trust

```text
Producer Identity -> Ingest Policy -> Dataset -> Consumer Authorization
```

### Lineage

```text
Source A ----\
              -> Transform X -> Dataset C
Source B ----/
```

### Lifecycle

```text
Create -> Publish -> Evolve -> Deprecate -> Retain/Delete
```

---

## Architecture decisions

Use ADRs for consequential decisions such as:

- lake/warehouse/lakehouse model;
- dataset contract;
- schema compatibility;
- lineage architecture;
- tenancy;
- retention;
- data-serving interfaces;
- backfill/reprocessing model.

---

## Recommended repository shape

Follow existing repository conventions first.

A generic data-platform repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── architecture/
│   ├── data-flows.md
│   ├── trust-boundaries.md
│   ├── domains.md
│   └── decisions/
├── contracts/
│   ├── datasets/
│   ├── schemas/
│   └── quality/
├── platform/
│   ├── ingestion/
│   ├── storage/
│   ├── processing/
│   ├── serving/
│   ├── metadata/
│   ├── lineage/
│   ├── quality/
│   └── access/
├── pipelines/
├── tests/
│   ├── schema/
│   ├── contract/
│   ├── integration/
│   ├── quality/
│   ├── security/
│   ├── recovery/
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

A data-platform repository should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-schema
test-contract
test-quality
test-integration
test-security
test-recovery
test-e2e
verify
```

These names are illustrative.

Use only commands the repository can actually implement.

---

## Acceptance criteria

A data platform is not complete because data can be queried.

Demonstrate the applicable subset of the following.

### Ownership and contracts

- datasets have owners;
- authoritative sources are identified;
- schemas are explicit;
- compatibility rules exist;
- consumer contracts are documented.

### Ingestion

- source identity is known;
- duplicate ingestion is safe;
- partial failure is visible;
- malformed input is quarantined or rejected;
- source credentials are least-privileged.

### Transformation

- transformation versions are recorded;
- outputs are attributable to inputs/code;
- reprocessing is safe;
- backfills are bounded and testable.

### Storage and serving

- logical dataset identity is independent of storage details where appropriate;
- access semantics are stable;
- query isolation exists;
- large exports are controlled.

### Governance

- classification is defined;
- row/column controls work where required;
- metadata/lineage is available;
- discoverability does not imply authorization.

### Quality

- freshness is measured;
- completeness is measured;
- semantic checks exist where required;
- process success is distinguished from data correctness.

### Security

- cross-tenant reads are denied;
- restricted fields are protected;
- exports are authorized;
- non-production access is controlled;
- untrusted processing code is isolated where needed.

### Lifecycle

- retention works;
- deletion propagates intentionally;
- schema deprecation has migration path;
- derived data can be rebuilt where promised.

### Resilience

- source/storage outage behavior is documented;
- backfill/replay recovery is tested;
- disaster recovery has restore evidence where required.

---

## Optional composition

Common combinations:

```text
platform-architecture + data-platform
```

For shared enterprise data capability architecture.

```text
data-platform + secure-data-pipeline
```

For implementation-level ingestion, transformation, retry, and data-movement
controls.

```text
data-platform + eventing-platform
```

For streaming ingestion, CDC, replay, and event-driven data products.

```text
data-platform + identity-platform
```

For workload/user identity and dataset-level authorization.

```text
data-platform + observability-platform
```

For freshness, quality, pipeline, and query telemetry.

```text
data-platform + software-supply-chain
```

For trusted transformation code, connectors, plugins, and data-platform
artifacts.

```text
data-platform + high-assurance
```

For stronger lineage, recovery, deletion, quality, and isolation verification.

```text
data-platform + ai-security
```

For governed AI/ML data use, provenance, poisoning resistance, and data-egress
controls.

---

## Anti-patterns

Avoid:

- calling one warehouse product "the data platform";
- ownerless datasets;
- raw CDC presented as stable domain contract accidentally;
- schema-less durable JSON with no evolution rules;
- silent type coercion;
- pipeline success treated as data correctness;
- production data copied to development by default;
- broad warehouse admin roles for convenience;
- tenant isolation implemented only through naming;
- sensitive data duplicated into every derived dataset;
- data classification lost during transformation;
- unbounded backfills;
- non-idempotent reprocessing;
- derived indexes treated as authoritative source;
- lineage that is manually maintained and obviously stale;
- catalog registration treated as semantic ownership;
- long-lived orphan staging data;
- one retention policy for every dataset;
- deletion requests that ignore derived stores;
- "anonymous" claims for merely pseudonymized data;
- external enrichment with no provenance/version;
- notebooks with unrestricted credentials and egress;
- query success used as proof that a dataset is trustworthy;
- backup claimed as DR without restore testing;
- every dataset forced into the same storage technology.

Do not mistake centralized storage for data architecture.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. data-platform purpose;
2. data domains and owners;
3. authoritative-source model;
4. dataset-contract/schema model;
5. ingestion modes and identity model;
6. validation/quarantine model;
7. transformation/versioning model;
8. provenance/lineage model;
9. storage/serving architecture;
10. access/tenant isolation model;
11. classification/privacy model;
12. quality/freshness/completeness model;
13. retention/deletion model;
14. backfill/reprocessing/recovery model;
15. capacity/cost model;
16. schema/quality/integration tests executed;
17. security/recovery/deletion tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a data platform is "governed", "trusted", "high quality",
"reproducible", "multi-tenant", or "compliant" merely because it has a catalog,
warehouse, lineage product, lakehouse, or access-control feature.

Describe the actual ownership, source authority, schema contracts, lineage,
quality evidence, access boundaries, lifecycle controls, and verification.

---

## Guiding principle

A good data platform should make data trustworthy because its history,
ownership, and constraints are explainable.

Sources should have authority.

Schemas should have contracts.

Transformations should have versions.

Derived data should have lineage.

Sensitive data should retain its classification.

Invalid data should not silently become valid.

Consumers should know freshness and quality.

Access should be explicit.

Deletion should propagate intentionally.

Reprocessing should be safe.

And when someone asks:

> Where did this value come from, who owns it, what changed it, how fresh is it,
> who may use it, and can we reproduce or delete it?

the platform should be able to answer with evidence rather than folklore.
