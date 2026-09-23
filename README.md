# Agent Control Plane

A reusable engineering constitution and architecture playbook for software-building agents.

This repository externalizes the engineering standards, architecture patterns, security constraints, and verification expectations that would otherwise be repeated in every prompt.

The goal is simple:

> Give the agent the task. Keep the engineering rules in the repository.

`AGENTS.md` defines the baseline engineering policy. Recipes describe **what kind of system is being built**. Profiles describe **how that system must be constrained, secured, or evaluated**. Orchestration prompts such as `SCAFFOLD` and `ELEVATE` determine whether the agent is designing greenfield architecture or improving an existing system.

The result is a composable control plane for engineering agents rather than a growing collection of one-off prompts.

---

## Why this exists

Coding agents are increasingly capable of producing substantial systems, but raw capability is not the same as consistent engineering judgment.

Without explicit policy, an agent may:

- introduce unnecessary dependencies;
- replace existing architecture rather than understand it;
- cargo-cult Kubernetes, microservices, service meshes, or cloud products;
- claim security from product selection rather than enforcement;
- confuse authentication with authorization;
- retry non-idempotent operations blindly;
- treat generated output as trusted;
- create tests that only prove the happy path;
- invent CI/CD, ownership, licensing, or operational assumptions;
- declare work complete without running the relevant verification.

Repeatedly explaining those expectations in chat is expensive and inconsistent.

This repository moves them into versioned engineering artifacts.

```text
task prompt
    |
    v
AGENTS.md
    |
    +--> SCAFFOLD or ELEVATE
    |
    +--> RECIPE(S)
    |
    +--> PROFILE(S)
    |
    v
implementation
    |
    v
test / evaluate / verify
    |
    v
evidence
```

The agent still makes implementation decisions.

It just does so inside an explicit engineering system.

---

# Core model

The repository is organized around four concepts.

## `AGENTS.md` — engineering constitution

`AGENTS.md` contains the baseline engineering policy that applies across projects.

It defines expectations around:

- repository discovery;
- precedence of local project rules;
- change discipline;
- architecture preservation;
- testing;
- dependency selection;
- security;
- supply-chain hygiene;
- reliability;
- observability;
- migrations;
- documentation;
- verification;
- completion evidence.

It intentionally avoids prescribing one language, framework, cloud, CI product, or runtime.

Local repositories may add their own `AGENTS.md`, `CONTRIBUTING.md`, ADRs, or project-specific instructions.

The general precedence model is:

```text
current task
    >
project-local instructions / ADRs / contracts
    >
executable project configuration
    >
global engineering policy
    >
external references
    >
agent defaults
```

The repository should never cause an agent to replace established local conventions merely because a generic default exists here.

---

## Orchestration prompts — how the work begins

The system is designed around two primary orchestration modes.

### `SCAFFOLD`

Use for greenfield work.

`SCAFFOLD` is allowed to make foundational choices when the repository does not already provide them.

Its job is to:

- classify the system being built;
- choose conservative defaults;
- establish architecture;
- create the smallest useful repository structure;
- define verification interfaces;
- compose the appropriate recipes and profiles.

The governing principle is:

> Scaffold is allowed to decide.

### `ELEVATE`

Use for brownfield work.

`ELEVATE` must discover the existing architecture before changing it.

Its preferred order is:

```text
improve existing
    >
configure existing
    >
add small missing capability
    >
introduce new platform/tool
    >
replace existing architecture
```

Its job is to:

- inspect the repository;
- identify what is already present;
- classify gaps as `PRESENT`, `PARTIAL`, `MISSING`, or `NOT APPLICABLE`;
- distinguish required work from recommendations;
- preserve sound local architecture;
- close justified gaps with the smallest coherent change.

The governing principle is:

> Elevate is required to discover.

---

# Recipes and profiles

The most important distinction in this repository is:

```text
RECIPE  = WHAT are we building?
PROFILE = HOW must it be constrained?
```

Keeping those separate prevents recipe explosion.

Instead of creating:

```text
secure-internet-facing-zero-trust-ai-kubernetes-service.md
```

compose:

```text
ai-service
+ kubernetes-workload
+ zero-trust-service
+ software-supply-chain
+ ai-evaluation
```

This also makes architecture decisions easier to reason about and reuse.

---

# Recipe catalog

## Platform architecture

These recipes operate at the architecture-of-systems level rather than the individual application level.

### [`platform-architecture.md`](./platform-architecture.md)

Foundational recipe for shared platforms.

Use it to define:

- consumers;
- platform contract;
- capability boundaries;
- control plane vs runtime/data plane;
- authoritative state;
- tenancy;
- identity;
- policy;
- self-service;
- golden paths;
- bootstrap;
- failure domains;
- scaling;
- migration;
- operating model.

Core principle:

> Define the platform contract before selecting the platform implementation.

A platform is a product with consumers, contracts, lifecycle, and an operating model. It is not merely a collection of infrastructure products.

---

### [`control-plane.md`](./control-plane.md)

Generic architecture recipe for systems that convert desired state into controlled action.

Use it for:

- resource control planes;
- fleet management;
- infrastructure management;
- policy control;
- orchestration;
- reconciliation systems.

It covers:

- desired vs observed state;
- authoritative state;
- reconciliation vs workflow;
- idempotency;
- unknown outcomes;
- delegation;
- policy enforcement;
- push vs pull;
- leader election;
- fencing;
- sharding;
- data-plane survivability;
- upgrade and recovery.

This is intentionally generic.

It is distinct from `platform-control-plane.md`, which focuses specifically on software build, artifact, promotion, and delivery control planes.

---

### [`developer-platform.md`](./developer-platform.md)

Architecture recipe for internal developer platforms and shared software-delivery capabilities.

It covers:

- platform-as-product;
- developer journeys;
- platform APIs;
- portals, CLIs, and SDKs;
- golden paths;
- paved roads;
- escape hatches;
- build and deployment capabilities;
- workload identity;
- observability;
- scorecards;
- templates;
- support;
- cost;
- adoption;
- migrations.

Core principle:

> The platform API is more important than the platform portal.

A developer platform should remove incidental complexity without hiding consequential complexity.

---

### [`identity-platform.md`](./identity-platform.md)

Shared architecture for human, workload, automation, device, and delegated identity.

It covers:

- trust domains;
- human identity;
- workload identity;
- federation;
- OIDC/OAuth/SAML/mTLS considerations;
- issuer and audience validation;
- token exchange;
- delegation;
- RBAC/ABAC/ReBAC;
- PKI;
- credential rotation;
- revocation;
- JIT access;
- break-glass;
- tenant isolation.

Core distinction:

```text
identity
!= authentication
!= authorization
!= entitlement
!= session
```

---

### [`api-platform.md`](./api-platform.md)

Architecture recipe for shared API exposure and governance.

It covers:

- public, partner, internal, and privileged APIs;
- gateway responsibilities;
- service responsibilities;
- authentication;
- resource-level authorization;
- schemas;
- API versioning;
- idempotency;
- retries and timeout budgets;
- quotas;
- abuse controls;
- caching;
- webhooks;
- compatibility;
- deprecation.

Core principle:

> Network routing is not API governance.

A gateway may enforce cross-cutting policy, but it should not become the place where application semantics go to die.

---

### [`eventing-platform.md`](./eventing-platform.md)

Architecture recipe for shared messaging, events, streams, and queues.

It covers:

- event vs command semantics;
- topic ownership;
- event envelopes;
- schemas;
- delivery guarantees;
- ordering;
- partitioning;
- consumer groups;
- retries;
- poison messages;
- dead-letter handling;
- replay;
- outbox/inbox patterns;
- backpressure;
- retention;
- multi-tenancy.

Core principle:

> “Exactly once” is an end-to-end claim, not a broker feature.

---

### [`observability-platform.md`](./observability-platform.md)

Architecture recipe for shared telemetry platforms.

It covers:

- metrics;
- logs;
- traces;
- events;
- audit;
- common resource identity;
- collection;
- routing;
- sampling;
- cardinality;
- redaction;
- alerting;
- SLOs;
- retention;
- query isolation;
- cost;
- meta-monitoring.

Core principle:

> Telemetry is production data, not exhaust.

More telemetry does not automatically mean better observability.

---

### [`data-platform.md`](./data-platform.md)

Architecture recipe for shared data capabilities.

It covers:

- data domains;
- authoritative sources;
- dataset contracts;
- ingestion;
- storage;
- transformation;
- schema evolution;
- metadata;
- lineage;
- quality;
- authorization;
- retention;
- deletion;
- backfills;
- serving;
- multi-tenancy;
- data egress.

Core principle:

> A data platform moves trust as well as bytes.

Queryable data is not automatically trustworthy data.

---

### [`ai-platform.md`](./ai-platform.md)

Architecture recipe for shared enterprise AI capability.

It covers:

- model gateways;
- model catalogs;
- external providers;
- self-hosted inference;
- prompt/version management;
- RAG;
- embeddings;
- agents;
- tool registries;
- evaluation;
- policy;
- data egress;
- quotas and cost;
- model/prompt/index/tool rollout;
- AI incident response.

Core principle:

> The AI platform should provide capabilities and enforcement primitives, not one giant agent framework that every application is forced to use.

---

## Architecture contracts

### [`protocol-design.md`](./protocol-design.md)

Recipe for designing durable machine-to-machine protocols.

Use it for:

- service protocols;
- controller/agent protocols;
- agent/tool protocols;
- streaming protocols;
- device protocols;
- plugin protocols;
- state synchronization.

It covers:

- framing;
- schemas;
- state machines;
- authentication;
- authorization;
- version negotiation;
- capability negotiation;
- ordering;
- idempotency;
- retries;
- cancellation;
- resumption;
- backpressure;
- replay resistance;
- downgrade protection;
- conformance testing;
- fuzzing.

Core principle:

> A protocol is a state machine with compatibility rules.

A serialization format is not a protocol.

---

## Application and component recipes

### [`secure-service.md`](./secure-service.md)

Production network service recipe.

Covers:

- API boundaries;
- validation;
- authn/authz;
- multitenancy;
- dependency behavior;
- persistence;
- health;
- lifecycle;
- observability;
- SSRF;
- webhook security;
- container/runtime security;
- supply chain;
- testing.

---

### [`ai-service.md`](./ai-service.md)

Production AI-backed service recipe.

Core model:

```text
request
    ->
validated input
    ->
model invocation
    ->
untrusted model output
    ->
deterministic validation/policy
    ->
response or action
```

Use for applications that call models but are not primarily autonomous tool-using agents.

---

### [`agentic-system.md`](./agentic-system.md)

Recipe for tool-using or action-capable AI systems.

Core model:

```text
model proposes action
    ->
deterministic policy / authorization
    ->
approval if required
    ->
tool invocation
    ->
effect
```

Covers:

- authority boundaries;
- tool classes;
- approvals;
- delegation;
- prompt injection;
- budgets;
- recursion;
- memory;
- exfiltration;
- auditing;
- incident controls.

---

### [`rag-system.md`](./rag-system.md)

Recipe for retrieval-augmented generation systems.

Covers:

- source ownership;
- ingestion;
- chunking;
- embeddings;
- indexing;
- authorization-before-retrieval;
- provenance;
- poisoning;
- freshness;
- deletion;
- citations;
- retrieval evaluation.

Core principle:

> Retrieval is a trust, authorization, provenance, and integrity problem — not merely a relevance problem.

---

### [`mcp-tool-server.md`](./mcp-tool-server.md)

Recipe for MCP-compatible tool servers and comparable capability servers.

Covers:

- tool schemas;
- narrow capabilities;
- delegated authority;
- deterministic authorization;
- side-effect classification;
- safe error models;
- idempotency;
- external backend trust;
- resource discovery;
- audit.

