# ai-platform.md

> Recipe for scaffolding or elevating a shared AI platform.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md`, `control-plane.md`,
> `developer-platform.md`, `identity-platform.md`, `api-platform.md`,
> `data-platform.md`, `observability-platform.md`, `model-serving.md`,
> `ai-service.md`, `agentic-system.md`, `rag-system.md`,
> `mcp-tool-server.md`, `library-sdk.md`, and profiles such as
> `ai-security.md`, `ai-evaluation.md`, `software-supply-chain.md`,
> `zero-trust-service.md`, `high-assurance.md`, or `hostile-input.md` where
> appropriate.
>
> This recipe defines architecture expectations for platforms that provide
> shared model access, model serving, prompting, retrieval, agent/tool runtime,
> evaluation, policy, identity, observability, safety, cost, and lifecycle
> capabilities to multiple AI application teams.
>
> It does not require a specific model provider, model gateway, vector database,
> agent framework, prompt-management product, cloud, inference runtime, or AI
> observability vendor.

## Purpose

Use this recipe when the architecture provides reusable AI capabilities to
multiple teams, products, or workloads.

Typical examples:

- enterprise AI platforms;
- shared LLM platforms;
- model gateways;
- inference platforms;
- RAG platforms;
- agent platforms;
- AI developer platforms;
- model catalog platforms;
- AI policy/evaluation platforms;
- internal copilots platforms;
- multi-provider AI access layers;
- self-hosted model platforms;
- regulated AI platforms.

The goal is not to centralize every AI application into one framework.

The goal is to create a coherent platform where teams can answer:

- Which models may be used?
- Which providers may receive which data?
- Which model/version produced this output?
- How are prompts and policies versioned?
- How is model access authorized?
- What does the platform enforce centrally?
- What remains the application team's responsibility?
- How are RAG sources authorized?
- How are tools exposed safely to agents?
- How are evaluations run before release?
- How are model/prompt/tool changes promoted?
- How are cost, latency, and quotas controlled?
- How are incidents investigated?
- How can one provider or model be replaced without rewriting every product?
- What evidence supports the use of a model for a given risk class?

---

## Core principle

The AI platform should provide capabilities and enforcement primitives, not one
giant agent framework that every application is forced to use.

A useful architecture model is:

```text
AI Application Teams
        |
        v
Platform Contract
        |
        +--> model access
        +--> prompt/config
        +--> embeddings
        +--> retrieval
        +--> tools
        +--> evaluation
        +--> policy
        +--> telemetry
        +--> quotas
        |
        v
AI Platform
   +----+----+----+----+----+
   |         |         |    |
 Model     RAG      Agents Tools
 Access    Plane    Runtime Broker
   |         |         |    |
   +---------+---------+----+
             |
      Policy / Identity
             |
      Observability / Eval
             |
         Providers /
       Self-hosted Models
```

The platform should make AI capabilities easier to consume while preserving
application ownership of domain behavior.

---

## Architectural invariants

An AI platform SHOULD satisfy these invariants unless there is a documented
reason not to.

### 1. Model identity is explicit

Every invocation should be attributable to a specific model identity.

Model identity may include:

- provider;
- model name;
- version/revision;
- quantization;
- adapter;
- serving runtime.

Do not treat mutable marketing names as sufficient identity.

### 2. Model output is untrusted

Model output must not bypass:

- schema validation;
- authorization;
- business rules;
- safety policy;
- tool policy.

### 3. Provider trust is explicit

Each external or internal model provider is a separate trust domain.

Do not assume all providers have equivalent data-handling, retention, or
security properties.

### 4. Data egress is intentional

Sensitive prompts, retrieved context, attachments, and tool results should not
leave approved trust boundaries accidentally.

### 5. Authorization remains deterministic

Models may assist decisions.

They should not be the sole authority for:

- access control;
- tenant isolation;
- credential scope;
- destructive action approval.

### 6. Prompts/configuration are versioned assets

System prompts, templates, safety config, tool definitions, and retrieval
settings materially affect behavior.

Treat them as release artifacts.

### 7. Evaluation is part of release evidence

Model/prompt/tool/retrieval changes should be evaluated before broad rollout.

### 8. Cost and capacity are bounded

Tokens, requests, GPU capacity, retrieval, and tool execution are finite
resources.

Self-service must remain bounded.

### 9. Tools expose capabilities, not ambient authority

Possession/discovery of a tool does not imply permission to invoke every action
the backing system can perform.

### 10. Retrieval preserves authorization and provenance

The platform should not retrieve unauthorized content merely because it is
relevant.

### 11. Model/provider failures are expected

Applications should have explicit behavior for:

- timeout;
- provider outage;
- refusal;
- malformed output;
- quota exhaustion;
- model unavailability.

### 12. AI platform controls are observable

Important model, prompt, policy, retrieval, and tool behavior should produce
evidence without requiring hidden chain-of-thought.

---

## Platform intent

Start with a concise platform intent statement.

Example:

```text
For product and engineering teams, the AI platform provides approved model,
retrieval, agent, tool, and evaluation capabilities through stable APIs and
SDKs while enforcing identity, data-egress, policy, quota, and release
controls. Application teams retain domain logic, user experience, and
application-level authorization.
```

Define non-goals.

Examples:

- platform does not own every application prompt;
- platform does not determine business truth;
- platform does not centralize all domain authorization;
- platform does not force one agent framework;
- platform does not make all models interchangeable;
- platform does not guarantee model correctness.

---

## Consumer model

Identify AI platform consumers.

Typical consumers:

- application teams;
- data teams;
- ML teams;
- security teams;
- platform teams;
- research teams;
- automated agents;
- internal tools.

For each consumer define:

- allowed models;
- data classes;
- environments;
- quota;
- tool access;
- support expectations.

---

## Capability map

A mature AI platform may expose capabilities such as:

```text
Model Access
  |
  +--> external provider access
  +--> self-hosted inference
  +--> model routing
  +--> model catalog

Knowledge / RAG
  |
  +--> ingestion
  +--> embeddings
  +--> retrieval
  +--> citations
  +--> source authorization

Agentic
  |
  +--> orchestration
  +--> tool registry
  +--> approvals
  +--> budgets
  +--> memory

