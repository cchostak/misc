# ai-service.md

> Recipe for scaffolding or elevating a production AI-backed service.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> This recipe defines the architectural, operational, reliability, security,
> privacy, and evaluation expectations for services that depend on language
> models or other generative models.
>
> Compose with profiles such as `zero-trust-service.md`,
> `software-supply-chain.md`, `ai-security.md`, `internet-facing.md`, or
> `high-assurance.md` where appropriate.
>
> This recipe does not require a specific model vendor, framework, vector
> database, orchestration library, cloud, or serving stack.

## Purpose

Use this recipe when a repository implements a long-running service whose
behavior materially depends on an AI model.

Typical examples:

- LLM-backed APIs;
- summarization services;
- extraction/classification services;
- copilots;
- natural-language interfaces;
- model-backed workflow services;
- multimodal inference APIs;
- model-assisted internal tools;
- AI features embedded behind an application API.

Do **not** automatically apply this recipe to:

- deterministic services with no model dependency;
- offline evaluation repositories;
- model-training repositories;
- pure RAG indexing pipelines;
- autonomous tool-using agents where `agentic-system.md` is the better primary recipe.

The goal is not to make the repository look "AI-native".

The goal is to produce a service whose model-dependent behavior is:

- bounded;
- observable;
- testable;
- explainable enough to operate;
- secure at trust boundaries;
- resilient to provider/model failure;
- economical enough to run intentionally;
- explicit about uncertainty;
- safe to integrate into deterministic systems.

---

## Core principle

A model is a probabilistic dependency.

Treat it accordingly.

The service must not assume that model output is:

- correct;
- complete;
- deterministic;
- safe;
- authorized;
- policy-compliant;
- validly structured;
- free of sensitive data;
- free of adversarial influence.

The service should have a conventional software boundary around model behavior:

```text
request
   |
   v
validated input
   |
   v
prompt / model invocation
   |
   v
untrusted model output
   |
   v
schema / policy / business validation
   |
   v
application response or downstream action
```

The model is part of the implementation.

It is not the trust boundary.

---

## Architectural invariants

An AI service SHOULD satisfy these invariants unless the repository has a
documented reason not to.

### 1. Model output is untrusted input

Every model response must be treated as untrusted until validated for the
operation being performed.

If downstream code expects structured output:

- parse it;
- validate schema;
- validate ranges;
- validate enum values;
- validate identifiers;
- validate business invariants.

Do not execute arbitrary model-generated code, shell, SQL, URLs, or actions
without explicit validation and authorization.

### 2. Model identity is explicit

The service should know what model or model class it is invoking.

Record enough metadata to identify the inference configuration, such as:

- provider;
- model identifier;
- model version/revision where available;
- deployment name;
- prompt/template version;
- relevant inference parameters.

Do not silently switch models in production without an intentional change
process.

### 3. Prompts are versioned assets

System prompts, templates, schemas, policies, and model instructions should be
treated as software artifacts.

They should be:

- version controlled;
- reviewable;
- testable;
- attributable to releases.

Do not hide critical production behavior in a vendor console if it can be
managed declaratively.

### 4. Prompts do not contain secrets

Do not embed:

- API keys;
- passwords;
- signing material;
- production credentials;
- sensitive internal tokens

inside system prompts or examples.

Assume prompts and model context may eventually appear in logs, traces,
debugging output, or provider-side telemetry unless explicitly controlled.

### 5. Authorization remains deterministic

Do not rely solely on a model to decide whether a caller is authorized.

Security-critical authorization must be enforced by deterministic application
or policy logic.

A model may assist with classification or recommendation, but it should not be
the only control between an untrusted caller and a privileged operation.

### 6. Model failure is an expected dependency failure

Define behavior for:

- provider timeout;
- rate limiting;
- malformed response;
- schema failure;
- safety refusal;
- content filter rejection;
- quota exhaustion;
- provider outage;
- model unavailability;
- context overflow.

Do not allow indefinite retries or silent degradation into unsafe behavior.

### 7. Cost is an operational resource

Model tokens, GPU time, inference seconds, and provider spend are resources.