Core principle:

> Expose capability, not ambient authority.

---

### [`model-serving.md`](./model-serving.md)

Recipe for production inference/model-serving systems.

Covers:

- model artifacts;
- runtime identity;
- tokenizer/config;
- GPU/accelerator assumptions;
- capacity;
- batching;
- overload;
- model loading;
- rollouts;
- privacy;
- observability;
- model supply chain.

---

### [`secure-data-pipeline.md`](./secure-data-pipeline.md)

Recipe for individual secure data pipelines.

Use it below `data-platform.md`.

Covers:

- batch/stream ingestion;
- validation;
- transformation;
- sensitive data;
- provenance;
- replay;
- backfill;
- deletion;
- failure;
- observability.

---

### [`kubernetes-workload.md`](./kubernetes-workload.md)

Recipe for applications intentionally deployed to Kubernetes.

Covers:

- controller selection;
- immutable images;
- signals;
- graceful shutdown;
- probes;
- resource requests/limits;
- security context;
- service accounts;
- RBAC;
- networking;
- storage;
- autoscaling;
- rollout;
- failure behavior.

Core principle:

> Kubernetes manages desired state. Your application still owns application correctness.

---

### [`operator-controller.md`](./operator-controller.md)

Recipe for Kubernetes controllers/operators.

Covers:

- CRD/API design;
- reconciliation;
- spec/status;
- observed generation;
- idempotency;
- finalizers;
- ownership;
- leader election;
- webhooks;
- external systems;
- upgrade/version skew;
- real-cluster testing.

---

### [`library-sdk.md`](./library-sdk.md)

Recipe for reusable libraries and SDKs.

Covers:

- public API design;
- compatibility;
- dependency cost;
- errors;
- retries;
- cancellation;
- serialization;
- packaging;
- semantic versioning;
- release integrity.

Core principle:

> A library is an API contract before it is an implementation.

---

### [`cli.md`](./cli.md)

Recipe for production command-line interfaces.

Covers:

- command design;
- exit codes;
- stdout/stderr;
- JSON/NDJSON;
- config precedence;
- secrets;
- destructive operations;
- non-interactive execution;
- subprocess safety;
- retries/timeouts;
- packaging;
- testing the installed executable.

Core principle:

> A CLI has two audiences: humans and automation.

---

### [`security-tool.md`](./security-tool.md)

Recipe for security scanners, analyzers, policy tools, proxies, and security automation.

It distinguishes:

```text
PASS
FAIL
UNKNOWN
ERROR
NOT_APPLICABLE
```

rather than turning tool failure into false confidence.

Core principle:

> Findings are evidence, not truth.

---

### [`platform-control-plane.md`](./platform-control-plane.md)

Specialized recipe for secure software-delivery control planes.

It separates:

```text
BUILD PLANE
source -> build -> artifact -> provenance/signing

DEPLOY PLANE
desired state -> promotion -> policy -> rollout -> runtime
```

Its immutable artifact identity forms the contract between the planes.

Use this when designing build/promotion/GitOps/release architecture rather than a generic control plane.

---

# Profiles

Profiles add constraints to recipes.

They should not redefine the system archetype.

## [`software-supply-chain.md`](./software-supply-chain.md)

Hardening profile for:

- source integrity;
- dependencies;
- isolated/reproducible builds;
- provenance;
- SBOMs;
- signing;
- verification;
- artifact promotion;
- registry policy;
- release credentials;
- rebuilds;
- exception handling.

Core chain:

```text
trusted source
+ declared dependencies
+ controlled build
+ immutable artifact
+ bound evidence
+ trusted identity
+ verification
+ explicit promotion
```

Signing without verification is incomplete.

---

## [`zero-trust-service.md`](./zero-trust-service.md)

Zero-trust hardening profile for applications and platforms.

Core principle:

> Network location never quietly becomes identity.

It covers:

- human/workload identity;
- authentication;
- authorization;
- least privilege;
- delegation;
- resource-aware policy;
- segmentation;
- egress;
- SSRF;
- secret lifecycle;
- break-glass;
- audit.