Evaluation
  |
  +--> datasets
  +--> harnesses
  +--> regression
  +--> security evals
  +--> release gates

Platform Controls
  |
  +--> identity
  +--> policy
  +--> quotas
  +--> telemetry
  +--> audit
```

Do not add capabilities merely because AI vendors advertise them.

---

## Platform contract

The platform contract may include:

- inference API;
- embeddings API;
- model catalog API;
- retrieval API;
- tool registry/API;
- evaluation API;
- prompt/config registry;
- quota API;
- policy interface;
- observability metadata.

Keep contracts stable.

Do not expose provider implementation details unnecessarily.

---

## Model access plane

The model access plane may provide:

- provider routing;
- authentication;
- quotas;
- retries;
- timeout;
- normalization;
- logging/telemetry;
- policy.

Do not turn it into a giant application framework.

---

## Model gateway

A model gateway can centralize cross-cutting controls.

Potential responsibilities:

- provider credential brokering;
- allowed-model policy;
- quota;
- request limits;
- data classification checks;
- telemetry;
- fallback/routing.

Avoid placing application-specific prompt construction and domain behavior
inside the gateway.

---

## Gateway versus application

A useful split:

```text
platform:
  provider access
  identity
  quota
  global policy
  telemetry
  data-egress enforcement

application:
  prompt semantics
  domain validation
  user authorization
  business logic
  domain fallback