Bound them.

Use:

- request size limits;
- output token limits;
- concurrency limits;
- quotas;
- timeout budgets;
- rate limits;
- caching where safe.

### 8. Evaluation is part of correctness

Conventional unit tests are necessary but insufficient for model behavior.

Model-dependent releases should have evaluation evidence appropriate to the
risk.

### 9. Data egress is intentional

If data leaves the service boundary for a model provider, that is an explicit
data flow.

Know:

- what data is sent;
- where it goes;
- how it is retained;
- whether it may be used for training;
- what contractual/platform controls apply.

Do not assume an external model API is equivalent to an internal function call.

### 10. Fallback behavior is explicit

If a fallback model or deterministic fallback exists, define exactly when it is
used.

Do not silently move from a stronger model to a weaker model for
security-sensitive decisions without recognizing the change in behavior.

---

## Service boundary definition

Before implementation, identify:

- caller(s);
- model provider or serving layer;
- prompt/template store;
- data sources;
- retrieval systems if present;
- model context boundaries;
- persistence;
- telemetry;
- external side effects;
- authorization boundaries.

A minimal architecture may look like:

```text
Client
  |
  v
AI Service
  |
  +--> deterministic validation/policy
  |
  +--> model provider / inference runtime
  |
  +--> optional retrieval/data source
  |
  +--> telemetry
```

If the service calls tools or performs autonomous actions, apply
`agentic-system.md`.

---

## Request validation

Validate all deterministic request fields before invoking the model.

Examples:

- input size;
- content type;
- locale;
- requested operation;
- tenant;
- output format;
- model selector if exposed;
- sampling options if exposed.

Do not let callers arbitrarily control model parameters when those parameters
affect:

- cost;
- latency;
- safety;
- output determinism;
- provider access.

Bound any user-configurable inference settings.

---

## Context limits

Context windows are finite resources.

The service should:

- bound input length;
- reject or truncate intentionally;
- preserve critical instructions when truncating;
- avoid accidental removal of system/security context;
- measure token usage where practical.

Do not rely on provider-side truncation when it could discard security or
business instructions unpredictably.

If summarization is used to compress context, treat the summary as model output
and therefore as untrusted/possibly lossy.

---

## Prompt construction

Prompt assembly should be explicit.

Prefer a structure that distinguishes:

- trusted system/developer instructions;
- trusted application context;
- retrieved external content;
- user input;
- tool/model outputs.

Do not concatenate all sources into one undifferentiated string when the model
API supports stronger role/content separation.

Preserve provenance of externally sourced content where practical.

---

## Prompt injection

Any untrusted text included in model context may contain instructions.

Examples:

- user input;
- web pages;
- documents;
- email;
- tickets;
- code comments;
- retrieved knowledge;
- model-generated summaries.

The application must not assume the model can reliably distinguish malicious
instructions from useful content.

For ordinary AI services:

- keep trusted instructions separate from untrusted content;
- minimize privileged capabilities;
- validate model output;
- prevent untrusted content from redefining application policy;
- avoid embedding secrets in context.

For tool-using or autonomous systems, apply `agentic-system.md` and
`ai-security.md`.

---

## Structured output

Prefer structured model output when downstream software consumes the result.

Use:

- JSON schema;
- typed response models;
- constrained decoding where available;
- explicit enumerations.

After generation:

1. parse;
2. validate;
3. reject or retry within a bounded policy;
4. apply business validation.

Do not use fragile regex parsing of free-form prose when a structured contract
is available.

A syntactically valid object may still be semantically invalid.

---

## Output validation

Validate model output according to downstream risk.

Possible checks include:

- schema;
- length;
- range;
- identifier format;
- URL scheme;
- allowed enum;
- citation presence;
- source reference;
- content policy;
- business invariant;
- authorization context.

Do not pass model output directly into:

- SQL;
- shell;
- templates with code execution;
- filesystem paths;
- network destinations;
- privileged APIs

without deterministic validation.

---

## Hallucination-sensitive behavior

If factual accuracy matters, design for verification.

Possible controls:

- retrieval from authoritative sources;
- deterministic lookup;
- citations;
- source grounding;
- confidence thresholds;
- abstention;
- human review;
- post-generation validation.

Do not make unsupported factual claims appear authoritative merely because they
were produced fluently.

For high-impact use cases, prefer a system that can say:

```text
I cannot establish this from available evidence.
```

over fabricated certainty.

---

## Confidence and uncertainty

Do not assume model-provided confidence values are calibrated.

If confidence materially affects system behavior:

- define how it is measured;
- evaluate calibration;
- set thresholds empirically;
- preserve an `unknown` state.

Do not convert probabilistic uncertainty into deterministic approval without
validation.

---

## Model abstraction

Abstract model-provider details only when the repository benefits from it.

Useful reasons include:

- testing;
- portability;
- provider fallback;
- self-hosted versus external deployments.

Do not create a large "universal AI provider framework" for a service that
uses one stable provider.

Keep abstractions narrow.

---

## Provider-specific behavior

Different providers/models differ in:

- tokenization;
- context size;
- tool/schema support;
- refusal behavior;
- streaming semantics;
- rate limits;
- timeout behavior;
- error codes;
- data retention;
- version stability.

Do not assume a provider swap is behaviorally neutral.

A model/provider change should trigger relevant evaluation.

---

## Model versioning

Pin or identify production models sufficiently to understand release behavior.

Avoid generic aliases such as:

```text
latest
default
recommended
```

where they may move behavior without review.

If the provider exposes only aliases, record observed model metadata where
possible and treat alias movement as an operational risk.

---

## Prompt versioning

Assign version identity to production-critical prompts/templates.

This may be:

- source commit;
- file hash;
- semantic prompt version;
- release version.

Evaluation results should be attributable to the prompt/model combination.

Do not evaluate one prompt and deploy another without tracking the difference.

---

## Inference parameters

Set production inference parameters explicitly.

Examples:

- temperature;
- top-p;
- max output tokens;
- seed where supported;
- reasoning mode where applicable;
- response format.

Do not rely on provider defaults for behavior-critical settings if those
defaults may change.

---

## Determinism

Perfect determinism may not be available.

Where repeatability matters:

- reduce unnecessary randomness;
- use seeds where supported;
- pin model/version;
- pin prompt;
- constrain output;
- run repeated evaluations.

Do not promise deterministic output from a probabilistic model unless the
underlying system actually guarantees it.

---

## Retries

Retry model requests only for retryable failures.

Examples:

- transient transport errors;
- selected provider 5xx responses;
- explicit rate limits with backoff.

Do not blindly retry:

- policy refusals;
- invalid input;
- deterministic schema errors indefinitely;
- quota exhaustion;
- authorization failures.

Bound retries.

Use jitter/backoff where appropriate.

Account for duplicate billing and side effects.

---

## Timeout budget

Model inference should have an explicit timeout.

The service should consider:

```text
client timeout
    >
service processing budget
    >
model timeout
```

with enough room for cleanup/error propagation.

Do not allow model calls to consume unbounded request lifetimes.

---

## Streaming

If streaming output:

- define how partial failure is represented;
- do not emit content that should have been validated only after completion
  unless risk permits;
- handle client disconnect;
- cancel provider work where possible;
- avoid leaking internal model/provider metadata.

Streaming complicates post-generation validation.

Use it intentionally.

---

## Model refusals

A model refusal is a distinct outcome.

Do not automatically convert refusal into:

- empty success;
- generic server error;
- retries until a model eventually complies.

Define whether a refusal means:

- safe user-visible refusal;
- fallback path;
- human review;
- unsupported task.

Track refusal behavior in evaluation if it matters to product correctness.

---

## Safety filters

Provider safety filters may be useful but should not be your only application
security boundary.

Know whether filtering occurs:

- before inference;
- after inference;
- both;
- only for specific categories.

Do not assume provider safety policy matches application requirements.

Application-specific safety logic should be explicit.

---

## Content moderation

If the product requires moderation:

- define categories;
- define thresholds;
- define action;
- evaluate false positives/negatives;
- document appeal/review flows where appropriate.

