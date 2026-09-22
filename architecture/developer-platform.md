# developer-platform.md

> Recipe for scaffolding or elevating an internal developer platform (IDP) or
> comparable shared software-delivery platform.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md`, `control-plane.md`,
> `software-supply-chain.md`, `zero-trust-service.md`,
> `kubernetes-workload.md`, `operator-controller.md`, `cli.md`,
> `library-sdk.md`, and other recipes/profiles where appropriate.
>
> This recipe defines platform-product, contract, self-service, ownership,
> golden-path, identity, policy, lifecycle, operability, adoption, migration,
> and developer-experience expectations for platforms used by engineering teams
> to build, ship, operate, or govern software.
>
> It does not require Backstage, Kubernetes, Terraform, GitHub, GitLab, Argo,
> Crossplane, service catalogs, portals, scorecards, a specific cloud, or a
> specific CI/CD product.

## Purpose

Use this recipe when the repository or architecture defines a developer-facing
platform that provides reusable software-delivery capabilities to engineering
teams.

Typical examples:

- internal developer platforms;
- application platforms;
- software delivery platforms;
- platform engineering products;
- self-service infrastructure platforms;
- service onboarding platforms;
- golden-path ecosystems;
- developer portals backed by real platform APIs;
- enterprise software supply-chain platforms;
- multi-team deployment platforms;
- shared runtime platforms.

The goal is not to create a portal with buttons.

The goal is to provide a coherent, supported platform that reduces repeated
engineering work while preserving:

- clear ownership;
- explicit contracts;
- security boundaries;
- compatibility;
- operability;
- flexibility;
- migration paths;
- consumer autonomy where appropriate.

A developer platform should remove incidental complexity without hiding
consequential complexity.

---

## Core principle

The platform API is more important than the platform portal.

A useful architecture model is:

```text
Developer / Team
      |
      v
Developer Interface
  +---+---+---+
  |   |   |   |
 CLI API SDK Portal
  |   |   |   |
  +---+---+---+
      |
      v
Platform Contract
      |
      +--> service lifecycle
      +--> build
      +--> artifact
      +--> deploy
      +--> runtime
      +--> identity
      +--> secrets
      +--> observability
      +--> data
      +--> policy
      |
      v
Platform Capabilities
      |
      v
Infrastructure / Providers / Runtime
```

The portal is one client of the platform.

It should not be the only place where the platform exists.

---

## Architectural invariants

A developer platform SHOULD satisfy these invariants unless there is a
documented reason not to.

### 1. The platform has explicit developer consumers

Identify the actual teams and workloads the platform serves.

Do not design for an abstract "developer persona" that hides materially
different needs.

### 2. The platform exposes stable contracts

Consumers should depend on:

- APIs;
- resource models;
- CLI behavior;
- SDKs;
- templates;
- service lifecycle semantics;
- policy rules.

They should not depend accidentally on internal implementation products.

### 3. The platform is a product, not a project

The platform needs:

- product ownership;
- roadmap;
- support;
- lifecycle;
- adoption strategy;
- consumer feedback.

Do not treat platform launch as completion.

### 4. Self-service is bounded by policy

Self-service reduces coordination.

It does not remove:

- authorization;
- quotas;
- approvals;
- policy;
- audit.

### 5. Golden paths are defaults, not magic

A golden path should encode good defaults and reduce repeated decisions.

It should not hide essential runtime, cost, data, or security behavior.

### 6. Platform and application ownership are separate

The platform should not silently become responsible for:

- business logic;
- application-level authorization;
- product-specific data semantics;
- application SLOs.

Responsibility must be explicit.

### 7. The platform minimizes cognitive load

Consumers should not need to know every underlying product.

But consequential choices must remain visible.

### 8. Platform changes preserve consumers

Platform APIs, templates, policies, build behavior, and deployment semantics
are compatibility-sensitive.

### 9. Escape hatches are designed

Some teams will need capabilities outside the paved road.

Define how that happens safely.

### 10. Platform success is measured through consumer outcomes

Infrastructure uptime alone is not sufficient.

Measure:

- time to first success;
- deployment lead time;
- failure/recovery friction;
- adoption;
- support burden;
- policy friction.

### 11. The platform is operable without heroics

Common actions should be:

- observable;
- documented;
- automatable;
- recoverable.

### 12. The platform can be exited

Consumers should have migration/export paths where reasonable.

Avoid unnecessary organizational lock-in.

---

## Platform intent

Start with a concise statement:

```text
For <engineering consumer>,
the platform provides <capabilities>,
through <developer contract>,
while owning <platform responsibilities>,
and leaving <consumer responsibilities> to the team.
```

Example:

```text
For product teams, the developer platform provides repeatable build,
deployment, runtime integration, identity, observability, and policy through
a declarative service contract. The platform owns delivery mechanics and
runtime integration; teams retain business logic, application data semantics,
and service-level authorization.
```

---

## Consumer segmentation

Different teams may need different platform contracts.

Common segments:

- application teams;
- infrastructure teams;
- data teams;
- security teams;
- ML/AI teams;
- mobile/frontend teams;
- third-party developers;
- platform extensions.

For each segment define:

- required capabilities;
- maturity;
- scale;
- support expectations;
- risk profile;
- required exceptions.

Do not force one generic workflow onto every engineering discipline.

---

## Developer journeys

Map real developer journeys.

Typical journeys:

```text
discover platform
    ->