```

---

## Model catalog

A model catalog should describe:

- model identity;
- provider;
- modality;
- context limit;
- approved use classes;
- data-handling constraints;
- latency/cost characteristics;
- evaluation evidence;
- lifecycle state.

Do not make "available in provider API" equivalent to "approved for enterprise
use".

---

## Model lifecycle

A useful lifecycle:

```text
candidate
experimental
approved
default
deprecated
retired
blocked
```

Define criteria for transitions.

---

## Model approval

Approval should be scoped.

A model may be approved for:

- public data;
- internal data;
- low-risk summarization;
- code assistance;
- restricted data;
- agentic actions.

Avoid one binary approved/not-approved flag when risk differs materially by use.

---

## Model identity

Use immutable or sufficiently precise identity.

Examples:

- exact provider revision;
- model digest;
- deployment ID;
- quantization;
- adapter ID.

---

## Mutable model aliases

Aliases like:

```text
latest
default
production
```

may be useful operationally.

Resolve them to an exact model identity for evidence.

---

## Self-hosted models

Compose with `model-serving.md`.

The platform should define:

- model artifact;
- serving runtime;
- hardware;
- rollout;
- capacity;
- access;
- eval evidence.

---

## External providers

Each provider should have a documented trust profile.

Consider:

- region;
- retention;
- training/reuse;
- subprocessors;
- logging;
- content moderation;
- API stability;
- outage history;
- model lifecycle.

Do not assume contract terms are identical across providers.

---

## Provider onboarding

Before adding a provider define:

- owner;
- risk classification;
- credentials;
- network path;
- data policy;
- supported models;
- quotas;
- telemetry.

---

## Provider credentials

Keep provider credentials in the platform where possible.

Applications should use platform identity rather than receive raw provider keys.

---

## Credential brokering

The platform may broker short-lived provider access.

Avoid distributing long-lived vendor API keys to every application team.

---

## Provider isolation

Different environments or tenants may require separate provider projects/accounts.

Use isolation proportionate to risk.

---

## Provider routing

Routing may depend on:

- requested capability;
- model class;
- region;
- data classification;
- latency;
- capacity;
- cost;
- policy.

Do not route sensitive data to a cheaper provider that is not approved for it.

---

## Routing transparency

Applications should know enough to reason about behavior.

If model choice can change dynamically, record the resolved model identity.

---

## Model fallback

Fallback may improve availability.

Define:

- compatible capability;
- safety/eval equivalence;
- prompt compatibility;
- data policy;
- output differences.

Do not silently fail over to a materially different model for high-risk tasks
without evidence.

---

## Cross-provider fallback

Cross-provider fallback may change:

- privacy;
- latency;
- semantics;
- safety filters.

Treat it as policy-sensitive.

---

## Request contract

Inference requests should define:

- model/capability;
- input;
- parameters;
- response format;
- timeout;
- metadata.

Bound:

- input size;
- output tokens;
- attachments.

---

## Parameter policy

The platform may constrain:

- temperature;
- max tokens;
- tool use;
- response format.

Do not override application requirements arbitrarily.

---

## Structured output

Where applications depend on machine-readable output:

- use schema-constrained generation where available;
- validate output deterministically.

Do not trust "valid JSON" claims without parsing.

---

## Output validation

Validate:

- schema;
- allowed enum;
- ranges;
- references;
- authorization-sensitive values.

---

## Streaming

If streaming model output:

- propagate cancellation;
- validate partial/final state;
- handle provider disconnect.

Do not perform irreversible action from an incomplete stream.

---

## Timeout

Model requests must have finite timeouts.

Different models/capabilities may need different budgets.

---

## Retry

Retry only safe failures.

Model calls can be nondeterministic and costly.

Bound retries.

Do not automatically retry after an unknown side effect from downstream tool
execution.

---

## Idempotency

Pure inference may be safe to retry from a side-effect perspective, but output
may differ.

If exact reproducibility matters, record full invocation config.

---

## Context window

The platform should expose model context limits.

Applications must not assume arbitrary input fits.

---

## Truncation

Truncation policy should be explicit.

Do not silently drop security-critical instructions or retrieved evidence.

---

## Prompt architecture

Prompts are versioned software assets.

Track:

- template;
- version;
- owner;
- model compatibility;
- eval evidence.

---

## System prompts

System prompts may encode:

- role;
- constraints;
- output format;
- safety boundaries.

Do not put secrets into prompts.

---

## Prompt registry

A registry may store:

- templates;
- versions;
- owners;
- eval results;
- lifecycle.

Do not centralize every small application prompt if local ownership is simpler.

---

## Prompt publication

Treat production prompt changes like code/config changes.

Use:

- review;
- version;
- evaluation;
- rollout.

---

## Prompt variables

Validate and delimit variables.

Do not concatenate untrusted content ambiguously into instruction sections.

---

## Instruction provenance

Distinguish:

```text
system/platform instruction
application instruction
user instruction
retrieved content
tool output
external content
```

Untrusted content must not silently promote itself to higher authority.

---

## Prompt injection

Assume direct and indirect prompt injection is possible.

Controls should include:

- instruction separation;
- source provenance;
- tool authorization;
- output validation;
- data minimization;
- deterministic policy.

Prompt wording alone is not a complete defense.

---

## Retrieval plane

Compose with `rag-system.md`.

The platform may provide:

- ingestion;
- chunking;
- embeddings;
- vector/hybrid retrieval;
- authorization filtering;
- provenance;
- citations.

---

## Retrieval authorization

Authorization should happen before disclosure.

The model should not receive documents the caller is not allowed to access.

---

## Source identity

Every retrieved item should preserve:

- source;
- document identity;
- version;
- tenant;
- authorization context.

---

## Retrieval provenance

Model output should be attributable to the context supplied where citations or
grounding matter.

---

## Embedding service

A shared embedding service may provide:

- approved models;
- batching;
- quotas;
- versioning.

Treat embeddings as derived potentially sensitive data.

---

## Embedding version

Index compatibility depends on embedding model/version.

Record it.

Do not mix incompatible embeddings silently.

---

## Vector stores

A vector database is storage, not an authorization system by itself.

Enforce tenant/document access independently or with verified native controls.

---

## Hybrid retrieval

The platform may support:

- lexical;
- vector;
- metadata filters;
- reranking.

Do not force one retrieval strategy for all domains.

---

## Reranking

Track reranker identity/version.

Reranking materially affects retrieved context.

---

## Index lifecycle

Define:

- build;
- version;
- freshness;
- reindex;
- delete;
- rollback.

---

## Retrieval freshness

Consumers should know how stale indexes may be.

---

## Document deletion

Deletion must propagate to:

- source;
- chunks;
- embeddings;
- indexes;
- caches.

---

## Poisoning

Treat source content as potentially malicious.

Preserve source trust and provenance.

Do not let retrieved content redefine platform policy.

---

## Agent runtime

Compose with `agentic-system.md`.

A shared agent runtime may provide:

- planning loop;
- tool broker;
- policy;
- budgets;
- approval;
- memory;
- audit.

Avoid forcing applications into one monolithic agent framework.

---

## Agent identity

Distinguish:

- user identity;
- agent identity;
- platform identity;
- tool/server identity.

Do not let an agent act solely as the hosting platform's global identity.

---

## Delegated authority

An agent should receive only the authority necessary for the user/task.

Preserve the initiating user and delegated scope.

---

## Autonomy levels

Classify agent autonomy.

Example:

```text
0 = suggest only
1 = read-only tools
2 = reversible writes
3 = high-impact writes with approval
4 = bounded autonomous actions
```

Use actual organizational semantics.

---

## Budgets

Bound:

- turns;
- tool calls;
- recursion;
- wall time;
- tokens;
- spend;
- concurrent actions.

---

## Loop termination

Every autonomous loop needs termination criteria.

Do not let models recursively call tools without bounds.

---

## Agent memory

Memory should be scoped by:

- user;
- tenant;
- task;
- environment.

Treat memory as potentially poisoned or stale.

---

## Memory provenance

Record source/time for durable memory.

Do not turn model-generated guesses into durable facts without validation.

---

## Tool plane

Compose with `mcp-tool-server.md`.

The platform may provide:

- tool registry;
- schemas;
- discovery;
- auth;
- policy;
- approval;
- execution broker.

---

## Tool registry

A tool registry should identify:

- tool name;
- owner;
- schema;
- side-effect class;
- auth requirements;
- environment;
- lifecycle.

---

## Tool capability classes

A useful classification:

```text
READ_ONLY
REVERSIBLE_WRITE
HIGH_IMPACT_WRITE
IRREVERSIBLE
```

Policy and approval should reflect impact.

---

## Tool possession

Discovery or possession of a tool does not imply permission to invoke it.

Authorization must be checked at execution time.

---

## Generic tools

High-risk generic tools include:

- shell;
- arbitrary HTTP;
- unrestricted SQL;
- broad filesystem;
- cloud admin.

Prefer narrow typed tools.

---

## Tool outputs

Treat tool results as untrusted external content.

Do not let a webpage/tool response redefine system policy.

---

## Tool approval

High-impact tool calls should bind approval to:

- exact tool;
- arguments;
- target;
- environment;
- expiry.

---

## MCP

MCP can provide protocol interoperability.

It does not replace:

- authentication;
- authorization;
- tool safety;
- audit;
- tenant isolation.

---

## Tool credentials

Tools should use server-side or workload-scoped credentials.

Do not pass global platform credentials into model context.

---

## Sandbox

Generated code, shell commands, or browser actions may require sandboxing.

Isolate:

- filesystem;
- network;
- process;
- cloud credentials.

Use risk-appropriate controls.

---

## Browser agents

Browser automation can cross trust boundaries.

Protect:

- credentials;
- downloads;
- prompt injection;
- destructive actions.

---

## Code execution

If the platform offers code execution:

- isolate runtime;
- limit network;
- limit filesystem;
- limit CPU/memory/time;
- sanitize artifacts.

---

## Evaluation plane

Compose with `ai-evaluation.md`.

The platform may provide:

- datasets;
- harness;
- baseline management;
- regression;
- security/adversarial suites;
- reports;
- release gates.

---

## Eval as release evidence

A model/prompt/retrieval/tool change should produce evidence appropriate to
risk.

Do not use one generic quality score.

---

## Evaluation dimensions

Possible dimensions:

- task success;
- correctness;
- groundedness;
- citation accuracy;
- refusal behavior;
- policy compliance;
- tool selection;
- side-effect safety;
- latency;
- cost.

---

## Component evals

Evaluate separately:

- model;
- retrieval;
- tool use;
- policy;
- end-to-end workflow.

This makes regressions diagnosable.

---

## Security evals

Include:

- direct prompt injection;
- indirect prompt injection;
- data leakage;
- unauthorized tool use;
- cross-tenant access;
- secret exfiltration;
- malicious retrieved content.

---

## Adversarial datasets

Preserve real failures as regression cases.

Version the dataset.

---

## Holdout evaluation

Keep a holdout set for meaningful release decisions where appropriate.

Avoid tuning directly against every evaluation case.

---

## LLM-as-judge

Use carefully.

Track:

- judge model/version;
- rubric;
- calibration;
- variance.

Do not use a model judge for deterministic security assertions that can be
tested directly.

---

## Human evaluation

Use human review where domain judgment matters.

Capture rubric and reviewer context.

---

## Evaluation provenance

A result should identify:

- application version;
- model;
- prompt;
- tool definitions;
- retrieval/index version;
- dataset;
- evaluator;
- parameters.

---

## Release gates

Classify checks:

```text
blocking
warning
informational
```

Critical security failures should not disappear inside an average score.

---

## Policy plane

A shared policy capability may enforce:

- allowed models;
- data classes;
- provider use;
- tool classes;
- quota;
- environment;
- human approval.

---

## Policy inputs

Policy should consume trusted inputs.

Examples:

- caller identity;
- tenant;
- environment;
- data classification;
- tool side-effect class;
- model catalog metadata.

Do not allow the model to self-declare authorization-relevant policy inputs.

---

## Model policy

Model policy may state:

```text
public data -> models A/B/C
confidential data -> models A/B
restricted data -> self-hosted A only
```

Use actual organizational classifications.

---

## Policy enforcement points

Potential enforcement points:

- model gateway;
- retrieval API;
- tool broker;
- deployment/release gate;
- ingestion plane.

---

## Policy explanation

A deny should identify:

- policy;
- reason;
- remediation.

Avoid opaque "AI policy failed" errors.

---

## Exceptions

Exceptions should be:

- scoped;
- approved;
- time-bounded;
- attributable;
- reviewable.

---

## Identity

Compose with `identity-platform.md`.

Distinguish:

- developer;
- workload;
- end user;
- platform service;
- model provider;
- agent;
- tool.

---

## End-user identity

If end-user identity matters to retrieval/tool authorization, propagate it
through trusted delegated context.

Do not rely on user ID text placed in prompts.

---

## Workload identity

Applications should authenticate to platform APIs through workload identity
where practical.

Avoid shared provider API keys.

---

## Provider identity

The platform should know which provider endpoint/account it is using.

---

## Tool identity

Tool servers should authenticate the caller/platform and enforce resource
authorization.

---

## Multi-tenancy

Tenant isolation should cover:

- model quotas;
- prompt/config;
- retrieval indexes;
- memories;
- tool credentials;
- telemetry;
- eval datasets.

Do not rely solely on application-supplied tenant fields.

---

## Cross-tenant leakage

Test:

- prompt history;
- cache;
- embeddings;
- retrieved docs;
- tool results;
- logs.

AI platforms create many accidental cross-tenant surfaces.

---

## Caching

Caches may include:

- model responses;
- embeddings;
- retrieval results;
- prompt templates.

Cache keys must include security-relevant dimensions.

---

## Response cache

Do not share personalized or tenant-specific output across callers accidentally.

---

## Semantic cache

Semantic caching can leak information if isolation is weak.

Use tenant/user scopes as appropriate.

---

## Prompt cache

Provider-side prompt caching may have data-policy implications.

Document it.

---

## Session state

Session history can contain sensitive data.

Define:

- owner;
- retention;
- deletion;
- tenant scope.

---

## Memory

Long-term AI memory is durable data.

Treat it like a data product:

- schema;
- provenance;
- authorization;
- deletion;
- retention.

---

## Conversation retention

Do not store conversations indefinitely by default.

Use explicit retention policy.

---

## Data minimization

Send only necessary data to models.

Avoid entire database rows/documents if a smaller representation works.

---

## Redaction

Redact secrets/PII before provider egress where required.

Do not assume the model provider will remove them.

---

## Provider data retention

Track provider settings/contracts for:

- request logging;
- retention;
- training;
- abuse monitoring.

---

## Training use

Do not assume provider "no training" semantics without verifying applicable
terms/configuration.

---

## Data residency

Route model requests according to residency requirements.

Do not let dynamic routing move restricted data across regions silently.

---

## Encryption

Use transport encryption.

For self-hosted storage/indexes, apply at-rest encryption per data policy.

---

## Secrets

Never put:

- API keys;
- private keys;
- passwords

into model context unless the task explicitly requires a tightly controlled
secret-handling workflow.

Prefer tools that use server-side credentials.

---

## Prompt logging

Prompts can contain sensitive data.

Logging must be explicit and minimized.

---

## Output logging

Model outputs may also contain sensitive or generated secrets.

Apply the same classification approach as inputs.

---

## Tool argument logging

Tool calls can reveal:

- file paths;
- user data;
- credentials;
- SQL;
- URLs.

Redact appropriately.

---

## Observability

Compose with `observability-platform.md`.

Track:

- model requests;
- resolved model;
- latency;
- token usage;
- cost;
- errors;
- refusals;
- retries;
- retrieval latency;
- tool calls;
- policy denials.

---

## AI request identity

Use a stable request/operation ID.

Correlate:

```text
user request
    ->
