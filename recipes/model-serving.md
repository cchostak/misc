# model-serving.md

> Recipe for scaffolding or elevating a production model-serving platform or
> inference service.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with profiles such as `ai-security.md`, `zero-trust-service.md`,
> `software-supply-chain.md`, `ai-evaluation.md`, `internet-facing.md`, or
> `high-assurance.md` where appropriate.
>
> This recipe defines the architectural, security, reliability, capacity,
> model-integrity, rollout, observability, and operational expectations for
> systems that host and serve machine-learning or generative models.
>
> It does not require a specific framework, accelerator vendor, model format,
> orchestration platform, cloud, serving runtime, or API protocol.

## Purpose

Use this recipe when the repository owns the runtime path that loads models
and serves inference.

Typical examples:

- self-hosted LLM serving;
- embedding services;
- reranker services;
- image/audio model serving;
- internal inference gateways;
- GPU-backed model APIs;
- model-serving control planes;
- multi-model inference platforms;
- model deployment infrastructure;
- edge inference services.

Do **not** automatically apply this recipe when:

- the system only calls a third-party model API;
- the repository only evaluates models;
- the repository only trains models;
- the model is embedded locally inside a desktop/mobile application and does
  not operate as a managed serving system.

The goal is not to maximize GPU utilization or serving-stack complexity.

The goal is to produce a system where models are:

- identified immutably;
- loaded intentionally;
- isolated appropriately;
- served predictably;
- capacity-managed;
- observable;
- reproducible enough to diagnose;
- protected as supply-chain artifacts;
- rolled out with evidence;
- retired safely.

---

## Core principle

A model-serving system is both:

```text
an application runtime
        +
a model artifact runtime
```

Both layers matter.

The service must be able to answer:

- Which exact model is loaded?
- Which exact serving runtime is executing it?
- Which configuration affects inference?
- Which hardware/runtime assumptions apply?
- Which callers may use it?
- How much capacity is available?
- What happens under overload?
- How is a new model promoted?
- How is a bad model rolled back?
- What evidence proves the running model is the intended one?

---

## Architectural invariants

A model-serving system SHOULD satisfy these invariants unless the repository
has a documented reason not to.

### 1. Model identity is immutable

A deployed model must have an immutable identity.

Prefer:

- artifact digest;
- immutable model revision;
- version plus checksum;
- signed model artifact reference.

Avoid relying only on mutable aliases such as:

```text
latest
production
recommended
main
```

Human-readable aliases may exist, but runtime identity should resolve to an
immutable artifact.

### 2. Runtime identity is explicit

Track the serving runtime and relevant execution environment.

Examples:

- serving framework version;
- container digest;
- CUDA/ROCm/runtime version;
- accelerator type;
- quantization;
- tokenizer;
- inference engine;
- model adapter version.

A model artifact without runtime context is incomplete operational evidence.

### 3. Model loading is controlled

Do not allow arbitrary callers to load arbitrary model artifacts into a
privileged serving runtime.

Model loading should be:

- authorized;
- validated;
- observable;
- resource-bounded.

### 4. Capacity is finite

Inference capacity is constrained by:

- accelerator memory;
- compute;
- batching;
- context length;
- concurrency;
- token generation;
- model count.

Bound it.

Do not accept unbounded work and hope the runtime survives.

### 5. Overload behavior is intentional

Define what happens when capacity is exhausted.

Possible outcomes:

- queue;
- shed load;
- rate limit;
- reject;
- degrade to another model;
- defer.

Do not let overload become uncontrolled memory growth or global instability.

### 6. Model rollout is a deployment event

A model change is a production behavior change.

It should have:

- version identity;
- evaluation evidence;
- rollout plan;
- observability;
- rollback path.

### 7. Model files are supply-chain artifacts

Weights, tokenizers, configs, adapters, and custom code are part of the
software supply chain.

Treat them accordingly.

### 8. Inference data is sensitive by default

Requests and outputs may contain:

- source code;
- personal data;
- credentials;
- proprietary documents;
- security findings;
- internal business information.

Logging and retention must be intentional.

### 9. Performance and correctness are linked

Changes that improve throughput but alter:

- numerical behavior;
- truncation;
- context handling;
- output quality;
- structured-output reliability

must be evaluated as behavior changes.

### 10. Model server health is distinct from model quality

A healthy process does not mean the model is producing acceptable output.

Operational health and behavioral evaluation are different concerns.

---

## Serving boundary definition

Before implementation, identify:

- model artifact store;
- model registry/catalog;
- serving runtime;
- API/protocol;
- accelerator/hardware;
- scheduler;
- batching layer;
- cache;
- tokenizer;
- adapters;
- authentication;
- authorization;
- telemetry;
- rollout mechanism.

A minimal architecture may look like:

```text
Client
  |
  v
Inference API
  |
  +--> AuthN/AuthZ
  |
  +--> Admission / Rate Limit
  |
  +--> Scheduler / Batcher
  |
  +--> Model Runtime
          |
          +--> Model Artifact
          +--> Tokenizer
          +--> Accelerator
  |
  +--> Metrics / Logs / Traces
```

If the service also implements application-level AI behavior, compose with
`ai-service.md`.

---

## Model artifact definition

Define what constitutes one deployable model unit.

Depending on the system, it may include:

- weights;
- tokenizer;
- configuration;
- generation defaults;
- adapters/LoRA;
- prompt template;
- custom model code;
- quantization metadata;
- license/usage metadata.

Do not treat only the primary weights file as the complete model identity if
other files materially affect behavior.

---

## Model registry

Use an explicit model artifact source.

Possible systems include:

- OCI registry;
- model registry;
- object store with immutable versions;
- package registry;
- internal artifact store.

The source should support:

- immutable identity;
- access control;
- metadata;
- retention;
- integrity verification.

Avoid fetching production model weights from arbitrary public URLs at startup.

---

## Model provenance

Where required, preserve provenance such as:

- upstream source;
- revision;
- fine-tuning dataset/reference;
- training job identity;
- quantization process;
- conversion process;
- checksum/digest;
- approval/release state.

Do not claim provenance that cannot be established.

For third-party models, record the upstream artifact and local transformations.

---

## Model integrity

Verify model artifact integrity before loading.

Possible controls:

- digest;
- checksum;
- signature;
- provenance policy.

A model should not be trusted solely because its filename matches expectation.

---

## Tokenizer integrity

The tokenizer can materially alter behavior.

Treat tokenizer files/config as part of the model artifact set.

Track tokenizer version/hash where relevant.

Do not mix arbitrary tokenizer/model combinations unless explicitly supported.

---

## Custom model code

Some model formats or repositories may include executable custom code.

Treat remote/custom model code as executable dependency.

Do not enable arbitrary remote code execution by default.

If custom code is required:

- review it;
- pin it;
- isolate it;
- include it in supply-chain evidence.

---

## Model conversion

If converting between model formats:

- version the conversion tool;
- record conversion configuration;
- verify output integrity;
- evaluate resulting behavior.

Examples:

- framework conversion;
- ONNX export;
- TensorRT conversion;
- quantization;
- sharding;
- compilation.

Do not assume conversion is behaviorally neutral.

---

## Quantization

Quantization can change:

- accuracy;
- hallucination rate;
- structured output;
- latency;
- memory usage.

Track:

- method;
- bit width;
- calibration data where applicable;
- runtime.

Evaluate quality and safety after quantization.

Do not promote quantized variants solely because they are cheaper/faster.

---

## Adapters and LoRA

Treat adapters as first-class artifacts.

Track:

- base model identity;
- adapter identity;
- merge state;
- adapter configuration.

Do not apply an adapter to an incompatible base model silently.

---

## Multi-model serving

If serving multiple models:

- isolate model identity;
- route explicitly;
- bound per-model resources;
- prevent one model from starving others;
- expose model-specific telemetry.

Do not let arbitrary callers select privileged or experimental models unless
authorized.

---

## Model routing

Routing may depend on:

- task;
- tenant;
- latency target;
- cost;
- region;
- capability.

Keep routing policy explicit.

Do not use a model to decide its own privileged routing unless deterministic
policy constrains the decision.

---

## Runtime configuration

Inference behavior may depend on:

- max sequence length;
- max output length;
- temperature;
- top-p;
- top-k;
- seed;
- repetition penalty;
- beam settings;
- speculative decoding;
- batching;
- cache configuration.

Version or capture settings that materially affect production behavior.

Do not rely blindly on changing runtime defaults.

---

## Context limits

Enforce context-window limits.

Define behavior for oversized requests:

- reject;
- truncate;
- summarize;
- chunk.

Do not silently truncate critical instructions or security context.

For raw model-serving APIs, return a clear validation error when possible.

---

## Output limits

Bound output size.

For generative models, define:

- max output tokens;
- stream timeout;
- overall request timeout.

Do not allow one request to occupy a worker indefinitely.

---

## Request validation

Validate:

- model name/version;
- input type;
- input length;
- content format;
- generation parameters;
- batch size;
- output settings.

Do not allow user-controlled parameters to exceed safe operational bounds.

---

## Authentication

If the serving endpoint is not intentionally public, authenticate callers.

Use an established mechanism appropriate to the environment.

Examples:

- workload identity;
- OAuth/OIDC;
- mTLS;
- signed service tokens.

Do not rely solely on private network placement.

---

## Authorization

Authorization may govern:

- model access;
- tenant;
- model tier;
- experimental model access;
- expensive inference modes;
- administrative operations.

A valid caller should not automatically access every model.

---

## Administrative API

Separate administrative operations from inference operations where practical.

Admin operations may include:

- load/unload model;
- change routing;
- update config;
- flush cache;
- restart workers.

Use stronger authorization.

Do not expose unauthenticated admin endpoints.

---

## Workload identity

For internal callers, prefer workload identity and short-lived credentials
where supported.

Do not distribute one static API key to every service.

---

## Multi-tenancy

For shared serving:

- isolate request state;
- isolate caches;
- isolate quotas;
- isolate logs;
- scope model access;
- scope adapters where tenant-specific.

Do not let one tenant infer another tenant's prompts/responses through caches
or metrics.

---

## Prompt/input privacy

Requests may be highly sensitive.

By default:

- do not log full prompts;
- do not expose prompts in metrics;
- do not include prompts in error traces.

If content logging is required:

- gate access;
- redact;
- define retention;
- document purpose.

---

## Output privacy

Generated outputs may reproduce sensitive request context.

Treat outputs according to application data policy.

Do not persist full outputs by default unless required.

---

## Data retention

Define retention for:

- request metadata;
- raw inputs;
- raw outputs;
- trace data;
- cache entries.

Do not retain everything indefinitely for debugging convenience.

---

## Caching

Model-serving caches may include:

- KV cache;
- prefix cache;
- response cache;
- compiled graph cache.

Security requirements differ.

Ensure cache isolation where one user's data might be reused across another
user or tenant.

Do not use semantic response caching across security domains without explicit
authorization-aware keys.

---

## Prefix/KV cache

Shared prefix or KV caches may create subtle isolation risks.

Understand whether the serving runtime can expose information across requests
or tenants.

Use isolation/partitioning where required.

---

## Batching

Batching improves throughput but affects latency and isolation.

Bound:

- batch size;
- wait time;
- memory usage.

Ensure one abusive caller cannot dominate batches.

Do not allow batching logic to mix tenant-sensitive outputs incorrectly.

---

## Scheduling

The scheduler should define fairness.

Possible strategies:

- FIFO;
- weighted fair;
- per-tenant quota;
- priority queues;
- reserved capacity.

Avoid starvation.

Priority should be policy-driven, not solely client-controlled.

---

## Backpressure

When the system is saturated:

- reject;
- queue within bounds;
- shed lower-priority work;
- return retry information where useful.

Do not allow unbounded queues.

Queue depth is a production metric.

---

## Rate limiting

Rate-limit by appropriate dimensions:

- caller;
- tenant;
- model;
- API key;
- operation.

Consider separate limits for:

- concurrent requests;
- tokens per minute;
- requests per minute;
- expensive model modes.

---

## Quotas

Quotas may bound:

- daily tokens;
- GPU seconds;
- requests;
- cost.

Quota state should be observable.