create service
    ->
configure
    ->
build
    ->
test
    ->
deploy
    ->
observe
    ->
promote
    ->
operate
    ->
decommission
```

Other journeys may include:

- request database;
- rotate credentials;
- add dependency;
- expose API;
- onboard new team;
- create environment;
- roll back release;
- investigate incident.

Architecture should support the full lifecycle, not only scaffolding.

---

## Time to first success

Measure how long it takes a new consumer to complete a meaningful first
workflow.

Example:

```text
new team
    ->
authenticated
    ->
service created
    ->
deployed
    ->
observable
```

Do not optimize only for "time to generate repository".

---

## Platform contract

The platform contract may include:

- service definition;
- application manifest;
- deployment API;
- environment API;
- build contract;
- artifact contract;
- runtime contract;
- identity contract;
- observability contract;
- policy contract.

Keep it coherent.

Avoid one YAML file with hundreds of unrelated platform knobs.

---

## Platform API

Prefer APIs that represent developer intent.

For example:

```yaml
service:
  runtime: http
  size: medium
  exposure: internal
  data_classification: confidential
```

rather than forcing every team to understand:

- load balancers;
- ingress classes;
- cloud IAM policies;
- container scheduling details;
- monitoring collectors.

But do not hide consequential semantics such as:

- internet exposure;
- data residency;
- privileged runtime;
- cost class.

---

## Declarative service contract

A declarative service/resource contract is often useful.

It may define:

- owner;
- runtime type;
- build inputs;
- deployment environments;
- network exposure;
- dependencies;
- identity;
- data sensitivity;
- operational tier.

Keep it versioned.

---

## Contract ownership

Every contract needs an owner responsible for:

- evolution;
- compatibility;
- documentation;
- migration.

Do not create orphaned platform schemas.

---

## Contract stability

Classify fields as:

```text
stable
experimental
deprecated
internal
```

Consumers should know what can change.

---

## Platform interface layers

A mature platform may expose:

- API;
- CLI;
- SDK;
- portal;
- Git-based configuration;
- automation hooks.

These should converge on the same underlying contract.

Avoid one behavior in portal and another through CLI.

---

## Portal

Use a portal for:

- discovery;
- navigation;
- visualization;
- onboarding;
- self-service initiation.

Do not embed unique platform behavior only inside portal code.

The portal should call platform APIs or write platform contracts.

---

## CLI

A CLI is useful for:

- automation;
- scripting;
- local workflows;
- debugging;
- CI.

Compose with `cli.md`.

Keep it aligned with the same platform resource model.

---

## SDK

SDKs can help when platform workflows need programmatic integration.

Compose with `library-sdk.md`.

Do not create SDKs before there is a stable API worth wrapping.

---

## Git-based interfaces

Git can be an effective desired-state interface.

Use it intentionally.

If Git is authoritative, define:

- repository ownership;
- validation;
- review;
- merge semantics;
- reconciliation;
- emergency path.

Do not mix direct UI/API writes into the same state without conflict semantics.

---

## Golden paths

A golden path should encode:

- supported defaults;
- required controls;
- common integrations;
- known-good lifecycle.

Examples:

- secure service;
- batch worker;
- event consumer;
- AI service.

Golden paths are not necessarily templates.

They may be:

- platform APIs;
- generators;
- examples;
- reference architectures;
- policies.

---

## Paved road model

Classify platform choices:

```text
REQUIRED
SUPPORTED
RECOMMENDED
EXPERIMENTAL
EXCEPTION-ONLY
UNSUPPORTED
```

This gives teams a clear risk/support model.

---

## Mandated controls

Mandatory controls should be few and justified.

Examples may include:

- identity;
- artifact integrity;
- security logging;
- production access policy.

Do not mandate implementation details merely for standardization aesthetics.

---

## Escape hatches

Design exceptions explicitly.

An escape hatch should answer:

- what may be bypassed;
- what controls remain mandatory;
- who approves;
- who owns support;
- how risk is recorded;
- when exception expires.

Avoid permanent hidden forks.

---

## Bring-your-own capability

Some platforms may allow teams to bring:

- custom runtime;
- database;
- CI;
- observability;
- network integration.

If allowed, define:

- integration boundary;
- minimum security requirements;
- support boundary;
- ownership.

---

## Platform capability map

Decompose the platform into capabilities.

Example:

```text
Developer Experience
    |
    +--> service creation
    +--> documentation/catalog
    +--> CLI/portal

Software Delivery
    |
    +--> source
    +--> build
    +--> test
    +--> artifact
    +--> promotion

Runtime
    |
    +--> deploy
    +--> network
    +--> compute
    +--> autoscaling

Security
    |
    +--> identity
    +--> secrets
    +--> policy
    +--> supply-chain verification

Operations
    |
    +--> logs
    +--> metrics
    +--> traces
    +--> incidents