Do not add moderation to products that do not require it merely because the
model provider offers it.

---

## Sensitive data

Classify what data may enter model context.

Potentially sensitive classes:

- credentials;
- secrets;
- personal data;
- health data;
- financial data;
- source code;
- customer documents;
- security findings;
- internal incident data.

Do not send sensitive data to a third-party model provider without an
intentional architecture and appropriate controls.

---

## Data minimization

Send only the context required for the task.

Do not transmit entire records, repositories, conversations, or documents when
a smaller subset is sufficient.

Data minimization reduces:

- privacy exposure;
- cost;
- prompt injection surface;
- context overflow;
- logging risk.

---

## Redaction

Where sensitive data must not leave a trust boundary, redact or tokenize before
model invocation.

Validate that redaction cannot be trivially reversed through included context.

Do not claim redaction works without testing representative samples.

---

## Provider retention and training

Document relevant provider behavior for production data.

Where configurable, select retention/training settings that match the
application's data policy.

Do not state that data is "not retained" or "not used for training" unless that
is actually established by the selected service/configuration.

---

## Logging

AI logs require special care.

Useful fields may include:

- request ID;
- operation;
- model/provider;
- model version;
- prompt/template version;
- input token count;
- output token count;
- latency;
- retry count;
- outcome;
- schema-validation result.

Do not log raw prompts/responses by default when they may contain sensitive
data.

If full-content logging is required for debugging or evaluation:

- make it explicit;
- restrict access;
- define retention;
- redact where possible.

---

## Metrics

Track model-specific operational signals where useful.

Examples:

- request rate;
- success/error rate;
- model latency;
- first-token latency;
- token consumption;
- cost estimate;
- rate-limit events;
- provider failures;
- schema-validation failures;
- refusal rate;
- fallback rate;
- cache hit rate.

Avoid labels with unbounded user content.

---

## Tracing

Tracing may include model spans.

Record metadata, not secrets.

Useful fields:

- model;
- operation;
- latency;
- token usage;
- provider request ID;
- prompt/template version.

Do not attach full prompts or responses automatically.

---

## Cost controls

Define cost boundaries.

Possible controls:

- per-request output limits;
- per-user/tenant quotas;
- concurrency limits;
- rate limits;
- cheaper model tiers for low-risk operations;
- caching;
- batching where appropriate.

Do not silently route security-sensitive requests to a cheaper model solely to
reduce cost.

---

## Quotas

If the provider has quota limits, plan for exhaustion.

Define:

- user-facing behavior;
- retry policy;
- fallback model;
- queueing behavior;
- alerting.

Do not let quota exhaustion cascade into unbounded retries.

---

## Caching

Model-response caching can reduce cost and latency.

Cache only where semantics allow it.

Consider:

- user/tenant isolation;
- authorization context;
- prompt version;
- model version;
- inference parameters;
- sensitive data;
- staleness.

Do not return one user's sensitive result to another because the cache key
ignored identity or tenant context.

---

## Retrieval

If the service uses retrieval-augmented generation, separate:

```text
retrieval
    from
generation
```

Treat retrieved content as untrusted.

Preserve source provenance where practical.

Apply `rag-system.md` when retrieval is a central architectural feature.

---

## Citations and provenance

Where the product presents factual answers from sources, expose citations or
source references when useful.

A citation should correspond to content actually retrieved or consulted.

Do not allow the model to invent source identifiers.

Validate citation references against available sources.

---

## Memory and conversation state

If the service retains conversational state:

- define retention;
- scope by identity/tenant;
- limit size;
- support deletion where required;
- avoid cross-user leakage;
- distinguish trusted application state from prior model output.

Prior model responses are not automatically trustworthy instructions.

---

## Session isolation

Conversation/session identifiers are not authorization.

Bind session access to authenticated identity where required.

Do not let possession of a guessable conversation ID grant access to another
user's context.

---

## Business logic boundaries

Use models for tasks where probabilistic behavior is acceptable.

Keep deterministic logic deterministic when practical.

Examples that usually should remain outside the model:

- authorization;
- cryptographic verification;
- financial totals;
- exact entitlement checks;
- schema enforcement;
- database uniqueness;
- policy enforcement.

A model may assist, but deterministic controls should own the final boundary.

---

## High-impact decisions

For decisions with meaningful impact on users or systems, require stronger
controls.

Possible measures:

- deterministic validation;
- human review;
- second-model verification where justified;
- source grounding;
- audit logging;
- conservative abstention.

Do not allow a model to become a silent single point of decision for high-impact
actions without explicit design.

---

## Human review

Human review is useful when:

- model uncertainty is high;
- consequences are difficult to reverse;
- evaluation evidence is insufficient;
- regulation/process requires it.

Review interfaces should show enough evidence to make a decision.

Do not make humans rubber-stamp opaque model output.

---

## Fallbacks

Fallback strategies may include:

- retry same model;
- switch region/provider;
- use alternate model;
- deterministic fallback;
- defer/queue;
- fail safely.

Each fallback should have known behavior.

Do not silently relax safety, privacy, or authorization constraints during
fallback.

---

## Provider outages

Treat model providers as external dependencies.

Define:

- timeout;
- circuit-breaker behavior where useful;
- backoff;
- queueing;
- degraded behavior;
- alerting.

Do not make the entire service hang indefinitely because the model provider is
unavailable.

---

## Local development

Developers should be able to work without uncontrolled production spend.

Possible approaches:

- provider sandbox account;
- local model;
- deterministic fake adapter;
- recorded fixtures;
- capped development credentials.

Do not let unit tests depend on paid remote inference by default.

Use real-model tests where they add value, but separate them from deterministic
unit tests.

---

## Test strategy

Conventional tests should validate deterministic code.

### Unit tests

Test:

- prompt assembly;
- configuration;
- schema validation;
- parsers;
- fallback selection;
- error mapping;
- cost/token limits;
- authorization boundaries.

Mocking the model is appropriate when testing deterministic application logic.

Do not write tests that only prove the mock returns what it was configured to
return.

### Contract tests

Where model/provider APIs matter, test the adapter against expected provider
behavior.

Validate:

- authentication;
- request shape;
- response parsing;
- error handling;
- streaming where used.

### Integration tests

Exercise the service boundary with representative model behavior.

Use controlled fixtures or a test model where possible.

### Regression tests

Every discovered model-related defect should produce a regression case where
practical.

Examples:

- malformed structured output;
- context truncation bug;
- prompt injection bypass;
- citation fabrication;
- unsafe fallback;
- cross-tenant cache leak.

---

## AI evaluation

Apply an evaluation harness to model-dependent behavior.

Relevant dimensions may include:

- task success;
- factual accuracy;
- structured-output validity;
- refusal correctness;
- hallucination rate;
- citation correctness;
- latency;
- token usage;
- cost;
- consistency;
- safety behavior.

Do not reduce evaluation to one aggregate score if important failure modes are
hidden by the average.

Prefer scenario-level evidence.

---

## Evaluation datasets

Evaluation data should include:

- representative normal cases;
- edge cases;
- known failure cases;
- adversarial cases where applicable;
- realistic input lengths;
- malformed inputs.

Separate:

- development data;
- evaluation data;
- production data

where leakage would invalidate results.

Do not tune prompts against the entire evaluation set until it ceases to
measure generalization.

---

## Evaluation reproducibility

Record enough metadata to reproduce evaluation runs:

- model;
- model version;
- prompt/template version;
- inference parameters;
- dataset version;
- evaluator version;
- run timestamp.

For nondeterministic models, use repeated samples where risk justifies it.

---

## LLM-as-judge

If another model evaluates outputs:

- treat judge output as probabilistic;
- validate judge quality against human-labeled examples;
- avoid using one judge as unquestioned ground truth;
- record judge model/version/prompt.

Do not claim objective correctness solely because another model assigned a
score.

---

## Security evaluation

Where the service handles untrusted text, include adversarial evaluation.

Potential cases:

- prompt injection;
- instruction hierarchy attacks;
- sensitive-data extraction attempts;
- malformed structured output;
- excessive input;
- Unicode/confusable tricks;
- refusal bypass attempts.

If tools/actions exist, use `agentic-system.md` and `ai-security.md` for
stronger controls.

---

## Release gates

Model/prompt changes may require evaluation before release.

A gate should define:

- dataset;
- metrics;
- acceptable regression thresholds;
- required deterministic tests;
- required security tests.

Do not gate on an opaque "AI quality score" with no interpretation.

Use tolerances appropriate to stochastic output.

---

## Model upgrades

Treat a model upgrade as a behavioral dependency change.

Before promotion:

- run regression evaluation;
- compare safety behavior;
- compare structured-output reliability;
- compare latency/cost;
- inspect known critical scenarios.

Do not assume a newer model is automatically better for the application.

---

## Prompt changes

Prompt changes are behavior changes.

Require:

- review;
- tests;
- relevant evaluation;
- versioning.

Do not treat prompt files as harmless documentation.

---

## Rollout

For high-traffic or high-risk AI services, consider gradual rollout of:

- model changes;
- prompt changes;
- retrieval changes;
- safety-policy changes.

Observe:

- quality proxies;
- error rate;
- latency;
- cost;
- schema failures;
- refusal rate;
- user-impact metrics.

Do not perform automated rollback solely on noisy quality metrics without
understanding their reliability.

---

## A/B testing

If experiments are used:

- define assignment;
- preserve user/tenant isolation;
- record model/prompt variant;
- avoid exposing sensitive populations to experimental behavior without
  appropriate governance.

Experiments should not bypass baseline security controls.

---

## Abuse controls

AI endpoints can be expensive and attractive for abuse.

Consider:

- authentication;
- rate limiting;
- quotas;
- request-size limits;
- output limits;
- per-tenant budgets;
- anomaly detection;
- concurrency caps.

Do not rely on model-provider rate limits as the application's only abuse
control.

---

## Denial-of-wallet

Treat uncontrolled model spend as a resource-exhaustion risk.

Protect expensive endpoints from:

- unauthenticated bulk use;
- recursive application behavior;
- unbounded retries;
- arbitrarily large context;
- arbitrarily large output;
- high-concurrency abuse.

Cost controls should be testable.

---

## Recursive/model loops

If one model call can trigger another:

- bound depth;
- bound total calls;
- bound total tokens/cost;
- propagate cancellation;
- detect loops where possible.

Do not allow accidental unbounded model recursion.

For autonomous loops, use `agentic-system.md`.

---

## Secrets and model context

Do not place secrets into model context unless strictly necessary and
explicitly designed.

If a model requires access to sensitive values:

- minimize scope;
- avoid logging;
- use ephemeral access;
- ensure downstream output cannot echo them casually.

A system prompt is not a secret vault.

---

## Model-generated code

If model-generated code is shown to users, treat it as untrusted content.

If the service executes generated code:

- isolate execution;
- restrict network;
- restrict filesystem;
- restrict credentials;
- apply CPU/memory/time limits;
- destroy the environment after use.

Do not execute generated code in the service's privileged runtime.

---

## Model-generated URLs

If model output can cause network requests:

- validate scheme;
- validate destination;
- enforce allowlists where appropriate;
- protect metadata/internal networks;
- limit redirects;
- bound response size.

Model output is not a trusted URL allowlist.

---

## Model-generated queries

If model output is used for database or search queries:

- prefer structured query construction;
- parameterize;
- constrain allowed operations;
- enforce tenant/resource filters independently;
- use read-only credentials where possible.

Do not grant unrestricted database authority to a model-generated query.

---

## Tool calling

If the model can call tools or perform side effects, this is no longer merely
an AI service concern.

Apply `agentic-system.md`.

At minimum:

- tools must have schemas;
- arguments must be validated;
- authorization must be deterministic;
- side effects must be bounded;
- irreversible actions require stronger controls.

---

## Privacy

Document how model-related data is handled.

Consider:

- provider egress;
- retention;
- logs;
- traces;
- evaluation datasets;
- caching;
- conversation history;
- human review.