model call
    ->
retrieval
    ->
tool call
    ->
final output
```

---

## Trace semantics

AI traces should capture observable execution, not hidden chain-of-thought.

Useful trace events:

- model invoked;
- retrieval performed;
- policy checked;
- tool requested;
- approval requested;
- tool result received.

---

## Chain-of-thought

Do not require or store hidden model reasoning as observability evidence.

Use structured decisions and actions instead.

---

## Token metrics

Track:

- input tokens;
- output tokens;
- cached tokens if applicable;
- context utilization.

---

## Cost metrics

Attribute cost to:

- team;
- application;
- tenant;
- environment;
- model.

---

## Cost estimation

Provider prices may change.

Keep cost models versioned/current enough for operational use.

---

## Budgets

Set budgets for:

- per request;
- per user;
- per app;
- per tenant;
- per day/month.

---

## Denial of wallet

Protect against cost exhaustion.

Use:

- quotas;
- rate limits;
- max tokens;
- concurrency;
- loop limits.

---

## Capacity

For self-hosted inference, model:

- GPU memory;
- concurrent requests;
- token throughput;
- batch size;
- context length.

Compose with `model-serving.md`.

---

## Capacity classes

Different models may have different capacity pools.

Do not allow low-priority experimentation to starve production inference.

---

## Priority

Use explicit workload priority.

Do not let applications self-assign critical priority without policy.

---

## Rate limits

Rate-limit by:

- application;
- tenant;
- user;
- model;
- expensive capability.

---

## Queueing

Under load, requests may queue.

Bound queue length and waiting time.

---

## Load shedding

Reject excess work predictably rather than allowing uncontrolled latency
growth.

---

## Scale-to-zero

For expensive self-hosted models, scale-to-zero may save cost.

Understand cold-start impact.

---

## Model warmup

Readiness should reflect actual model availability.

Do not route traffic merely because process is alive.

---

## Model rollout

A model change is a deployment event.

Use:

- canary;
- shadow;
- A/B;
- staged rollout.

---

## Prompt rollout

Prompt changes can alter behavior as much as code.

Use similar rollout/eval discipline.

---

## Retrieval rollout

Index/chunking/embedding changes should be versioned and evaluated.

---

## Tool rollout

Tool schema/behavior changes can break agents.

Version and test them.

---

## AI release manifest

A release manifest may bind:

- application version;
- model;
- prompt;
- retrieval/index;
- tool versions;
- policy version;
- eval result.

This makes production behavior explainable.

---

## Immutable release evidence

Persist release evidence with immutable identities.

Do not rely only on dashboard "current config".

---

## Rollback

Rollback may require restoring:

- model alias;
- prompt;
- index;
- tool definition;
- policy.

Keep known-good combinations.

---

## Cross-component compatibility

Not every prompt works with every model.

Not every tool schema works with every agent runtime.

Track compatibility intentionally.

---

## Model upgrade

Before upgrade evaluate:

- quality;
- safety;
- latency;
- cost;
- tool behavior;
- refusal;
- structured output.

---

## Provider deprecation

Providers may deprecate models suddenly.

Maintain inventory of dependencies.

Have migration plans for critical workloads.

---

## Portability

Be precise about portability goals.

Potential goals:

- provider portability;
- prompt portability;
- evaluation portability;
- model API portability.

Full model equivalence is unrealistic.

Do not hide model-specific capability differences behind an artificial common
denominator if applications need them.

---

## Provider abstraction

A narrow abstraction may cover common operations:

```text
generate
embed
stream
```

Expose provider-specific capability where needed.

Avoid enormous abstractions that mirror every provider feature.

---

## Open-source/self-hosted portability

Self-hosting changes:

- security;
- capacity;
- latency;
- operations;
- model licensing.

Treat as an architectural choice, not a drop-in replacement.

---

## Model licensing

Track license/use restrictions for self-hosted models.

Do not assume downloadable means unrestricted commercial use.

---

## Model provenance

For self-hosted artifacts track:

- source;
- version;
- checksum/digest;
- license;
- conversion;
- quantization.

---

## Model supply chain

Compose with `software-supply-chain.md`.

Protect:

- weights;
- tokenizer;
- adapters;
- runtime;
- container;
- custom code.

---

## Unsafe model artifacts

Avoid unsafe deserialization formats where possible.

Validate models before loading.

---

## Remote code

Some model repositories require custom remote code.

Treat that as code execution.

Review and pin it.

---

## Model scanning

Use relevant scanning for:

- malware;
- unsafe serialization;
- known vulnerable runtime components.

Do not claim model "safe" solely because scan passed.

---

## Evaluation datasets

Treat eval datasets as versioned assets.

Protect sensitive examples.

---

## Production feedback

Production feedback may improve eval coverage.

Apply privacy and anti-poisoning controls.

Do not automatically turn user feedback into training/eval truth.

---

## Feedback poisoning

A malicious user may manipulate ratings or examples.

Preserve provenance and review.

---

## Human review

For high-impact use, human review may remain required.

Define exactly:

- what reviewer sees;
- what decision they make;
- what evidence is available.

---

## Human-in-the-loop

Do not call a system human-in-the-loop if humans merely rubber-stamp outputs
without realistic ability to detect problems.

---

## Approval UX

Approval should clearly show:

- proposed action;
- target;
- important parameters;
- consequences.

Avoid presenting raw model prose as sufficient approval evidence.

---

## Safety architecture

Safety controls may include:

- content policy;
- model/provider filters;
- application validation;
- rate limits;
- human approval.

No single layer is complete.

---

## Moderation

Moderation may be:

- provider-native;
- platform service;
- application-specific.

Use where the product/risk requires.

Do not apply generic moderation blindly to every enterprise workflow.

---

## Refusal behavior

Define expected refusal behavior for prohibited/unsupported requests.

Evaluate it.

---

## Abstention

For uncertainty-sensitive systems, support abstention/escalation.

Do not force the model to answer every request.

---

## Hallucination control

Potential controls:

- retrieval;
- constrained output;
- citation;
- deterministic lookup;
- abstention;
- human review.

Do not claim hallucination is eliminated.

---

## Citation integrity

If platform provides citations, ensure they refer to actually retrieved
sources.

Do not fabricate source links from model text.

---

## Grounding

Grounding should preserve source provenance and authorization.

---

## Deterministic decision boundaries

Keep critical decisions deterministic.

Examples:

- auth;
- money movement;
- production access;
- deletion;
- legal status.

Models may assist but should not be sole policy authority.

---

## Generated code

Treat model-generated code as untrusted.

Require:

- review;
- testing;
- security checks;
- sandboxing where executed automatically.

---

## Generated SQL

Validate/parameterize.

Use read-only credentials where possible.

Do not execute arbitrary model-generated SQL with admin authority.

---

## Generated URLs

Validate destinations.

Protect against SSRF and exfiltration.

---

## Generated shell

Use narrow typed tools instead of arbitrary shell when possible.

If shell is required, sandbox it.

---

## Multimodal input

Images/audio/documents may carry hostile content.

Apply:

- size limits;
- parser safety;
- content classification.

---

## Encoded instructions

Prompt injection may be hidden in:

- images;
- PDFs;
- base64;
- Unicode;
- HTML;
- metadata.

Treat all external content as untrusted regardless of representation.

---

## AI control plane

The platform may expose a control plane for:

- model approvals;
- prompt releases;
- index lifecycle;
- tool registration;
- quotas;
- policy.

Compose with `control-plane.md`.

---

## Desired state

Represent desired platform state declaratively where useful.

Examples:

```yaml
application:
  allowed_models:
    - model-a
  data_classification: confidential
  tool_policy: read-only
  monthly_budget: ...