```

Keep capabilities aligned to consumer needs.

---

## Capability ownership

Each capability should have:

- accountable owner;
- public contract;
- operating model;
- lifecycle state;
- dependencies.

Avoid "everyone owns it".

---

## Capability lifecycle

Capabilities may be:

```text
proposed
experimental
supported
default
deprecated
retired
```

Consumers should know lifecycle state.

---

## Platform service tiers

If different support/availability levels exist, define tiers.

Examples:

- sandbox;
- standard;
- critical.

Do not invent tiers if all consumers receive the same guarantees.

---

## Service catalog

A catalog may represent:

- services;
- ownership;
- APIs;
- dependencies;
- environments;
- documentation.

Treat it as metadata unless it is explicitly authoritative.

Do not assume catalog registration means ownership is correct.

---

## Catalog ownership

Prefer ownership metadata synchronized from trusted sources where possible.

Do not require duplicate manual updates across many systems.

---

## Scorecards

Scorecards can surface:

- missing ownership;
- outdated runtime;
- security controls;
- operational readiness.

Use them as evidence and prioritization aids.

Do not make arbitrary score percentages a substitute for engineering judgment.

---

## Scorecard controls

Each scorecard rule should have:

- rationale;
- owner;
- evidence source;
- remediation guidance;
- applicability.

Avoid vanity metrics.

---

## Templates

Templates can accelerate onboarding.

Keep templates:

- minimal;
- versioned;
- tested;
- upgradeable where possible.

Do not copy large frozen stacks into hundreds of repositories without a
maintenance plan.

---

## Template drift

Generated repositories will diverge.

Decide whether later platform evolution uses:

- upgrade tooling;
- central reusable actions/modules;
- dependency automation;
- migration campaigns.

Do not assume templates remain current after generation.

---

## Reusable components

Prefer shared versioned components for behavior that must stay current.

Examples:

- CI libraries;
- deployment APIs;
- base images;
- SDKs;
- policy packages.

Use scaffolding for local ownership, not as a substitute for centralized
capabilities.

---

## Source control integration

If the platform integrates with source control, define:

- repository discovery;
- webhook/event trust;
- permissions;
- branch protection assumptions;
- ownership.

Do not grant the platform broad write access to every repository by default.

---

## Build capability

The platform may provide a build capability.

Define:

- build inputs;
- dependency resolution;
- isolation;
- cache;
- artifact output;
- provenance;
- untrusted contribution behavior.

Compose with `software-supply-chain.md`.

---

## Build contract

Consumers should know:

- what files are inputs;
- how dependencies are declared;
- how builds are invoked;
- what artifact is produced;
- how reproducibility is handled.

Do not require undocumented CI magic.

---

## CI abstraction

A platform may abstract CI execution.

Do not hide:

- failure;
- logs;
- test results;
- artifact identity.

Teams need evidence.

---

## Delivery capability

Separate build from deployment.

A useful flow:

```text
source
    ->
build/test
    ->
immutable artifact
    ->
promotion
    ->
desired runtime state
    ->