Do not rely only on global provider/runtime limits.

---

## Denial-of-wallet

Protect expensive serving endpoints from abuse.

Controls may include:

- authentication;
- rate limits;
- token limits;
- concurrency caps;
- quotas;
- anomaly detection.

Do not expose an expensive model publicly without intentional abuse controls.

---

## GPU/accelerator isolation

For shared accelerator hosts, consider:

- memory isolation;
- process isolation;
- device allocation;
- driver/runtime trust;
- workload tenancy.

Avoid running unrelated untrusted workloads with excessive device privileges.

---

## Device access

Grant only required device access.

Do not run inference containers with broad host privileges solely for
convenience.

Where possible:

- non-root runtime;
- minimal capabilities;
- scoped device mounts.

---

## Runtime sandboxing

Model-serving runtimes are large dependency surfaces.

Reduce privileges.

Consider:

- read-only root filesystem;
- minimal filesystem write paths;
- dropped capabilities;
- no privilege escalation;
- restricted egress;
- resource limits.

---

## Network egress

Most serving runtimes should not require arbitrary outbound network access at
inference time.

Where possible, restrict egress after model artifacts are loaded.

This reduces:

- exfiltration;
- accidental remote dependencies;
- supply-chain fetches at runtime.

---

## Model download phase

Separate artifact acquisition from serving where practical.

Prefer:

```text
verified model artifact
    ->
staged locally
    ->
serving runtime
```

over:

```text
serving runtime starts
    ->
downloads arbitrary model from internet
```

---

## Startup

Startup may include:

- artifact verification;
- model download/staging;
- loading weights;
- compilation;
- warmup.

Expose startup state.

Do not report readiness before the model can successfully serve requests.

---

## Readiness

Readiness should indicate whether the instance can accept inference.

It may depend on:

- model loaded;
- tokenizer loaded;
- accelerator available;
- required cache initialized;
- runtime healthy.

---

## Liveness

Liveness should answer whether restarting the process may help.

Do not make liveness fail due to every temporary upstream or load condition.

Avoid restart loops under saturation.

---

## Warmup

Some runtimes require warmup for:

- compilation;
- memory allocation;
- kernel selection;
- cache initialization.

Make warmup explicit.

Observe warmup duration and failures.

---

## Graceful shutdown

Shutdown should:

1. stop accepting new requests;
2. mark not-ready;
3. drain active requests within a bound;
4. cancel/terminate remaining work safely;
5. release accelerator resources;
6. flush required telemetry;
7. exit.

Do not kill long-running generation abruptly without defining client behavior.

---

## Streaming

For streaming inference:

- propagate client cancellation;
- release compute promptly;
- define partial-failure behavior;
- bound stream duration.

Do not continue generating after the client disconnects if cancellation is
available.

---

## Cancellation

Cancellation should free:

- scheduler slot;
- memory;
- KV cache;
- generation state.

Monitor cancellation effectiveness.

---

## Timeouts

Define:

- queue timeout;
- inference timeout;
- stream timeout;
- overall request timeout.

Do not use one huge timeout for every stage.

---

## Retry semantics

Clients should not blindly retry expensive inference.

Server-side retries should be rare and bounded.

Do not retry deterministic invalid requests.

If retries occur, understand billing/capacity impact.

---

## Capacity planning

Capacity planning should use representative workloads.

Measure:

- tokens/sec;
- requests/sec;
- concurrent sequences;
- memory per request;
- model load memory;
- context-length effects;
- batch effects.

Do not size infrastructure using only synthetic short prompts if production
uses long context.

---

## Performance profiling

Profile before optimizing.

Track:

- prefill latency;
- decode latency;
- first-token latency;
- tokens/sec;
- batch utilization;
- GPU utilization;
- memory utilization;
- queue time.

Average end-to-end latency alone is insufficient for generative serving.

---

## Tail latency

Use percentile metrics.

Track at least meaningful high percentiles for production workloads.

Averages can hide queueing and overload problems.

---

## Throughput

Define throughput in meaningful units.

Examples:

- requests/sec;
- input tokens/sec;
- output tokens/sec;
- sequences/sec.

Do not compare systems using incompatible metrics.