The filename is historical; the profile can be composed with services, control planes, platforms, operators, and AI systems.

---

## [`ai-evaluation.md`](./ai-evaluation.md)

Behavioral evaluation profile for AI systems.

Core distinction:

```text
tests = deterministic software correctness
evals = probabilistic behavioral evidence
```

It covers:

- datasets;
- scenarios;
- regression corpora;
- model behavior;
- retrieval quality;
- tool use;
- security evaluation;
- LLM-as-judge;
- human evaluation;
- nondeterminism;
- cost;
- latency;
- release gates.

Critical security behavior should remain deterministically testable where possible.

---

# Composition examples

Recipes and profiles are intended to be stacked.

## Production service

```text
SCAFFOLD
+ secure-service
+ zero-trust-service
+ software-supply-chain
```

---

## Public API service

```text
SCAFFOLD
+ secure-service
+ api-platform
+ zero-trust-service
+ software-supply-chain
+ internet-facing
```

---

## Internal developer platform

```text
SCAFFOLD
+ platform-architecture
+ developer-platform
+ control-plane
+ identity-platform
+ observability-platform
+ software-supply-chain
+ zero-trust-service
```

---

## Software delivery platform

```text
SCAFFOLD
+ platform-architecture
+ platform-control-plane
+ developer-platform
+ software-supply-chain
+ zero-trust-service
```

---

## Event-driven platform

```text
SCAFFOLD
+ platform-architecture
+ eventing-platform
+ identity-platform
+ observability-platform
+ software-supply-chain
```

---

## Enterprise data platform

```text
SCAFFOLD
+ platform-architecture
+ data-platform
+ eventing-platform
+ observability-platform
+ identity-platform
+ zero-trust-service
+ software-supply-chain
```

---

## Enterprise AI platform

```text
SCAFFOLD
+ platform-architecture
+ ai-platform
+ identity-platform
+ data-platform
+ observability-platform
+ ai-evaluation
+ software-supply-chain
+ zero-trust-service
+ ai-security
```

Add:

```text
model-serving
```

when hosting models directly.

Add:

```text
rag-system
```

for retrieval.

Add:

```text
agentic-system
+ mcp-tool-server
```

for tool-using agents.

---

## Kubernetes operator

```text
SCAFFOLD
+ operator-controller
+ kubernetes-workload
+ zero-trust-service
+ software-supply-chain
```

---

## Protocol-driven control plane

```text
SCAFFOLD
+ platform-architecture
+ control-plane
+ protocol-design
+ identity-platform
+ zero-trust-service
+ high-assurance
```

---

# How to use this repository with an agent

The simplest workflow is to tell the agent which artifacts govern the task.

Example:

```text
Read AGENTS.md.

This is greenfield work.

Apply the platform-architecture, developer-platform, control-plane,
identity-platform, software-supply-chain, and zero-trust-service guidance.

Design the architecture first. Then implement the smallest coherent slice that
proves the platform contract.

Run the applicable verification and report actual evidence. Do not claim a
command, test, deployment, benchmark, or security property succeeded unless
you observed it.
```

For existing repositories:

```text
Read AGENTS.md.

Treat this as brownfield ELEVATE work.

Discover the existing architecture before changing it.

Apply secure-service, software-supply-chain, and zero-trust-service only where
applicable. Preserve sound existing conventions and do not introduce new
platform products unless a demonstrated gap justifies them.

Report:
- what was already present;
- what was partial;
- what was missing;
- what you changed;
- what you deliberately left unchanged;
- commands actually run;
- observed results;
- remaining assumptions.
```

The recipe text is intentionally normative enough that the task prompt can remain short.

---

# Suggested repository layout

A mature repository may use:

```text
agent-control-plane/
├── AGENTS.md
├── README.md
│
├── prompts/
│   ├── SCAFFOLD.md
│   └── ELEVATE.md
│
├── recipes/
│   ├── architecture/
│   │   └── protocol-design.md
│   │
│   ├── platform/
│   │   ├── platform-architecture.md
│   │   ├── control-plane.md
│   │   ├── platform-control-plane.md
│   │   ├── developer-platform.md
│   │   ├── identity-platform.md
│   │   ├── api-platform.md
│   │   ├── eventing-platform.md
│   │   ├── observability-platform.md
│   │   ├── data-platform.md
│   │   └── ai-platform.md
│   │
│   ├── application/
│   │   ├── secure-service.md
│   │   ├── security-tool.md
│   │   ├── cli.md
│   │   └── library-sdk.md
│   │
│   ├── ai/
│   │   ├── ai-service.md
│   │   ├── agentic-system.md
│   │   ├── rag-system.md
│   │   ├── mcp-tool-server.md
│   │   └── model-serving.md
│   │
│   └── infrastructure/
│       ├── secure-data-pipeline.md
│       ├── kubernetes-workload.md
│       └── operator-controller.md
│
└── profiles/
    ├── software-supply-chain.md
    ├── zero-trust-service.md
    └── ai-evaluation.md
```

The exact directory structure is optional.

The conceptual separation is more important than the filesystem.

A flat layout is perfectly valid if it is easier for the agent and humans to navigate.

---

# Design philosophy

## Architecture before products

The repository intentionally avoids instructions such as:

```text
use Kubernetes
use Kafka
use Backstage
use Istio
use Terraform
use Prometheus
```

unless the selected archetype already requires that technology.

Architecture should answer:

```text
what capability is required?
what contract exists?
what state is authoritative?
what is the trust boundary?
what fails together?
what must be versioned?
what evidence proves correctness?
```

Technology selection follows those answers.

---

## Tools are examples, not objectives

CNCF and other ecosystem landscapes are useful maps.

They are not shopping lists.

A mature platform architecture often contains fewer products than an immature one.

---

## Boring is a feature

Prefer:

```text
established > novel
standard library > unnecessary dependency
one coherent toolchain > overlapping tools
simple > speculative abstraction
explicit > magical
```

Complexity must earn its place.

---

## Small changes stay small

Agents should prefer the smallest coherent change that satisfies the requirement.

Do not combine a feature with:

- unrelated refactoring;
- framework replacement;
- dependency churn;
- speculative abstractions;
- repository-wide cleanup.

---

## Discovery before modification

Brownfield systems contain architecture even when nobody documented it.

Before modifying them, discover:

- language;
- framework;
- package manager;
- build;
- tests;
- CI/CD;
- deployment;
- public APIs;
- state;
- ownership;
- security boundaries;
- local instructions.

Executable configuration usually tells the truth better than assumptions.

---

## Security is architecture

Security should appear in normal system design:

```text
identity
authorization
state
failure
data
network
supply chain
operations
```

not as a final checklist.

---

## Network location is not identity

An internal subnet, VPN, namespace, hostname, cluster, or sidecar does not
automatically make a caller trusted.

Authentication and resource-aware authorization remain separate concerns.

---

## Immutable identity matters

Where artifacts, models, datasets, releases, or external resources are
important, prefer immutable or versioned identity.

Examples:

- source revision;
- artifact digest;
- model revision;
- schema version;
- dataset snapshot;
- policy version;
- prompt version.

Mutable names are useful aliases.

They are weak evidence.

---

## Signing without verification is incomplete

A signed artifact only becomes meaningful when a trusted enforcement point
verifies:

- identity;
- signature;
- policy;
- artifact binding.

The same principle applies to provenance and attestations.

---

## Failure is part of the design

Every architecture should answer:

```text
what happens if this dependency is unavailable?
what happens if the operation times out after succeeding?
what happens if the process restarts halfway through?
what happens if the caller retries?
what happens if state disagrees?
```

A system designed only for the happy path is unfinished.

---

## Unknown is not success

This is especially important for security tools, distributed systems, and AI.

Distinguish:

```text
PASS
FAIL
UNKNOWN
ERROR
NOT_APPLICABLE
```