reconciliation
```

Do not deploy mutable workspaces directly from CI.

---

## Environment promotion

Promote the same artifact across environments where practical.

Avoid environment-specific rebuilds unless required.

---

## GitOps

GitOps may be one implementation strategy.

If used:

- define authoritative desired-state repositories;
- validate before merge;
- reconcile continuously;
- avoid direct deployment bypass.

Do not require GitOps merely because the platform runs Kubernetes.

---

## Runtime capability

Define what the platform promises about runtime.

Examples:

- process execution;
- container runtime;
- network;
- scaling;
- health;
- identity;
- observability.

Do not expose every orchestrator primitive unless consumers need them.

---

## Kubernetes

Kubernetes may be an implementation detail.

Use `kubernetes-workload.md` for workload-level expectations.

Do not force every platform consumer to become a Kubernetes expert unless
that is an explicit platform choice.

---

## Runtime classes

If multiple runtime archetypes exist, expose them as platform concepts.

Examples:

```text
web-service
worker
scheduled-job
model-serving
stateful
```

Map these to implementation internally.

---

## Infrastructure provisioning

Self-service infrastructure may include:

- databases;
- queues;
- object storage;
- caches;
- secrets;
- DNS.

Use declarative lifecycle and stable identities.

Do not expose raw cloud-admin APIs unless the consumer model requires it.

---

## Infrastructure abstraction

Avoid lowest-common-denominator abstractions that make all providers equally
bad.

Expose platform-level capabilities with meaningful semantics.

---

## Data services

If providing data capabilities, define:

- ownership;
- backup;
- restore;
- deletion;
- data classification;
- access.

Do not provision persistent state without lifecycle semantics.

---

## Identity capability

The developer platform should integrate human and workload identity.

Separate:

```text
developer identity
platform identity
workload identity
provider identity
```

Do not propagate one broad human credential through automation.

---

## Workload identity

Prefer workload identity over static credentials.

Applications should receive the minimum identity needed for runtime.

---

## Delegation

When the platform acts on behalf of a developer, retain initiating identity and
delegated scope.

Do not collapse every action into "platform-service".

---

## Secrets capability

Provide a clear secret lifecycle.

Questions:

- how created;
- who may read;
- how workloads consume;
- how rotated;
- how revoked;
- how audited.

Avoid requiring developers to copy secrets through portals manually.

---

## Secret references

Prefer references over secret value duplication.

Do not store secret values in platform metadata, service catalogs, or
generated repositories.

---

## Policy capability

The platform may enforce:

- allowed runtime classes;
- artifact trust;
- deployment environments;
- network exposure;
- data classifications;
- resource quotas.

Keep policy explainable.

A deny should tell the consumer what failed and how to remediate.

---

## Policy timing

Apply policy early where possible.

Useful stages:

```text
authoring
pull request
build
admission
deployment
runtime
```

Do not defer every rejection until production deployment.

---

## Policy ownership

Each mandatory policy needs an owner.

Avoid orphaned controls no one can explain.

---

## Policy exceptions

Exceptions should be:

- scoped;
- approved;
- time-bounded;
- visible;
- reviewable.

Do not hard-code permanent bypass lists.

---

## Observability capability

The platform may provide:

- logs;
- metrics;
- traces;
- dashboards;
- alerting;
- SLO tooling.

Expose conventions and integration.

Do not promise "observability" merely because telemetry agents are installed.

---

## Default telemetry

A golden path may automatically provide:

- request metrics;
- runtime metrics;
- logs;
- trace context.

But application-specific business telemetry remains the application team's
responsibility.

---

## Alert ownership

Define who receives alerts.

Avoid platform teams becoming default on-call for every consumer application.

---

## SLO ownership

Platform should provide primitives and platform SLOs.

Consumers own application/product SLOs unless explicitly shared.

---

## API exposure capability

If the platform exposes services:

- define internal/external classes;
- TLS;
- identity;
- rate limits;
- routing;
- domain/DNS.

Compose with `api-platform.md` when available.

---

## Networking

Hide incidental network plumbing where possible.

Keep consequential exposure explicit.

A consumer should know if a service is:

```text
private
internal
partner
public
```

---

## Service discovery

Provide stable discovery semantics.

Do not couple applications to ephemeral instance addresses.

---

## Event capability

If the platform provides messaging/events, define:

- topic/queue ownership;
- schema;
- auth;
- replay;
- retention;
- dead-letter behavior.

Compose with `eventing-platform.md`.

---

## AI capability

If the platform provides shared AI services, expose:

- model access;
- quotas;
- prompt/eval integration;
- tool/agent boundaries;
- data egress rules.

Compose with `ai-platform.md`, `ai-service.md`, and `ai-security.md`.

---

## Developer environments

If the platform provides developer environments, define:

- lifecycle;
- identity;
- isolation;
- secrets;
- cost;
- expiration;
- network access.

Do not create permanent ungoverned cloud sandboxes by default.

---

## Preview environments

Preview environments can improve feedback.

Bound:

- lifetime;
- cost;
- data access;
- external exposure.

Do not copy production data automatically.

---

## Local development

The platform should support a practical local workflow.

Do not require remote production-like infrastructure for every code/test loop.

Provide:

- local mocks;
- lightweight services;
- dev containers;
- local emulators;
- test environments

only where they materially help.

---

## Production parity

Aim for parity in contracts and behavior, not identical infrastructure.

Local environments may use smaller/simpler implementations.

---

## Ownership model

Define ownership for:

- platform capability;
- application;
- data;
- security policy;
- runtime incident;
- deployment incident.

Use a responsibility matrix.

---

## Responsibility matrix

Example:

```text
Platform team owns:
- build platform
- deployment control plane
- runtime integration
- workload identity
- platform telemetry

Application team owns:
- application code
- business logic
- app-level authz
- application data semantics
- application SLOs

Shared:
- production incident triage
- capacity planning
- migration
```

Adjust to the actual organization.

---

## Incident responsibility

A platform should help determine whether an incident is:

```text
platform
consumer application
external dependency
shared
```

Do not make every issue a platform-team ticket by default.

---

## Support model

Define:

- support channel;
- response expectations;
- escalation;
- ownership transfer;
- office hours or self-service docs where useful.

---

## Operational transparency

Consumers should be able to see:

- platform incidents;
- degraded capabilities;
- maintenance;
- relevant change notices.

Avoid hidden platform changes that surprise teams.

---

## Status page

Internal status reporting may be useful at scale.

Do not create status tooling without operational ownership.

---

## Platform SLOs

Potential platform SLOs:

- API availability;
- build queue latency;
- deploy convergence;
- environment provisioning;
- identity issuance;
- telemetry ingestion.

Use consumer-relevant measures.

---

## Reliability tiers

Different capabilities may have different reliability needs.

Example:

```text
runtime identity       -> high criticality
new template creation  -> lower criticality
portal search          -> degradable
```

Do not give every component identical HA architecture.

---

## Degraded operation

Define what works when:

- portal is down;
- catalog is down;
- CI is down;
- deployment control plane is down;
- identity provider is degraded;
- telemetry backend is down.

The platform should degrade intentionally.

---

## Portal independence

Consumers should still be able to operate critical workflows without the portal
where practical.

The portal should not become a single point of failure for every capability.

---

## Control-plane design

Use `control-plane.md` for control semantics.

Developer platforms often contain several control planes:

- build;
- deployment;
- identity;
- environment;
- policy.

Do not combine them into one universal controller unless there is a strong
reason.

---

## Blast radius

Partition platform components according to risk.

Potential boundaries:

- environment;
- region;
- tenant;
- capability;
- cluster;
- account.

A single policy bug should not necessarily break every team globally.

---

## Tenant model

Define the tenancy unit.

Possible units:

- team;
- application;
- project;
- business unit.

Use it consistently for:

- access;
- quota;
- billing;
- ownership;
- isolation.

---

## Team boundaries

A team should not gain another team's:

- secrets;
- runtime control;
- environments;
- artifacts;
- logs

without explicit authorization.

---

## Platform administrators

Separate platform admin privileges from normal developer privileges.

Use least privilege and break-glass for highly sensitive operations.

---

## Production access

Prefer platform-mediated, auditable production actions.

Avoid permanent broad shell access as the default operating model.

---

## Break-glass access

Emergency access should be:

- explicit;
- time-limited;
- audited;
- revocable.

Do not make normal deployment dependent on privileged manual access.

---

## Environment separation

Define boundaries between:

- development;
- test;
- staging;
- production.

Consider separate:

- accounts/projects;
- credentials;
- clusters;
- policies;
- data.

Do not let local/dev credentials silently reach production.

---

## Data classification

The platform should understand enough data classification to apply relevant
controls.

Examples:

- public;
- internal;
- confidential;
- restricted.

Do not over-centralize business-specific data semantics.

---

## Compliance integration

Platform controls can provide evidence.

They do not make an application compliant automatically.

Avoid "compliance by platform" claims without scope.

---

## Security defaults

The paved road should make secure behavior easier than insecure behavior.

Examples:

- TLS on;
- workload identity on;
- signed artifacts verified;
- least-privilege default role;
- secrets not in repositories.

---

## Supply-chain integration

A mature developer platform should support:

```text
source
    ->