---

## Autoscaling

Autoscaling may consider:

- queue depth;
- concurrent requests;
- GPU utilization;
- token throughput;
- custom saturation metrics.

CPU utilization alone may be a poor signal for GPU inference.

Test scale-up and scale-down behavior.

---

## Scale-to-zero

Scale-to-zero may save cost but increase cold-start latency.

Use it only where the latency tradeoff is acceptable.

Do not surprise latency-sensitive users with hidden multi-minute cold starts.

---

## Admission control

Reject work that cannot be served safely.

Examples:

- input too large;
- requested batch too large;
- model unavailable;
- queue full;
- quota exceeded.

Admission should happen before expensive allocation where possible.

---

## OOM handling

Out-of-memory behavior must be intentional.

Avoid repeated crash loops from requests that exceed known memory limits.

Consider:

- pre-admission estimates;
- context/output bounds;
- batch limits;
- worker isolation.

A single pathological request should not take down the entire fleet if
avoidable.

---

## Memory fragmentation

Long-running accelerator processes can fragment memory.

Observe and test long-lived workloads.

Do not assume a benchmark after fresh startup reflects day-long behavior.

---

## Model isolation

If multiple models share hosts, failure in one model should not unnecessarily
take down all others.

Use process or worker isolation where justified.

---

## Model loading attacks

Treat model files as potentially unsafe inputs, especially third-party
artifacts.

Risks may include:

- unsafe serialization;
- executable custom code;
- malformed tensor files;
- parser/runtime vulnerabilities.

Prefer safer model formats and validated loading paths where possible.

---

## Unsafe serialization

Avoid model formats that deserialize arbitrary code where safer alternatives
exist.

Do not load untrusted serialized objects into privileged runtimes.

---

## Model licensing

Track model license and usage constraints.

Do not invent legal interpretations.

If license status is unclear, surface it as a human decision.

Model licensing can affect distribution and serving rights.

---

## Export restrictions / policy constraints

Where organizational or legal policy imposes restrictions on model use, make
those constraints explicit.

Do not infer them without evidence.

---

## Supply chain

Compose with `software-supply-chain.md` for stronger controls.

At minimum consider:

- immutable model digest;
- immutable serving image;
- pinned runtime dependencies;
- SBOM for serving software;
- provenance for model transformations;
- signatures;
- verification before load.

---

## Signed model artifacts

If model artifacts are signed, verify before loading.

Do not sign model bundles without defining a verification point.

Verification should bind to the immutable artifact.

---

## Model release record

For important models, maintain a release record that can identify:

- model digest;
- base model;
- adapter/quantization;
- tokenizer;
- serving runtime;
- evaluation results;
- approval;
- rollout status.

Avoid scattered metadata that cannot be correlated.

---

## Evaluation before promotion

A model should not enter production solely because it loads successfully.

Apply `ai-evaluation.md`.

Evaluate dimensions appropriate to the product, such as:

- task quality;
- safety;
- structured-output reliability;
- latency;
- memory;
- cost;
- regression scenarios.

---

## Runtime benchmark versus model eval

Keep performance benchmarking separate from behavioral evaluation.

A faster model may be worse.

A more accurate model may not fit latency/cost requirements.

Both dimensions matter.

---

## Canary rollout

For model changes, consider canary rollout.

Route a small share of eligible traffic first.

Observe:

- errors;
- latency;
- queueing;
- OOM;
- token throughput;
- quality proxies;
- safety metrics.

Do not route sensitive or high-impact users to experimental models without
policy.

---

## Shadow deployment

Shadow traffic can compare candidate and current models without affecting user
responses.

Use it to compare:

- latency;
- capacity;
- output quality;
- resource use.

Ensure shadow calls obey privacy/data-egress requirements.

---

## A/B testing

Where product experimentation is appropriate:

- record assignment;
- separate evaluation from authorization;
- preserve privacy;
- ensure baseline security controls remain enforced.

Do not use experimentation to weaken critical safety controls.

---

## Rollback

Rollback should restore a known-good immutable model/runtime combination.

Do not "rollback" by resolving a mutable alias.

Know whether:

- cache state;
- tokenizer;
- adapter;
- runtime config

must also roll back.

---

## Model compatibility

Define compatibility between:

- weights;
- tokenizer;
- runtime;
- adapter;
- quantization;
- serving image.

Validate combinations before production.

---

## API versioning

If the model-serving API is consumed by applications, preserve contract
compatibility.

Model changes should not silently break:

- response schema;
- streaming behavior;
- error semantics;
- token accounting fields.

---

## OpenAI-compatible or generic APIs

If implementing a compatibility API, document exactly which subset is
supported.

Do not claim compatibility while silently differing on important semantics.

---

## Structured output support

If the runtime supports constrained or structured output:

- test schema enforcement;
- test malformed-schema behavior;
- measure latency/quality impact.

Do not assume "JSON mode" guarantees semantic validity.

---

## Function/tool calling

If the serving layer exposes tool-call structured output, treat it as model
output.

Applications must still:

- validate;
- authorize;
- constrain side effects.

The serving runtime should not execute application tools directly unless that
is explicitly part of its architecture.

---

## Model gateway

If the serving system acts as a gateway over multiple model backends:

- authenticate callers;
- authorize model access;
- route deterministically;
- record selected backend;
- isolate credentials;
- normalize errors carefully.

Do not hide model substitution from operators.

---

## Fallback routing

Fallback may route to:

- smaller model;
- alternate runtime;
- alternate region.

Define when this is allowed.

Do not silently fall back when model behavior or policy constraints differ
materially.

---

## Regional serving

If serving across regions:

- define data residency;
- model availability;
- failover policy;
- routing.

Do not route sensitive requests cross-region without understanding policy.

---

## Observability

Track at minimum the metrics relevant to the serving architecture.

Potential signals:

- request count;
- error rate;
- queue depth;
- queue latency;
- inference latency;
- first-token latency;
- input tokens;
- output tokens;
- tokens/sec;
- active sequences;
- batch size;
- accelerator utilization;
- accelerator memory;
- OOM events;
- model-load failures;
- cancellation;
- rate limits;
- fallback rate.

---

## Model-specific metrics

Include model identity in telemetry in a bounded way.

Track:

- model version;
- runtime version;
- deployment/canary variant.

Avoid unbounded labels from user input.

---

## Logging

Log metadata, not raw content by default.

Useful fields:

- request ID;
- model;
- runtime;
- latency;
- token counts;
- result status;
- error category;
- queue duration.

Do not log full prompts/outputs unless deliberately enabled.

---

## Tracing

Tracing may include:

- admission;
- queue;
- model invocation;
- streaming;
- downstream gateway hops.

Do not put raw user content into traces by default.

---

## Audit

Administrative actions should be attributable.

Examples:

- model load;
- model unload;
- routing change;
- config change;
- production promotion;
- emergency rollback.

---

## Alerting

Alerts should focus on actionable failure modes.

Examples:

- rising OOM;
- model load failures;
- queue saturation;
- high error rate;
- severe latency regression;
- capacity exhaustion;
- repeated fallback;
- artifact verification failure.

Avoid alerting on every transient spike.

---

## SLOs

Define SLOs only when the product has actual service objectives.

Possible dimensions:

- availability;
- latency;
- first-token latency;
- successful request rate.

Do not invent SLO numbers without requirements.

---

## Disaster recovery

For critical serving:

- identify model artifact recovery;
- runtime image recovery;
- configuration recovery;
- capacity replacement;
- regional failover where required.

If artifacts are fully reproducible and stored durably, rebuild may be the DR
strategy.

---

## Backup

Model artifacts usually belong in durable artifact storage.

Do not rely on serving-node local disks as the only copy.

Caches generally do not need backup if reconstructable.

---

## Upgrades

Treat changes to:

- drivers;
- accelerator runtime;
- serving framework;
- compiler;
- quantization library;
- kernel library

as potentially behavior- and performance-affecting.

Test them.

Do not auto-upgrade critical GPU/runtime components blindly.

---

## Driver/runtime compatibility

Track compatibility between:

- host driver;
- container runtime;
- CUDA/ROCm;
- serving framework;
- model kernel/extension.