Do not retain raw model inputs/outputs indefinitely by default.

---

## Data deletion

If the application stores conversations or prompts, deletion behavior should
cover relevant copies.

Consider:

- primary storage;
- caches;
- search indexes;
- evaluation stores;
- logs where feasible;
- provider-side retention constraints.

Do not promise deletion beyond what the architecture can actually enforce.

---

## Multi-tenancy

For multi-tenant AI services:

- isolate conversation state;
- isolate caches;
- scope retrieval;
- scope prompts/context;
- enforce tenant authorization deterministically;
- avoid cross-tenant eval/log leakage.

Tenant identity must not be derived solely from model output or user-provided
text.

---

## Configuration

Configuration may include:

- provider;
- model;
- model deployment name;
- token limits;
- timeout;
- retry policy;
- sampling settings;
- safety settings;
- prompt/template location.

Validate configuration at startup.

Fail early when model configuration is unsafe or contradictory.

---

## Secrets

Provider/API credentials must:

- stay out of source;
- stay out of prompts;
- stay out of logs;
- be scoped narrowly;
- be rotated;
- use short-lived identity where supported.

Do not share one high-privilege model credential across unrelated environments
if the platform offers narrower scopes.

---

## Observability boundaries

Telemetry should help debug model behavior without turning observability into a
data-exfiltration system.

Prefer metadata-first observability.

Only store full content when there is a deliberate, access-controlled need.

---

## Incident response

AI incidents may include:

- unexpected harmful outputs;
- prompt injection;
- sensitive-data leakage;
- provider compromise;
- model regression;
- runaway cost;
- unsafe fallback behavior;
- cross-tenant context leakage.

Preserve enough metadata to answer:

- which model;
- which prompt version;
- which application release;
- which request/session;
- which relevant data source;
- which policy configuration.

---

## Supply chain

Treat prompts, schemas, evaluation datasets, model adapters, and deployment
configuration as release-relevant artifacts.

Compose with `software-supply-chain.md` when stronger provenance is required.

For self-hosted models, also track:

- model weights;
- model digest;
- tokenizer;
- runtime;
- serving image;
- quantization/configuration.

---

## Documentation

The service documentation should explain:

- what the AI feature does;
- what model/provider it depends on;
- what inputs are accepted;
- what outputs are produced;
- known limitations;
- data flow to model providers;
- configuration;
- evaluation process;
- cost/latency considerations;
- fallback behavior;
- safety/security boundaries.

Do not state that the model is "accurate", "safe", or "private" without
bounded evidence.

---

## Architecture decisions

Use ADRs for consequential choices such as:

- provider/model selection;
- hosted versus self-hosted inference;
- fallback model;
- prompt-management strategy;
- retrieval architecture;
- safety/moderation architecture;
- retention policy;
- human-review requirement.

Do not create ADRs for every prompt wording change.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic AI service may resemble:

```text
.
├── AGENTS.md
├── README.md
├── src/
│   ├── api/
│   ├── model/
│   ├── prompts/
│   ├── policy/
│   └── observability/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── regression/
│   └── security/
├── evals/
│   ├── datasets/
│   ├── scenarios/
│   └── results/
├── docs/
│   ├── architecture/
│   └── adr/
├── config/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

Do not make `evals/` a dumping ground for unversioned manual examples.

---

## Verification interface

An AI-service repository should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-integration
test-security
eval
eval-regression
build
run
```

These names are illustrative.

Separate deterministic tests from model evaluations.

A useful conceptual distinction is:

```text
test
    = deterministic software correctness

eval
    = probabilistic model behavior
```

Both matter.

---

## Acceptance criteria

An AI service is not complete because it successfully calls a model API.

Demonstrate the applicable subset of the following.

### Service behavior

- service starts;
- request validation works;
- model dependency is reachable or appropriately simulated;
- expected model-backed request succeeds;
- model/provider failure produces intentional behavior.

### Output safety

- structured output is parsed and validated where applicable;
- malformed model output is rejected or handled;
- model output cannot bypass deterministic authorization;
- dangerous downstream values are validated.

### Resource controls