Do not convert inability to determine into success.

---

## Probabilistic behavior gets deterministic boundaries

AI systems introduce nondeterministic components.

The engineering response is not to pretend they are deterministic.

Instead put deterministic controls around them:

```text
input validation
authorization
policy
tool boundaries
schemas
budgets
approval
evaluation
audit
```

The model may propose.

The system decides what is allowed.

---

## Evaluation is evidence

For AI systems:

```text
tests = deterministic software correctness
evals = probabilistic behavioral evidence
```

Neither replaces the other.

One aggregate score should never hide a catastrophic failure mode.

---

## Design patterns are vocabulary, not objectives

Patterns such as:

- Strategy;
- Adapter;
- Facade;
- Observer;
- State;
- Composite;
- Proxy;
- Bridge;
- Flyweight

are useful names for designs that emerge from requirements.

They are not architecture goals.

Do not add a pattern merely to make a design appear sophisticated.

---

# Verification contract

A central rule across this repository is:

> Never claim a command, test, benchmark, deployment, security control, or recovery path succeeded unless it was actually executed and the result was observed.

A final agent report should distinguish:

```text
implemented
verified
not verified
not applicable
assumed
deferred
```

Where applicable, recipes define verification interfaces such as:

```text
check
test
test-integration
test-security
test-policy
test-contract
test-failure
test-recovery
eval
eval-regression
build
verify
```

These names are examples.

The repository should use its actual native toolchain rather than inventing commands just to match this playbook.

---

# Completion evidence

For non-trivial work, an agent should finish with evidence.

A useful report includes:

1. what architecture or behavior was changed;
2. what existing architecture was preserved;
3. which recipes/profiles were applied;
4. important trust/state/failure assumptions;
5. tests added or changed;
6. commands actually executed;
7. observed results;
8. security or compatibility checks performed;
9. anything not verified;
10. deliberate omissions or follow-up work.

The goal is not ceremonial documentation.

The goal is to make confidence inspectable.

---

# Adding a new recipe

Create a recipe when a system archetype has a distinct architecture.

Good candidates answer:

> What are we building?

Examples:

- API platform;
- identity platform;
- operator;
- CLI;
- agentic system.

A recipe should generally include:

- purpose;
- core principle;
- architecture/invariants;
- boundaries;
- state;
- identity/security;
- failure behavior;
- lifecycle;
- operability;
- testing;
- verification interface;
- acceptance criteria;
- anti-patterns;
- completion evidence;
- composition guidance.

Avoid creating vendor-specific recipes unless the repository is intentionally
about that vendor.

---

# Adding a new profile

Create a profile when a constraint applies across multiple system archetypes.

Good candidates answer:

> How must this system be hardened, constrained, or evaluated?

Examples:

- software supply chain;
- zero trust;
- high assurance;
- hostile input;
- internet facing;
- AI security;
- AI evaluation;
- regulated data;
- air-gapped;
- multi-tenant.

A profile should not redefine the underlying architecture.

For example:

```text
internet-facing
```

should be composable with:

```text
secure-service
api-platform
ai-service
mcp-tool-server
```

rather than requiring separate recipes for each combination.

---

# What this repository deliberately does not do

This repository is not:

- a framework;
- a code generator;
- a cloud platform;
- an architecture certification;
- a replacement for ADRs;
- a replacement for local project documentation;
- a guarantee that generated code is correct;
- a list of mandatory CNCF products;
- a universal reference architecture.

It provides engineering policy and reusable architecture reasoning.

The agent still has to inspect the actual system.

Humans still own consequential architecture decisions.

---

# Working with local repositories

The recommended model is:

```text
global control plane
    +
project-local overlay
```

The reusable repository defines broad principles.

The target project defines:

- language/framework conventions;
- exact build commands;
- deployment topology;
- public API contracts;
- persistence;
- organization-specific policy;
- ADRs;
- ownership.

Do not copy generic guidance into a project when the project already expresses
the same rule more precisely.

---