Infrastructure upgrades can break inference independently of application code.

---

## Dependency pinning

Pin critical serving dependencies sufficiently for reproducibility.

Avoid mutable container tags and floating Python packages in production
serving images.

---

## Security patching

Serving runtimes may lag because accelerator stacks are complex.

Maintain an explicit patch process.

Do not indefinitely freeze vulnerable runtime components because upgrades are
inconvenient.

---

## Custom kernels/extensions

Treat custom kernels as privileged native code.

Review and test:

- source;
- build process;
- compatibility;
- failure behavior.

A custom CUDA/ROCm extension is part of the trusted compute base.

---

## Local development

Developers should be able to test serving logic without requiring production
accelerators for every change.

Possible approaches:

- CPU mode;
- tiny test model;
- mocked runtime;
- local lightweight inference engine.

Keep at least some real-runtime integration coverage.

---

## Test model

Use a deliberately small, redistributable test model where feasible.

Do not use large proprietary model artifacts in ordinary unit tests.

---

## Deterministic tests

Test:

- config validation;
- request validation;
- authn/authz;
- scheduler rules;
- quota logic;
- routing;
- error mapping;
- model-selection logic.

---

## Integration tests

Exercise:

- model load;
- tokenizer load;
- inference;
- streaming;
- cancellation;
- readiness;
- artifact verification.

Use a representative small model where possible.

---

## Load tests

Use representative request shapes.

Include:

- short context;
- long context;
- high concurrency;
- streaming;
- cancellation;
- burst load.

Measure queueing and saturation.

---

## Soak tests

For long-running serving stacks, soak testing may reveal:

- memory leaks;
- fragmentation;
- cache growth;
- degradation;
- driver instability.

Use soak tests where operational risk justifies them.

---

## Fault injection

Consider fault tests for:

- model artifact missing;
- accelerator unavailable;
- worker crash;
- registry unavailable;
- disk full;
- OOM;
- config error.

The system should fail predictably.

---

## Security tests

Test:

- unauthorized model access;
- admin API denial;
- oversized requests;
- malformed parameters;
- cross-tenant cache isolation;
- model artifact integrity failure.

---

## Negative-path tests

At least one meaningful failure should be exercised end to end.

Examples:

- invalid model digest rejected;
- unauthorized caller denied;
- oversized context rejected;
- model load failure keeps instance unready;
- queue saturation returns controlled rejection;
- unsigned/untrusted model artifact rejected where policy requires.

---

## Benchmark reproducibility

Benchmark reports should record:

- model;
- runtime;
- hardware;
- batch settings;
- context length;
- output length;
- concurrency;
- quantization;
- software versions.

Do not publish throughput numbers without workload context.

---

## Performance regression gates

For important serving systems, define acceptable performance regression
thresholds.

Do not invent thresholds without baseline evidence.

Track both:

- throughput;
- tail latency.

---

## Cost evaluation

For owned infrastructure, estimate:

- accelerator-hours;
- idle capacity;
- throughput per device;
- cost per request/token where useful.

Do not optimize only for utilization if latency collapses under peak load.

---

## Energy/resource efficiency

Where relevant, compare model variants by useful work per resource.

Do not make environmental claims without measurement.

---

## Documentation

The repository should document:

- supported models;
- artifact format;
- model registry;
- runtime;
- hardware assumptions;
- API;
- authn/authz;
- capacity limits;
- configuration;
- rollout;
- rollback;
- observability;
- known limitations.

Operators should be able to answer:

> What exact model/runtime combination is serving this request?

---

## Architecture decisions

Use ADRs for consequential choices such as:

- serving runtime;
- model artifact format;
- GPU scheduling;
- multi-model architecture;
- quantization;
- model gateway;
- caching;
- autoscaling;
- rollout strategy.

Do not create ADRs for every tuning knob.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic model-serving repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── src/
│   ├── api/
│   ├── runtime/
│   ├── scheduler/
│   ├── routing/
│   ├── auth/
│   ├── artifacts/
│   └── observability/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── load/
│   └── security/
├── benchmarks/
├── models/
│   └── metadata/
├── docs/
│   ├── architecture/
│   └── adr/
└── <ecosystem build/dependency files>
```

Do not commit large model weights directly unless the repository intentionally
uses an artifact mechanism that supports them.

---

## Verification interface

A model-serving repository should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-integration
test-security
benchmark
load-test
verify-model
run
```