build
    ->
test
    ->
artifact
    ->
provenance
    ->
verification
    ->
promotion
```

Compose with `software-supply-chain.md`.

---

## Artifact model

Define:

- artifact identity;
- registry;
- immutability;
- retention;
- promotion;
- provenance.

Do not let platform deployment depend on mutable tags alone.

---

## Build isolation

Untrusted contribution builds should not receive production credentials.

Separate privileged release builds from untrusted PR execution where needed.

---

## Deployment identity

Build systems and deployment systems should use separate identities.

CI should not automatically become cluster administrator.

---

## Policy at deployment

Validate:

- artifact identity;
- signature/provenance where required;
- target environment;
- ownership;
- policy.

---

## Progressive delivery

The platform may offer:

- rolling;
- canary;
- blue/green.

Expose behavior clearly.

Do not claim exact traffic weighting if the implementation only changes replica
ratio.

---

## Rollback

Rollback should be:

- simple;
- documented;
- based on immutable artifact/config identity.

Do not depend on recreating an old build.

---

## Database migration integration

Do not pretend application schema changes are solved by platform deployment.

Provide patterns and guardrails.

Application teams remain responsible for data semantics.

---

## Cost model

Self-service creates cost.

Expose:

- major cost drivers;
- quotas;
- expensive resource classes;
- cleanup/TTL.

Do not make cost invisible until billing review.

---

## Cost attribution

Where useful, attribute by:

- team;
- service;
- environment;
- capability.

Avoid precise showback systems if nobody uses the data.

---

## Resource quotas

Bound:

- environments;
- build concurrency;
- compute;
- storage;
- AI tokens;
- expensive managed services.

Quota denial should be clear.

---

## Lifecycle automation

Automatically clean up resources with clear temporary lifecycle.

Examples:

- preview environments;
- ephemeral test databases;
- temporary credentials.

Do not automatically delete durable resources based on weak heuristics.

---

## Platform metadata

Keep metadata useful and bounded.

Potential metadata:

- owner;
- service;
- environment;
- tier;
- data class;
- lifecycle status.

Avoid dozens of mandatory fields with no consumer value.

---

## Metadata authority

Define which source owns each field.

Do not create competing ownership in:

- portal;
- YAML;
- CMDB;
- source repository.

---

## Developer documentation

Documentation should be task-oriented.

Key topics:

- onboarding;
- common workflows;
- platform contract;
- support;
- limits;
- exceptions;
- migration.

Do not make teams reverse-engineer the platform from pipelines.

---

## Reference applications

Maintain small reference applications for key golden paths.

Use them to validate:

- docs;
- templates;
- platform APIs;
- upgrades.

Do not make reference apps production-scale examples.

---

## Samples

Samples should be:

- small;
- current;
- secure by default;
- executable.

---

## Error quality

Developer experience depends heavily on errors.

A platform error should answer:

- what failed;
- why;
- owner;
- remediation;
- operation/request ID.

Avoid "platform provisioning failed".

---

## Policy errors

A policy denial should identify:

- policy;
- violated condition;
- safe remediation.

Do not make developers ask the platform team to interpret every denial.

---

## Platform diagnostics

Provide diagnostic tooling where complexity justifies it.

Potential checks:

- auth;
- repository config;
- platform connectivity;
- environment state;
- deployment status.

Keep diagnostics safe and redact secrets.

---

## Automation-first design

Every important portal action should be automatable where reasonable.

Avoid human-only click paths for repeatable platform operations.

---

## Human approval

Some high-impact actions may require approval.

Bind approval to:

- action;
- target;
- environment;
- revision;
- expiry.

Do not create generic permanent approvals.

---

## Change management

Platform changes affect many teams.

Classify changes:

```text
internal
backward-compatible
behavior-changing
migration-required
breaking
security emergency
```

Use change process proportionate to impact.

---

## Versioning

Version:

- platform APIs;
- resource schemas;
- templates;
- reusable CI components;
- SDKs.

Do not version the portal and assume everything beneath it is covered.

---

## Backwards compatibility

Preserve supported consumer contracts.

Where breaking change is necessary:

- communicate;
- provide migration tooling;
- run parallel support;
- track adoption.

---

## Deprecation

Every deprecation should include:

- deprecated capability;
- replacement;
- migration;
- support end;
- owner.

Do not leave obsolete paths indefinitely.

---

## Migration campaigns

Platform teams should expect migrations.

A migration campaign may include:

```text
inventory
    ->