```

---

## Reconciliation

If the platform reconciles provider resources:

- idempotent create/update/delete;
- stable external IDs;
- drift handling;
- safe retries.

---

## Provider drift

Detect external changes to:

- model deployment;
- quota;
- network;
- credentials.

Do not overwrite intentional provider-side changes blindly if ownership is
shared.

---

## Bootstrap

Document how foundational AI platform pieces are established.

Possible dependencies:

- identity;
- network;
- secret management;
- data platform;
- model provider;
- evaluation store.

Avoid circular bootstrap.

---

## Platform outage

Define what happens if:

- model gateway unavailable;
- eval service unavailable;
- prompt registry unavailable;
- tool registry unavailable;
- policy service unavailable.

---

## Data-plane survivability

Some applications may continue using cached model/provider config during
control-plane outage.

Define safe limits.

---

## Last-known-good configuration

Preserve validated:

- model routing;
- prompt;
- policy;
- tool schema.

Do not replace known-good config with malformed updates.

---

## Fail-open versus fail-closed

Choose per control.

Examples:

- logging failure: often fail open;
- data-egress policy unavailable: often fail closed;
- eval service unavailable during runtime: usually fail open;
- tool authorization unavailable: fail closed.

---

## Disaster recovery

Recover:

- platform config;
- model catalog;
- prompt versions;
- policies;
- eval evidence;
- index metadata;
- tool registry.

---

## Model artifacts in DR

For self-hosted models, ensure model artifacts remain retrievable.

---

## RAG recovery

Indexes may be rebuildable from source.

Prefer rebuild over backup where practical and within recovery objectives.

---

## Prompt/eval backup

Prompts/eval datasets are small but critical.

Keep them versioned.

---

## Platform SLOs

Potential SLOs:

- model gateway availability;
- inference success;
- p95 latency;
- embedding availability;
- retrieval latency;
- tool-broker availability.

Do not invent targets without product requirements.

---

## Application SLOs

Application teams own end-user SLOs.

Platform SLOs should support them.

---

## Quality SLOs

Behavioral quality may be tracked, but be cautious calling probabilistic eval
metrics SLOs unless semantics are well understood.

---

## Audit

Audit consequential AI platform actions:

- model approval;
- provider policy change;
- tool registration;
- high-impact tool execution;
- prompt release;
- quota override;
- data-egress exception.

---

## AI audit event

Useful fields:

- actor;
- application;
- model;
- prompt version;
- tool;
- resource;
- policy decision;
- approval;
- outcome.

Do not store hidden reasoning.

---

## Incident response

AI incidents may include:

- provider outage;
- cost spike;
- prompt injection exploit;
- data leakage;
- unsafe tool action;
- bad model rollout;
- retrieval poisoning;
- cross-tenant leak.

Prepare runbooks.

---

## Kill switches

Provide ability to disable:

- model;
- provider;
- tool;
- application;
- tenant;
- prompt version.

Understand blast radius.

---

## Emergency policy

Emergency controls should be:

- explicit;
- audited;
- reversible.

---

## Quarantine

Quarantine compromised:

- tool server;
- model artifact;
- prompt;
- index/source.

---

## Production investigation

Preserve enough evidence to reconstruct:

- exact configuration;
- model;
- request context;
- policy;
- tool calls.

Respect privacy.

---

## Developer experience

A good AI platform should make the safe path easy.

Developers should be able to:

- discover models;
- request access;
- test locally;
- run evals;
- inspect cost;
- see policy errors;
- promote changes.

---

## AI playground

A playground may help exploration.

Keep it isolated from production authority.

Do not let experimentation credentials silently become production access.

---

## Sandbox

Provide safe environments for:

- model testing;
- prompt iteration;
- tool testing;
- RAG experiments.

Use synthetic/non-sensitive data by default.

---

## Local development

Support local stubs/fakes/test models where useful.

Do not require live paid provider calls for every unit test.

---

## Test model

A deterministic or small test model may improve CI.

Do not treat its behavioral performance as evidence for production model
quality.

---

## Mock provider

Mock transport failures separately from model behavioral eval.

---

## SDK

A platform SDK may simplify:

- auth;
- model invocation;
- telemetry;
- policy metadata.

Compose with `library-sdk.md`.

Avoid locking consumers to one framework.

---

## CLI

A CLI may support:

- model discovery;
- eval;
- prompt release;
- quota inspection;
- tool registration.

Compose with `cli.md`.

---

## Portal

A portal may provide discovery and governance.

The platform should remain automatable without it.

---

## Catalog UX

Expose:

- models;
- restrictions;
- cost;
- eval evidence;
- lifecycle.

Avoid ranking models as "best" without context.

---

## Policy UX

Developers should understand why a model/provider/tool was denied.

---

## Evaluation UX

Make regression results easy to compare.

Do not reduce all results to one score.

---

## Platform product model

Treat AI capabilities as products.

Track:

- consumers;
- adoption;
- incidents;
- support;
- quality;
- cost;
- deprecations.

---

## Capability lifecycle

Capabilities may be:

```text
experimental
supported
default
deprecated
retired
```

---

## Provider lifecycle

Have a process to:

- onboard;
- approve;
- monitor;
- deprecate;
- remove

providers.

---

## Model deprecation

A model retirement should identify:

- affected apps;
- replacement candidates;
- eval comparison;
- migration deadline.

---

## Consumer inventory

Know which applications use:

- provider;
- model;
- prompt;
- tool.

This is critical for migration.

---

## Migration

A model migration may require:

```text
inventory
    ->