- request/context size is bounded;
- output tokens are bounded;
- inference timeout exists;
- retries are bounded;
- concurrency/rate/cost controls exist where appropriate.

### Model configuration

- model/provider identity is explicit;
- prompt/template version is identifiable;
- inference settings are explicit where behavior-critical;
- model/prompt changes are reviewable.

### Privacy/security

- secrets are not embedded in prompts;
- sensitive context is minimized;
- provider data flow is documented;
- raw prompts/responses are not logged unintentionally;
- tenant/session isolation is tested where applicable.

### Evaluation

- representative evaluation dataset exists where model quality matters;
- regression evaluation can be run;
- known critical scenarios are covered;
- model/prompt version is captured in results;
- security/adversarial cases exist where applicable.

### Operations

- model latency/error metrics exist where useful;
- token/cost usage is observable;
- provider outage behavior is defined;
- fallback behavior is explicit.

### Documentation

- limitations are documented;
- data flow is documented;
- model/provider dependency is documented;
- evaluation process is documented;
- known unverified assumptions are visible.

---

## Optional composition

Common combinations:

```text
secure-service + ai-service
```

For conventional production-service hardening around model-backed behavior.

```text
ai-service + zero-trust-service
```

For identity-aware AI APIs with explicit authorization boundaries.

```text
ai-service + software-supply-chain
```

For provenance of application, prompt, model configuration, and deployment
artifacts.

```text
ai-service + ai-security
```

For stronger prompt-injection, data-boundary, model-output, and adversarial
security requirements.

```text
ai-service + rag-system
```

When retrieval, indexing, document provenance, and retrieval authorization are
central to the architecture.

```text
ai-service + high-assurance
```

For deeper evaluation, stricter release gates, independent verification, and
high-impact use cases.

If the model is allowed to autonomously select tools or create side effects,
prefer:

```text
agentic-system
```

as the primary recipe.

---

## Anti-patterns

Avoid:

- treating model output as trusted data;
- using the model as the sole authorization engine;
- embedding secrets in system prompts;
- logging all prompts/responses by default;
- arbitrary user control over model/token parameters;
- unbounded retries;
- unbounded token generation;
- silent model/provider switching;
- vague aliases such as `latest` for behavior-critical models;
- prompt changes with no tests/evals;
- model upgrades with no regression evaluation;
- passing model-generated SQL/shell/URLs directly to execution;
- sending entire sensitive records when small context is sufficient;
- claiming "no hallucinations";
- treating model confidence as calibrated by default;
- using one aggregate eval score to hide critical regressions;
- paid remote model calls in every unit test;
- caching responses without user/tenant/model/prompt context;
- treating provider safety filters as application authorization;
- silently degrading to a weaker model during security-sensitive operations;
- calling a service "private" without understanding provider data handling;
- exposing autonomous tools without applying `agentic-system.md`.

Do not mistake successful inference for production readiness.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. AI-service use case;
2. model/provider architecture;
3. model/version identity strategy;
4. prompt/template version strategy;
5. input/context limits;
6. structured-output and validation strategy;
7. authorization boundary;
8. sensitive-data/provider egress model;
9. logging/telemetry policy;
10. timeout/retry/fallback behavior;
11. cost/quota controls;
12. model-change release process;
13. evaluation datasets/scenarios;
14. deterministic tests executed;
15. model evaluations executed;
16. adversarial/security tests executed;
17. commands actually run;
18. observed results;
19. unverified assumptions;
20. controls deliberately not implemented and why.

Never claim the AI behavior is "safe", "accurate", "reliable", "private", or
"production-ready" based solely on successful API calls or a handful of happy
path examples.

---

## Guiding principle

A production AI service should put deterministic engineering around
probabilistic behavior.

The model may generate.

The application must validate.

The model may suggest.

Policy must authorize.

The model may fail.

The service must degrade intentionally.

The model may change.

Evaluation must detect meaningful regression.

The model may process sensitive context.

The architecture must control where that data goes.

And whenever probabilistic output crosses into deterministic software, treat
that boundary as a place where assumptions must become enforceable checks.