impact assessment
    ->
automation
    ->
dry run
    ->
wave rollout
    ->
exception tracking
    ->
retirement
```

Do not rely on broadcast email alone.

---

## Consumer inventory

Know which consumers depend on a capability before changing it.

Catalogs and telemetry can help.

---

## Usage telemetry

Track platform usage at a level appropriate to privacy.

Potential measures:

- active teams;
- services;
- deploy frequency;
- capability adoption;
- deprecated API use.

Do not collect source code or command payloads merely for adoption analytics.

---

## Product metrics

Useful platform-product metrics may include:

- onboarding time;
- platform adoption;
- deploy success;
- support-ticket volume;
- exception rate;
- policy denial rate;
- migration completion.

Avoid vanity metrics such as number of portal page views.

---

## Feedback loops

Use:

- developer interviews;
- support issues;
- incident reviews;
- adoption data;
- migration friction.

Do not treat platform engineering as one-way standardization.

---

## Consumer satisfaction

Satisfaction matters but should be interpreted alongside operational evidence.

A developer may prefer an unsafe shortcut.

The platform still owns necessary controls.

---

## Platform governance

Define how architecture decisions are made.

Potential stakeholders:

- platform;
- security;
- application teams;
- SRE;
- data;
- architecture.

Avoid central committees approving trivial consumer actions.

Govern platform standards, not every individual deployment.

---

## Architecture review

Use review for consequential changes:

- new capability;
- tenancy model;
- identity;
- global policy;
- API break;
- major vendor dependency.

---

## ADRs

Use ADRs for decisions such as:

- contract model;
- Git as desired state;
- portal role;
- runtime abstraction;
- identity model;
- exception model;
- build/deploy separation.

---

## Vendor selection

Select products against platform capabilities.

Do not let one product redefine the whole developer contract unless that is
intentional.

---

## Build versus buy

Ask:

- is this capability differentiating;
- is the market mature;
- can the team operate it;
- what is switching cost;
- what contract can shield consumers.

---

## Vendor abstraction

Abstract only where there is real value.

Do not build a generic cloud API over every provider just to claim portability.

---

## Open standards

Prefer open protocols/formats where they improve:

- interoperability;
- portability;
- ecosystem support.

Do not choose standards solely for ideological reasons.

---

## Platform topology

Document major platform domains and trust boundaries.

A generic view:

```text
                 Developers
                     |
          +----------+----------+
          |          |          |
         CLI        API       Portal
          |          |          |
          +----------+----------+
                     |
               Platform API
                     |
       +-------------+-------------+
       |             |             |
     Build        Deploy       Environment
       |             |             |
    Artifact       Runtime      Data/Secrets
       |             |             |
       +-------------+-------------+
                     |
               Observability