candidate selection
    ->
eval
    ->
shadow/canary
    ->
consumer migration
    ->
retire old
```

---

## Prompt migration

Prompt changes may be application-owned.

Use eval and rollout.

---

## Provider migration

Changing providers may require:

- API adaptation;
- prompt changes;
- safety changes;
- data policy review.

---

## Tool migration

Changing tool schema/API can break agents.

Maintain compatibility window.

---

## Index migration

For RAG changes:

- build new index;
- evaluate;
- shadow;
- cut over;
- retain rollback.

---

## Cost architecture

AI cost may come from:

- tokens;
- GPU hours;
- vector search;
- storage;
- evals;
- tool calls;
- external APIs.

Expose major cost drivers.

---

## Cost attribution

Attribute by:

- application;
- team;
- tenant;
- environment;
- model.

---

## Cost guardrails

Use:

- quotas;
- budget alerts;
- model limits;
- max context;
- concurrency.

---

## Cost-aware routing

Routing may choose cheaper models where quality/risk allows.

Do not let cost override safety/data policy.

---

## Capacity reservation

Critical workloads may need reserved inference capacity.

Avoid over-reserving expensive hardware without demand.

---

## GPU scheduling

For self-hosted models:

- isolate workloads;
- avoid unsafe device sharing;
- monitor memory fragmentation.

Compose with `model-serving.md`.

---

## Platform security threat model

At minimum consider:

- prompt injection;
- indirect injection;
- model/provider compromise;
- stolen provider credentials;
- malicious application;
- cross-tenant cache leak;
- tool privilege escalation;
- confused deputy;
- RAG poisoning;
- memory poisoning;
- generated-code execution;
- denial of wallet;
- model artifact compromise;
- evaluation poisoning.

---

## Malicious application

An internal application may be compromised or misconfigured.

Protect platform-wide resources with:

- quotas;
- identity;
- tenant isolation;
- provider policy.

---

## Compromised provider

A provider compromise may expose:

- prompts;
- outputs;
- credentials;
- model behavior.

Minimize sent data and privilege.

---

## Compromised tool

A tool server may return malicious content or perform unsafe actions.

Authenticate tools and treat outputs as untrusted.

---

## RAG poisoning

Malicious documents may influence model behavior.

Use:

- source trust;
- provenance;
- content isolation;
- policy.

---

## Eval poisoning

If release gates use mutable datasets, attackers may manipulate evidence.

Protect eval datasets and results.

---

## Secret exfiltration

Guard against models/tools exposing:

- environment variables;
- credentials;
- internal documents.

Use least privilege and context minimization.

---

## Denial of wallet

Attackers may generate expensive loops/contexts/tool calls.

Bound every dimension.

---

## Model extraction

For proprietary/self-hosted models, consider extraction risk where applicable.

Do not expose excessive logits/weights/artifacts unintentionally.

---

## Supply chain

Compose with `software-supply-chain.md`.

Protect:

- model artifacts;
- serving images;
- agent frameworks;
- prompt packages;
- tool plugins;
- evaluation code.

---

## Dependency sprawl

AI ecosystems move quickly.

Keep dependencies intentional.

Do not add five orchestration frameworks for overlapping use cases.

---

## Framework abstraction

Avoid platform APIs tightly coupled to one agent framework.

Prefer stable domain contracts.

---

## Model provider SDKs

Hide raw provider credentials.

Wrap only where the platform gains meaningful policy/telemetry value.

---

## Version pinning

Pin critical SDKs/runtime/model artifacts sufficiently for reproducibility.

---

## AI package scanning

Scan code dependencies and container artifacts normally.

Model-specific scanning complements, not replaces, software supply-chain
controls.

---

## Licensing

Track model and dataset licenses where relevant.

Do not assume public availability means unrestricted use.

---

## Dataset rights

For training/eval/RAG sources, understand rights and permitted use.

---

## Governance

AI governance should be tied to actual platform controls.

Possible governance objects:

- model approval;
- use-case classification;
- data classification;
- eval evidence;
- exception;
- owner.

Avoid governance that exists only as forms/spreadsheets disconnected from
runtime enforcement.

---

## Use-case classification

Different use cases carry different risk.

Examples:

- summarization;
- content generation;
- coding assistant;
- internal search;
- decision support;
- autonomous action.

Use classification to drive controls.

---

## Risk tiers

If using tiers, define concrete requirements per tier.

Avoid arbitrary scoring systems.

---

## Evidence

For a production AI use case preserve:

- owner;
- risk class;
- model;
- prompt;
- data policy;
- eval;
- approval where required.

---

## Regulatory claims

Do not claim regulatory compliance from platform features alone.

Evidence, process, and organizational controls still matter.

---

## Model cards

Model cards can capture:

- intended use;
- limitations;
- evaluation;
- safety notes.

Use provider documentation critically.

---

## Application cards

An application-level record may be more useful than model-only documentation.

Capture:

- purpose;
- users;
- model;
- tools;
- data;
- risks;
- eval.

---

## Documentation

The platform should document:

- model access;
- model catalog;
- provider constraints;
- prompt/versioning;
- retrieval;
- tools;
- evals;
- quotas;
- data handling;
- incident response.

---

## Developer documentation

Consumers should know:

- how to authenticate;
- how to select model;
- how to classify data;
- how to run evals;
- how to request tools;
- how to observe cost.

---

## Operator documentation

Operators should know:

- provider failover;
- model rollback;
- quota changes;
- tool quarantine;
- prompt rollback;
- index rebuild;
- incident handling.

---

## Runbooks

Create runbooks for:

- provider outage;
- cost spike;
- model regression;
- cross-tenant leak;
- prompt injection incident;
- unsafe tool action;
- RAG poisoning;
- model-serving saturation.

---

## Architecture diagrams

Useful views include:

### Capability view

```text
Applications
  |
  +--> Models
  +--> RAG
  +--> Agents/Tools
  +--> Evaluation
  |
 Identity / Policy / Quotas / Telemetry