These names are illustrative.

Use the ecosystem's native tooling where appropriate.

---

## Acceptance criteria

A model-serving system is not complete because a model successfully generated
one response.

Demonstrate the applicable subset of the following.

### Artifact integrity

- model artifact has immutable identity;
- tokenizer/config identity is known;
- artifact integrity is verified;
- runtime image/version is identifiable.

### Serving

- model loads successfully;
- readiness reflects load state;
- inference succeeds;
- streaming/cancellation works where supported;
- shutdown drains safely.

### Capacity

- input/output limits are enforced;
- queue is bounded;
- overload behavior is controlled;
- concurrency limits work;
- OOM behavior is understood.

### Security

- unauthorized model access is rejected;
- admin operations are protected;
- secrets are not logged;
- model artifacts cannot be loaded arbitrarily;
- cross-tenant state/caches are isolated where applicable.

### Performance

- representative benchmark exists;
- tail latency is measured;
- throughput is measured;
- hardware/runtime metadata is captured.

### Evaluation

- promoted model has behavioral evaluation evidence;
- runtime/quantization changes are evaluated where behavior can change;
- rollback target is known.

### Operations

- model/runtime metrics exist;
- model identity appears in telemetry;
- load failures are observable;
- rollback procedure exists;
- artifact source is durable.

---

## Optional composition

Common combinations:

```text
model-serving + ai-service
```

For application-facing AI services that also own inference infrastructure.

```text
model-serving + software-supply-chain
```

For signed/verifiable models, serving images, provenance, and promotion.

```text
model-serving + zero-trust-service
```

For strongly authenticated internal inference endpoints and scoped model access.

```text
model-serving + ai-evaluation
```

For behavioral and performance release gates.

```text
model-serving + high-assurance
```

For deeper fault injection, independent artifact verification, and strict
rollout controls.

---

## Anti-patterns

Avoid:

- production model references by mutable alias only;
- downloading arbitrary public model code at startup;
- unverified model artifacts;
- model weights on ephemeral local disk as the only copy;
- admin and inference APIs with identical permissions;
- one static API key for all callers;
- logging every prompt/response by default;
- unbounded context length;
- unbounded generation;
- unbounded queue depth;
- CPU-only autoscaling signals for GPU saturation;
- claiming readiness before model load completes;
- treating average latency as sufficient performance evidence;
- production quantization changes with no eval;
- mixing incompatible tokenizers/models;
- silently routing to fallback models;
- scale-to-zero with undocumented cold-start impact;
- unrestricted model loading by clients;
- treating GPU access as harmless container privilege;
- publishing benchmark numbers without workload/hardware context;
- confusing server health with model quality.

Do not mistake successful inference for production-grade serving.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. model artifact format and identity;
2. tokenizer/config identity;
3. model registry/source;
4. integrity/provenance controls;
5. serving runtime/version;
6. hardware/accelerator assumptions;
7. authentication/authorization model;
8. request/context/output limits;
9. batching/scheduling strategy;
10. rate/quota/backpressure behavior;
11. caching/isolation model;
12. readiness/liveness/shutdown behavior;
13. rollout/rollback model;
14. observability signals;
15. performance benchmarks executed;
16. integration/security tests executed;
17. behavioral evaluations executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a serving system is "production-ready", "fast", "secure", or
"reliable" based only on one successful inference or one synthetic benchmark.

Describe the concrete artifact, runtime, capacity, security, rollout, and
verification evidence.

---

## Guiding principle

A production model server should make model execution boring.

The artifact should be known.

The runtime should be known.

The caller should be known.

The limits should be known.

The capacity should be bounded.

The overload behavior should be intentional.

The rollout should be reversible.

The telemetry should explain what is happening.

And the system should always be able to answer:

> Which exact model, tokenizer, runtime, and configuration produced this
> response, under what policy, on what infrastructure, and with what evidence
> that this was the intended deployment?