```

Use actual capability boundaries, not vendor logos.

---

## Failure domains

Identify what fails together.

Examples:

- build system unavailable;
- deployment API unavailable;
- portal unavailable;
- artifact registry unavailable;
- runtime control plane unavailable.

Define consumer impact for each.

---

## Portal outage

A portal outage should ideally not prevent:

- running workloads;
- CLI/API access;
- incident recovery;
- emergency deployment

where other interfaces exist.

---

## Registry outage

Existing workloads should often continue.

New deploy/build may pause.

Define expected behavior.

---

## CI outage

Runtime should continue.

Do not couple running application health to CI availability.

---

## Identity outage

Identity failures can have broad impact.

Define:

- cached sessions;
- workload credential behavior;
- break-glass.

---

## Observability outage

Do not make telemetry backend outage crash applications.

Loss of telemetry should be visible but appropriately decoupled.

---

## Capacity planning

Model:

- developers;
- repositories;
- builds;
- deployments;
- runtime workloads;
- telemetry volume;
- platform API traffic.

Use actual scale assumptions.

---

## Scaling dimensions

Scale capabilities independently where possible.

A surge in builds should not overload the developer portal or identity system.

---

## Multi-region platform

If multi-region is needed, define:

- control-plane placement;
- runtime autonomy;
- artifact replication;
- identity;
- failover;
- data residency.

Avoid multi-region complexity without requirements.

---

## Disaster recovery

Identify critical platform state:

- desired deployment state;
- catalog/metadata if authoritative;
- identity mappings;
- configuration;
- policy;
- artifact metadata.

Prefer rebuildable components where possible.

---

## Backup and restore

Test restore for state that cannot be recreated.

Do not assume SaaS products are automatically backed up in the way you need.

---

## Bootstrap

Document how the platform is created from nothing.

Questions:

- where source lives;
- how foundational CI runs;
- how identity is established;
- how runtime is created;
- how platform itself is deployed.

Avoid circular dependencies.

---

## Dogfooding

The platform may deploy itself using its own capabilities after bootstrap.

Keep a minimal recovery/bootstrap path outside that dependency.

Do not make platform recovery impossible if the platform is down.

---

## Security threat model

Consider:

- compromised developer account;
- malicious repository;
- malicious pull request;
- compromised build runner;
- stolen deployment identity;
- tenant escape;
- malicious template;
- poisoned plugin;
- compromised platform admin;
- supply-chain substitution.

---

## Untrusted repositories

Do not assume all repository content is trusted.

PR builds may execute attacker-controlled code.

Separate privileges.

---

## Template trust

Templates can propagate vulnerabilities at scale.

Version, review, and test them.

---

## Plugin trust

If the platform supports plugins/extensions:

- define publisher trust;
- permissions;
- isolation;
- lifecycle;
- compatibility.

Do not give plugins platform-admin rights by default.

---

## Security review triggers

Stronger review should apply to changes affecting:

- workload identity;
- deployment authorization;
- production access;
- artifact verification;
- tenant isolation;
- secret distribution;
- global policy;
- build privilege.

---

## Compliance evidence

Platform automation can generate evidence such as:

- artifact provenance;
- approvals;
- deployment history;
- policy decisions.

Keep evidence tied to actual operations.

Do not generate checkboxes disconnected from enforcement.

---

## Testing strategy

### Unit tests

Test platform business logic and policy.

### Contract tests

Validate API/resource compatibility.

### Integration tests

Validate capability interactions.

### Template tests

Instantiate golden paths and verify outputs.

### Security tests

Test tenant and privilege boundaries.

### Failure tests

Test capability outages.

### Migration tests

Test old consumer contracts against new platform versions.

### End-to-end tests

Exercise real developer journeys.

---

## Golden-path tests

For each supported golden path:

```text
create
    ->
build
    ->
deploy
    ->
observe
    ->
update
    ->
rollback
    ->
delete
```

Validate the actual path.

---

## Portal tests

Test portal workflows, but do not let portal tests substitute for API/contract
tests.

---

## CLI/API parity tests

Where CLI and portal are wrappers over the same API, test semantic parity.

---

## Policy tests

Test:

- expected allow;
- expected deny;
- exception;
- expired exception.

---

## Tenant isolation tests

Attempt:

- cross-team read;
- cross-team deploy;
- cross-team secret access;
- cross-team log access.

Expect denial.

---

## Supply-chain tests

Test:

- untrusted PR cannot publish production artifact;
- unsigned/untrusted artifact blocked where required;
- immutable digest promoted.

---

## Failure tests

Simulate:

- artifact registry down;
- CI unavailable;
- deploy controller down;
- portal down;
- telemetry down.

Confirm documented degraded behavior.

---

## Migration tests

Run supported older templates/contracts against new platform releases.

Do not discover incompatibility from production teams first.

---

## Scale tests

Test:

- concurrent builds;
- deployment burst;
- large service inventory;
- policy evaluation load;
- portal/catalog search

where relevant.

---

## Acceptance path

A useful reference acceptance path:

```text
new team authenticates
    ->
discovers supported golden path
    ->
creates service
    ->
repository/config generated
    ->
build executes
    ->
tests pass
    ->
immutable artifact produced
    ->
artifact promoted
    ->
runtime reconciles
    ->
workload receives identity
    ->
telemetry visible
    ->
team updates service
    ->
rollout succeeds
    ->
rollback works
```

Also test a negative path:

```text
team attempts unauthorized production change
    ->
policy/authorization denies
    ->
no deployment side effect occurs
```

---

## Documentation

The platform should document:

- purpose;
- capabilities;
- supported golden paths;
- contracts;
- ownership;
- self-service;
- policy;
- exceptions;
- support;
- lifecycle;
- migration.

---

## Quickstart

The quickstart should demonstrate one meaningful platform journey.

Do not make it a long product-installation guide.

---

## Architecture documentation

Provide:

- context diagram;
- capability map;
- control/data-plane boundaries;
- identity/trust boundaries;
- platform/consumer responsibilities;
- authoritative-state map.

---

## Runbooks

Create runbooks for real platform incidents:

- build backlog;
- deploy stuck;
- registry outage;
- identity failure;
- portal outage;
- policy outage;
- secret rotation failure.

---

## Recommended repository shape

Follow existing repository conventions first.

A developer-platform repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── architecture/
│   ├── context.md
│   ├── capabilities.md
│   ├── responsibility-model.md
│   ├── trust-boundaries.md
│   └── decisions/
├── api/
├── contracts/
├── templates/
├── platform/
│   ├── build/
│   ├── deploy/
│   ├── runtime/
│   ├── identity/
│   ├── observability/
│   └── policy/
├── cli/
├── portal/
├── tests/
│   ├── contract/
│   ├── golden-path/
│   ├── integration/
│   ├── security/
│   ├── migration/
│   └── e2e/
├── docs/
│   ├── consumers/
│   ├── operations/
│   ├── migration/
│   └── runbooks/
└── <ecosystem build/dependency files>
```