```

### Trust view

```text
User -> App -> AI Platform -> Provider / Tool / Data Source
```

### Release view

```text
Model + Prompt + Index + Tools + Policy + Eval -> Release
```

### Agent view

```text
Model -> Proposed Tool Call -> Policy -> Approval -> Tool
```

---

## Architecture decisions

Use ADRs for consequential choices such as:

- model gateway scope;
- provider strategy;
- self-hosted vs external;
- model abstraction;
- RAG ownership;
- agent runtime;
- tool protocol;
- eval gates;
- data-egress policy.

---

## Recommended repository shape

Follow existing repository conventions first.

A generic AI-platform repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── architecture/
│   ├── capabilities.md
│   ├── trust-boundaries.md
│   ├── data-flows.md
│   └── decisions/
├── contracts/
│   ├── model-api/
│   ├── retrieval/
│   ├── tools/
│   └── evaluation/
├── platform/
│   ├── model-gateway/
│   ├── model-catalog/
│   ├── retrieval/
│   ├── agents/
│   ├── tools/
│   ├── evaluation/
│   ├── policy/
│   └── observability/
├── prompts/
├── evals/
├── policies/
├── sdk/
├── cli/
├── tests/
│   ├── contract/
│   ├── integration/
│   ├── security/
│   ├── evaluation/
│   ├── failure/
│   └── e2e/
├── docs/
│   ├── consumers/
│   ├── operators/
│   ├── migration/
│   └── runbooks/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

An AI-platform repository should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-contract
test-integration
test-security
eval
eval-regression
eval-adversarial
test-failure
test-e2e
verify
```