# Example: greenfield platform architecture

A task might be:

```text
Build an internal platform that lets teams provision and operate event-driven
services across multiple environments.
```

A useful composition could be:

```text
AGENTS.md
+ SCAFFOLD
+ platform-architecture
+ developer-platform
+ control-plane
+ eventing-platform
+ identity-platform
+ observability-platform
+ software-supply-chain
+ zero-trust-service
```

The expected architecture work is then about:

- platform consumers;
- service contract;
- desired state;
- workload identity;
- event contracts;
- replay;
- quotas;
- observability;
- promotion;
- ownership;
- migration.

Not:

> Which twenty platform products can we install?

---

# Example: brownfield AI service

A task might be:

```text
Improve the security and production readiness of this existing LLM-backed API.
```

A useful composition:

```text
AGENTS.md
+ ELEVATE
+ ai-service
+ zero-trust-service
+ software-supply-chain
+ ai-evaluation
```

If it has tool execution:

```text
+ agentic-system
```

If it has retrieval:

```text
+ rag-system
```

The agent should inspect the existing implementation before adding new
frameworks or infrastructure.

---

# Example: architecture review

These documents can also be used without code generation.

Ask the agent to review a proposed architecture against a recipe.

Example:

```text
Review this proposed control-plane design against:

- platform-architecture.md
- control-plane.md
- protocol-design.md
- zero-trust-service.md

Do not redesign it from scratch.

Identify:
- sound existing decisions;
- missing contracts;
- state-authority ambiguity;
- trust-boundary issues;
- failure-mode gaps;
- compatibility risks;
- verification evidence that should exist.
```

This turns the repository into an architecture review system as well as an
implementation policy.

---

# Principles worth keeping visible

A few recurring principles run through nearly every file:

> Define the contract before selecting the implementation.

> Authentication is not authorization.

> Network location is not identity.

> Possession of a tool is not permission to use it.

> Signing without verification is incomplete.

> A timeout does not prove an operation failed.

> Exactly-once is an end-to-end claim.

> A portal is not a platform.

> A gateway is not API governance.

> A broker is not an eventing architecture.

> A warehouse is not a data platform.

> Metrics, logs, and traces do not automatically create observability.

> A model gateway is not an AI platform.

> A serialization format is not a protocol.

> Kubernetes cannot compensate for broken application semantics.

> A library is an API contract before it is an implementation.

> A CLI is an API for humans and automation.

> Model output is untrusted input.

> Retrieved content is not trusted instruction.

> The platform should expose capability, not ambient authority.

---

# Status and evolution

This repository is intentionally expected to evolve.

New architecture domains should be added when they represent reusable
engineering problems.

Existing recipes should be improved when:

- recurring failure modes are discovered;
- operational experience exposes missing constraints;
- architectures evolve;
- agent behavior reveals ambiguous instructions.

Avoid adding rules merely because a new tool or trend exists.

The question should remain:

> Does this make the engineering decision more explicit, repeatable, secure,
> testable, or operable?

If not, it probably does not belong here.

---

# License

Add the repository's chosen license explicitly.

Do not let an agent invent one.

---

# Contributing

Contributions should preserve the architecture of the repository:

- vendor-neutral by default;
- normative where correctness matters;
- explicit about applicability;
- composable;
- test/evidence oriented;
- skeptical of unsupported claims;
- clear about trust, state, failure, and lifecycle;
- conservative about introducing new tooling.

When proposing a new recipe, explain why it cannot be represented by an
existing recipe plus one or more profiles.

When proposing a new profile, explain why the constraint applies across
multiple recipes.

---

# Final principle

The purpose of this repository is not to make agents more opinionated.

It is to make engineering decisions more deliberate.

Give agents enough context to understand:

- the system being built;
- the architecture that already exists;
- the boundaries they must preserve;
- the failures they must anticipate;
- the security properties they must enforce;
- the evidence required before claiming success.

Then let them do the work.

Trust the agent with implementation.

Do not trust it to invent the engineering constitution every time.