Do not force monorepo structure if capabilities live in separate repositories.

---

## Verification interface

A developer-platform project should expose obvious commands or equivalent
interfaces for executable checks such as:

```text
check
test
test-contract
test-policy
test-golden-path
test-security
test-integration
test-migration
test-e2e
verify
```

Use only commands the repository can actually implement.

---

## Acceptance criteria

A developer platform is not complete because developers can click "Create
Service" in a portal.

Demonstrate the applicable subset of the following.

### Product

- explicit consumer groups exist;
- platform purpose/non-goals are documented;
- ownership/support model exists;
- platform roadmap/lifecycle exists.

### Contract

- platform API/resource contract is explicit;
- CLI/portal/SDK align to the same contract;
- versioning/deprecation rules exist;
- consumer responsibilities are clear.

### Golden paths

- supported archetypes are explicit;
- templates/defaults are versioned;
- golden path completes end to end;
- escape-hatch behavior is documented.

### Security

- developer/workload/platform identities are separated;
- production actions are authorized;
- tenant boundaries are enforced;
- secrets are not duplicated into unsafe metadata;
- untrusted builds do not inherit production authority.

### Delivery

- build and deploy responsibilities are distinct;
- artifacts are immutable;
- promotion uses artifact identity;
- rollback uses known-good artifacts.

### Operations

- platform SLOs exist where justified;
- capability outages have degraded behavior;
- consumer/platform incident ownership is defined;
- support channels exist.

### Developer experience

- onboarding path works;
- errors are actionable;
- automation interface exists;
- platform usage does not require portal-only workflows.

### Lifecycle

- capability lifecycle states are explicit;
- deprecated paths have migration;
- consumer inventory supports migration campaigns;
- exit path exists where appropriate.

### Verification

- golden-path E2E test exists;
- deny/security path exists;
- migration compatibility is tested;
- at least one platform dependency failure is tested.

---

## Optional composition

Common combinations:

```text
platform-architecture + developer-platform
```

For the broad platform product and capability architecture.

```text
developer-platform + control-plane
```

For declarative environment, deployment, or resource management.

```text
developer-platform + software-supply-chain
```

For build, artifact, provenance, signing, promotion, and deployment integrity.

```text
developer-platform + zero-trust-service
```

For workload identity, delegated developer authority, and explicit service trust.

```text
developer-platform + kubernetes-workload
```

For Kubernetes-backed runtime golden paths.

```text
developer-platform + operator-controller
```

For Kubernetes-native platform resource reconciliation.

```text
developer-platform + cli + library-sdk
```

For automation-friendly developer interfaces alongside a portal.

```text
developer-platform + ai-platform
```

For shared AI model, RAG, agent, tool, evaluation, and policy capabilities.

---

## Anti-patterns

Avoid:

- calling Backstage or any portal "the platform";
- portal-only workflows;
- product inventory presented as architecture;
- one YAML file exposing every infrastructure knob;
- a golden path that hides operational reality;
- forcing every team into one runtime without need;
- mandatory standards with no exception process;
- platform team owning application business logic;
- one global platform credential;
- CI with direct production admin rights;
- mutable deployment artifacts;
- templates copied forever with no upgrade strategy;
- scorecards with arbitrary percentages;
- service catalog treated as authoritative without evidence;
- self-service with unlimited quota;
- portal uptime used as the only platform SLO;
- teams required to understand every underlying Kubernetes/cloud primitive;
- abstraction layers whose only purpose is pretending vendors are interchangeable;
- platform adoption measured only by number of onboarded repositories;
- breaking platform changes with no migration tooling;
- deprecations announced without consumer inventory;
- requiring the platform itself in order to recover the platform.

Do not mistake centralized developer tooling for a developer platform.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. platform purpose;
2. consumer segments;
3. platform non-goals;
4. consumer/platform responsibility boundary;
5. platform contract;
6. capability map;
7. golden-path model;
8. self-service and exception model;
9. developer/workload/platform identity model;
10. build/artifact/deploy model;
11. runtime and environment model;
12. policy/security enforcement model;
13. observability/support model;
14. capacity/cost/quota model;
15. compatibility/deprecation/migration model;
16. golden-path and contract tests executed;
17. security/failure/migration tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a developer platform is "self-service", "secure", "easy to use",
"standardized", or "production-ready" merely because it contains a portal,
templates, pipelines, or Kubernetes.

Describe the consumer contract, ownership boundaries, enforced controls,
developer journeys, migration model, and actual evidence.

---

## Guiding principle

A good developer platform should make the right thing easier without making the
important thing invisible.

Developers should understand what the platform promises.

The platform team should understand what it owns.

Automation should work without a portal.

Golden paths should reduce repeated decisions.

Security controls should be built into the path.

Exceptions should be explicit.

Platform changes should preserve consumers.

Migration should be part of lifecycle, not an emergency.

And the platform should be judged by whether engineering teams can build,
ship, operate, and evolve software with less unnecessary complexity—not by how
many platform products have been installed.