These names are illustrative.

Use only commands the repository can actually implement.

---

## Acceptance criteria

An AI platform is not complete because applications can call a model.

Demonstrate the applicable subset of the following.

### Model access

- model identity is explicit;
- allowed models/providers are governed;
- provider credentials are not broadly distributed;
- routing/fallback behavior is documented.

### Data handling

- data classes control provider/model use;
- prompts/context are minimized;
- provider retention/training settings are understood;
- cross-tenant caches/history are isolated.

### Prompt/config

- prompts are versioned;
- prompt changes are evaluated;
- production prompt identity is observable;
- secrets are not embedded in prompts.

### Retrieval

- retrieval authorization precedes disclosure;
- source provenance is preserved;
- index/embedding versions are explicit;
- deletion propagates.

### Agents/tools

- tool discovery does not imply authorization;
- high-impact actions require deterministic policy/approval;
- budgets bound loops;
- tool outputs are treated as untrusted.

### Evaluation

- regression suites exist;
- security/adversarial evals exist;
- release gates are explicit;
- critical failures are not hidden by aggregate scores.

### Security

- prompt injection is threat-modeled;
- tenant isolation is tested;
- generated shell/SQL/URLs are controlled;
- provider/tool compromise has bounded impact.

### Operations

- cost/token usage is observable;
- quotas exist;
- model/provider outage behavior is defined;
- rollback exists for model/prompt/index/tool changes.

### Lifecycle

- models/providers/tools have lifecycle states;
- consumer inventory supports migration;
- deprecated models have migration path;
- release evidence binds exact components.

---

## Optional composition

Common combinations:

```text
platform-architecture + ai-platform
```

For shared enterprise AI capability architecture.

```text
ai-platform + model-serving
```

For self-hosted model artifacts, inference runtime, capacity, and rollout.

```text
ai-platform + rag-system + data-platform
```

For governed retrieval, source authorization, provenance, and index lifecycle.

```text
ai-platform + agentic-system + mcp-tool-server
```

For tool-using agents with bounded authority and explicit tool contracts.

```text
ai-platform + identity-platform + zero-trust-service
```

For user/workload/agent identity, delegation, and least privilege.

```text
ai-platform + ai-evaluation
```

For versioned behavioral evidence and release gates.

```text
ai-platform + ai-security
```

For prompt injection, exfiltration, tool abuse, poisoning, and model-specific
security controls.

```text
ai-platform + software-supply-chain
```

For trusted model artifacts, runtimes, plugins, and AI platform releases.

---

## Anti-patterns

Avoid:

- calling one model gateway "the AI platform";
- one agent framework forced on every application;
- raw provider API keys distributed to teams;
- "latest" model aliases with no resolved production identity;
- data classification ignored during provider routing;
- fallback to an unapproved provider;
- prompts changed directly in production with no version/eval;
- RAG retrieval before authorization;
- vector database treated as an authorization boundary;
- one global vector index for all tenants without isolation;
- model output used directly for authorization;
- model-generated tool calls executed without deterministic checks;
- arbitrary shell/HTTP/SQL tools exposed by default;
- long-running agents with no budget;
- tool credentials placed in prompts;
- hidden cross-tenant semantic caches;
- prompt/output logs retained indefinitely;
- evals reduced to one average score;
- LLM judge used for deterministic security assertions;
- critical failures averaged away;
- provider outage handled by infinite retry;
- cost controls added only after budget incidents;
- self-hosted models called "private" without reviewing runtime/network/storage;
- model download treated as trusted software;
- AI governance represented only by spreadsheets and approvals with no runtime
  enforcement.

Do not mistake access to models for an AI platform architecture.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. AI-platform purpose;
2. consumer/use-case classes;
3. capability map;
4. model/provider catalog and lifecycle model;
5. model access/routing/fallback model;
6. data-classification/egress model;
7. prompt/versioning model;
8. retrieval/index/provenance model;
9. agent/tool/approval model;
10. identity/delegation model;
11. policy enforcement model;
12. evaluation/release-gate model;
13. cost/quota/capacity model;
14. observability/audit model;
15. rollback/migration/deprecation model;
16. contract/eval/integration tests executed;
17. adversarial/security/failure tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim an AI platform is "safe", "enterprise-ready", "governed",
"provider-agnostic", "private", "multi-tenant", or "production-ready" merely
because it has a model gateway, model catalog, vector database, agent framework,
evaluation dashboard, or self-hosted model.

Describe the exact model identities, trust boundaries, data policy,
authorization, retrieval behavior, tool authority, evaluation evidence,
failure handling, and release controls that support the claim.

---

## Guiding principle

A good AI platform should make probabilistic capabilities usable without making
trust probabilistic.

Models should have identity.

Prompts should have versions.

Data egress should be intentional.

Retrieval should preserve authorization.

Tools should expose capability, not ambient authority.

Agents should have budgets.

Security decisions should remain deterministic.

Evaluations should be release evidence.

Cost should be bounded.

Failures should degrade intentionally.

And for any production AI behavior, the platform should be able to answer:

> Which exact model, prompt, retrieval state, tools, policy, identity, data
> boundary, and evaluation evidence produced or permitted this behavior, and
> what controls prevented that probabilistic system from exceeding the
> authority the application was supposed to have?
