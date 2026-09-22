# ai-evaluation.md

> Evaluation profile for AI, LLM, RAG, and agentic systems.
>
> Apply under `AGENTS.md`, together with `SCAFFOLD.md` or `ELEVATE.md`, and
> compose with recipes such as `ai-service.md`, `agentic-system.md`,
> `rag-system.md`, `mcp-tool-server.md`, or `security-tool.md`.
>
> This profile defines how AI behavior should be evaluated, compared, gated,
> monitored, and regression-tested.
>
> It does not require a specific model vendor, evaluation framework, judge
> model, dataset platform, benchmark suite, or observability product.

## Purpose

Use this profile when software behavior materially depends on a probabilistic
model.

Typical examples:

- LLM-backed services;
- RAG systems;
- tool-using agents;
- copilots;
- multimodal systems;
- classifiers using foundation models;
- model-assisted security tools;
- autonomous or semi-autonomous workflows.

The goal is not to produce a single "AI score".

The goal is to establish evidence that answers:

- Does the system perform the intended task?
- What kinds of mistakes does it make?
- Did a model/prompt/tool/retrieval change improve or regress behavior?
- Does it respect security and policy boundaries?
- Does it abstain appropriately?
- How stable is behavior across repeated runs?
- What does success cost in latency, tokens, money, and operations?
- What failures should block release?

---

## Core principle

AI evaluation is not equivalent to conventional testing.

A useful distinction is:

```text
tests
    = deterministic software correctness

evals
    = probabilistic behavioral evidence
```

Both are required.

Do not replace:

- unit tests;
- integration tests;
- policy tests;
- authorization tests;
- schema validation

with model-based evaluation.

Conversely, do not assume conventional tests can establish whether a
probabilistic system remains useful, grounded, robust, or safe.

---

## Evaluation invariants

An AI evaluation system SHOULD satisfy these invariants unless the repository
has a documented reason not to.

### 1. Evaluation is versioned

Every meaningful evaluation result should be attributable to:

- application version;
- model/provider;
- model version/revision where available;
- prompt/template version;
- tool schema/version where applicable;
- retrieval/index version where applicable;
- evaluator version;
- dataset version;
- inference parameters.

Do not compare runs whose material configuration is unknown.

### 2. Evaluation datasets are treated as code/data assets

Evaluation datasets should be:

- version controlled or immutably versioned;
- reviewable;
- documented;
- reproducible;
- protected from accidental contamination.

Do not keep critical eval cases only in spreadsheets, chat histories, or
people's heads.

### 3. Aggregate scores do not hide critical failures

A high average score must not compensate for:

- authorization bypass;
- secret leakage;
- destructive action;
- fabricated citation;
- cross-tenant access;
- unsafe tool use;
- catastrophic factual error.

Critical cases should have explicit gates.

### 4. Security behavior has deterministic expectations

Where the expected security outcome is clear, evaluate it as pass/fail.

Examples:

```text
unauthorized tool invocation -> must be denied
wrong tenant retrieval       -> must be denied
invalid approval             -> must be denied
secret extraction attempt    -> must not disclose secret
```

Do not reduce hard security boundaries to a soft "quality score".

### 5. Evaluation measures components and end-to-end behavior

Where architecture permits, evaluate:

- model output;
- retrieval;
- tool selection;
- tool arguments;
- policy enforcement;
- citations;
- final task outcome.

Do not evaluate only the final prose response if the system contains multiple
failure points.

### 6. Nondeterminism is measured, not ignored

For stochastic behavior, repeated runs may be required.

Report:

- variance;
- failure frequency;
- tail cases;
- worst-case behavior where meaningful.

One lucky run is weak evidence.

### 7. Human judgment and model judgment are distinct

Human evaluation may be necessary.

LLM-as-judge may be useful.

Neither should be treated as unquestionable ground truth.

### 8. Eval leakage invalidates evidence

Do not tune prompts/models against the same dataset indefinitely and continue
calling that dataset an unbiased evaluation set.

Separate development and evaluation data where practical.

### 9. Negative examples matter

Evaluation should include:

- cases that should succeed;
- cases that should fail;
- cases that should abstain;
- ambiguous cases;
- adversarial cases.

A system that only succeeds on happy-path prompts has not been meaningfully
evaluated.

### 10. Release gates are explicit

Define which regressions block release.

Do not inspect a dashboard manually after every change and call that a release
policy.

---

## Evaluation layers

Think about evaluation at several layers.

### Layer 1 — deterministic software

Evaluate with conventional tests:

- parsing;
- schemas;
- API behavior;
- policy;
- tool argument validation;
- authn/authz;
- state machines;
- budgets;
- retries;
- timeouts.

### Layer 2 — model behavior

Evaluate:

- task correctness;
- instruction following;
- output structure;
- factuality;
- refusal;
- uncertainty;
- consistency.

### Layer 3 — subsystem behavior

Examples:

- retrieval relevance;
- citation correctness;
- tool selection;
- tool argument correctness;
- memory behavior;
- planner behavior.

### Layer 4 — end-to-end outcomes

Evaluate whether the full system achieved the user's task safely and correctly.

### Layer 5 — operational behavior

Evaluate:

- latency;
- tokens;
- cost;
- rate-limit behavior;
- timeout behavior;
- fallback behavior;
- failure recovery.

---

## Evaluation taxonomy

Do not force every system into the same metrics.

Potential categories include:

- task success;
- factual correctness;
- groundedness;
- completeness;
- relevance;
- structured-output validity;
- citation correctness;
- refusal correctness;
- abstention correctness;
- authorization compliance;
- tool-use correctness;
- side-effect correctness;
- safety;
- privacy;
- prompt-injection resistance;
- robustness;
- latency;
- token consumption;
- monetary cost.

Choose the smallest meaningful set.

---

## Scenario definition

Prefer scenario-based evaluation over vague prompt lists.

A scenario should include:

- name;
- objective;
- inputs;
- identity/security context;
- environment;
- expected behavior;
- prohibited behavior;
- expected evidence;
- scoring/gating method.

Example:

```yaml
name: cross-tenant-retrieval-denied

input:
  user: tenant-a-user
  query: "Show me tenant B's incident report"

expected:
  outcome: deny
  prohibited:
    - content from tenant-b

gate:
  type: deterministic
  result: must-pass
```

---

## Expected outcomes

A scenario may expect:

```text
SUCCESS
DENY
REFUSE
ABSTAIN
ESCALATE
APPROVAL_REQUIRED
FAIL_SAFE
```

Do not assume every desirable behavior means "answer the request".

---

## Dataset composition

A useful evaluation set may contain:

- normal representative cases;
- boundary cases;
- long-context cases;
- malformed inputs;
- ambiguous instructions;
- known historical failures;
- adversarial cases;
- rare but high-impact cases.

Bias the dataset toward the actual risk and product behavior.

Do not create thousands of synthetic cases merely to inflate coverage.

---

## Development versus holdout sets

Where practical, separate:

```text
development set
    = used during prompt/model iteration

holdout set
    = used for less-biased release evaluation
```

If the holdout set becomes heavily optimized against, refresh it.

Do not pretend a repeatedly inspected benchmark is still independent evidence.

---

## Regression corpus

Every discovered material AI failure should become a regression case where
practical.

Examples:

- hallucinated source;
- wrong tool;
- missed approval;
- prompt injection;
- cross-tenant retrieval;
- malformed structured output;
- runaway loop;
- unsafe fallback;
- secret leakage.

Regression scenarios should remain after the bug is fixed.

---

## Golden examples

Golden examples may define expected:

- output;
- evidence;
- citation;
- tool sequence;
- decision.

Use exact-match assertions only where exact output is meaningful.

For open-ended generation, prefer semantic or rubric-based checks.

Do not force deterministic string equality onto naturally variable outputs.

---

## Deterministic assertions

Use deterministic assertions whenever possible.

Examples:

- valid JSON;
- exact schema;
- tool call count <= 5;
- no write tool invoked;
- citation source exists;
- no cross-tenant document IDs;
- approval required before action;
- answer contains no secret token.

Hard invariants should not be delegated to a fuzzy judge.

---

## Model-based scoring

Use model scoring for dimensions that genuinely require interpretation.

Possible uses:

- semantic correctness;
- relevance;
- clarity;
- groundedness;
- helpfulness.

Do not use model scoring for things easily checked deterministically.

---

## LLM-as-judge

If using a judge model:

- version the judge model;
- version the judge prompt;
- validate against human-labeled examples;
- inspect disagreement;
- monitor judge drift;
- use structured rubric output where possible.

Do not use the same model under test as the sole judge of its own behavior when
independence matters.

---

## Judge calibration

Before trusting a judge:

1. create a human-labeled sample;
2. run the judge;
3. compare agreement;
4. inspect false positives/negatives;
5. adjust rubric;
6. repeat.

Report calibration limits.

Do not say "judge score 0.93 means 93% correct".

---

## Human evaluation

Use human review where:

- semantics are ambiguous;
- subtle harmful behavior matters;
- domain expertise is required;
- automated judges are unreliable;
- product quality is subjective.

Provide reviewers with:

- consistent rubric;
- blind comparison where useful;
- examples;
- disagreement handling.

---

## Inter-rater agreement

For important human-evaluated datasets, measure agreement.

Low agreement may mean:

- ambiguous task;
- weak rubric;
- insufficient reviewer expertise;
- genuinely subjective behavior.

Do not hide disagreement behind averaged scores.

---

## Pairwise evaluation

Pairwise comparison can be useful when comparing:

- model A vs B;
- prompt A vs B;
- retrieval strategy A vs B.

Randomize presentation where human/judge bias matters.

Do not infer absolute quality from pairwise preference alone.

---

## Reference-based evaluation

Use reference answers when the task has stable ground truth.

Examples:

- extraction;
- classification;
- exact lookup;
- structured transformation.

Potential metrics:

- exact match;
- F1;
- precision/recall;
- field accuracy.

Do not force reference-answer scoring onto open-ended tasks where many answers
are valid.

---

## Groundedness

For source-grounded systems, evaluate whether claims are supported by provided
evidence.

Possible checks:

- claim supported;
- claim contradicted;
- claim unsupported;
- citation missing;
- citation irrelevant.

Separate:

```text
answer correctness
```

from:

```text
answer grounded in supplied source
```

They are related but not identical.

---

## Citation correctness

Evaluate citations independently.

Check:

- cited source exists;
- source was available to the system;
- citation supports the claim;
- citation location is correct;
- citation is not fabricated.

Fabricated citations should be a hard failure where citations are presented as
evidence.

---

## Retrieval evaluation

For RAG systems, evaluate retrieval separately.

Potential metrics:

- recall@k;
- precision@k;
- MRR;
- nDCG;
- hit rate.

Also evaluate:

- unauthorized result exclusion;
- source freshness;
- authoritative-source preference;
- poisoned-content handling.

High relevance does not compensate for authorization failure.

---

## Tool-use evaluation

For agentic systems, evaluate:

- correct tool selected;
- unnecessary tools avoided;
- arguments correct;
- resource scope correct;
- tool sequence correct where needed;
- side-effect class respected;
- approval obtained where required.

Do not score "task completed" as success if the agent used an unauthorized
route.

---

## Side-effect evaluation

Where tools modify state, verify the actual effect.

Evaluate:

- intended change occurred;
- unintended changes did not occur;
- idempotency;
- rollback/recovery where relevant.

Do not score based only on the agent's claim that the action succeeded.

---

## Policy compliance

Evaluate policy behavior directly.

Examples:

- permission denied;
- approval required;
- environment boundary enforced;
- budget enforced;
- kill switch works.

These should usually be deterministic gates.

---

## Refusal evaluation

Evaluate both:

```text
should refuse
```

and:

```text
should not refuse
```

Over-refusal is a product failure.

Under-refusal may be a safety/security failure.

Measure both.

---

## Abstention evaluation

For factual systems, include cases where available evidence is insufficient.

Evaluate whether the system:

- abstains;
- asks for clarification;
- states uncertainty;
- fabricates.

Reward appropriate uncertainty.

---

## Calibration

If the system exposes confidence, test calibration.

Compare confidence to observed correctness.

Do not rely on model self-reported confidence without empirical validation.

---

## Consistency

Repeated runs can reveal unstable behavior.

For selected scenarios:

- run multiple samples;
- measure outcome variance;
- track critical-failure frequency.

A system that passes 9/10 times may still be unacceptable for a high-impact
security boundary.

---

## Temperature and sampling

Record inference parameters.

When evaluating stochastic production settings, use parameters representative
of production.

Do not evaluate at temperature 0 and deploy at temperature 1 without
recognizing the mismatch.

---

## Seed handling

Where the provider supports seeds, they may improve reproducibility.

Do not mistake a seed for a guarantee of identical outputs across model
versions/providers.

---

## Statistical confidence

For metrics intended to drive release decisions, use enough samples to make the
comparison meaningful.

Consider:

- sample size;
- confidence intervals;
- effect size;
- failure count.

Do not overinterpret tiny score changes on tiny datasets.

---

## Critical-case weighting

Use explicit hard gates for critical scenarios rather than burying them in
weighted averages.

Example:

```text
overall_quality >= threshold
AND
cross_tenant_failures == 0
AND
unauthorized_tool_calls == 0
AND
secret_leaks == 0
```

---

## Security evaluation

Security evals should include threat-informed scenarios.

Potential categories:

- direct prompt injection;
- indirect prompt injection;
- tool-result injection;
- secret extraction;
- authorization bypass;
- cross-tenant access;
- environment escalation;
- approval bypass;
- memory poisoning;
- malicious retrieval;
- tool schema manipulation;
- data exfiltration;
- unsafe generated code;
- SSRF-inducing output.

Use the relevant recipe/profile threat model.

---

## Prompt injection evaluation

Test both:

### Direct injection

User explicitly attempts to override system instructions.

### Indirect injection

Malicious instructions arrive through:

- web content;
- documents;
- email;
- retrieved knowledge;
- code;
- tool output.

Indirect cases are often more representative for agents and RAG.

---

## Data leakage evaluation

Create controlled canary secrets or synthetic sensitive values.

Test whether they are exposed through:

- model output;
- retrieval;
- tools;
- logs;
- citations;
- memory.

Do not use real production secrets as eval fixtures.

---

## Cross-tenant evaluation

For multi-tenant systems, include cases where relevant unauthorized data exists.

Test that:

- retrieval excludes it;
- cache does not leak it;
- memory does not leak it;
- tools cannot access it;
- model does not infer access from user text.

---

## Authorization evaluation

Test:

- no identity;
- invalid identity;
- wrong scope;
- wrong tenant;
- wrong environment;
- insufficient privilege;
- expired approval;
- revoked access.

Security boundaries should be tested as expected failures.

---

## Adversarial mutation

Create variants of known attacks.

Examples:

- paraphrases;
- Unicode tricks;
- encoding changes;
- whitespace changes;
- roleplay framing;
- nested instructions;
- multilingual variants.

Do not overfit defenses to one exact attack string.

---

## Robustness evaluation

Test behavior under:

- long inputs;
- malformed inputs;
- missing fields;
- provider timeout;
- partial retrieval;
- stale data;
- irrelevant context;
- contradictory context;
- noisy tool output.

---

## Context-length evaluation

Include cases near:

- normal context;
- high context;
- maximum supported context.

Check that truncation does not remove:

- security policy;
- user objective;
- critical evidence.

---

## Latency evaluation

Measure:

- end-to-end latency;
- model latency;
- retrieval latency;
- tool latency;
- first-token latency where relevant.

Use percentile metrics where useful.

Do not report only average latency.

---

## Cost evaluation

Measure actual or estimated:

- input tokens;
- output tokens;
- model spend;
- embedding spend;
- tool/API spend;
- compute cost.

Track cost per successful task where meaningful.

A cheaper system that fails more often may not be cheaper operationally.

---

## Efficiency

Useful metrics may include:

```text
cost / successful task
tokens / successful task
tool calls / successful task
seconds / successful task
```

Avoid optimizing raw token count if it harms correctness.

---

## Budget evaluation

For agents, test that run budgets work.

Examples:

- max steps;
- max tokens;
- max cost;
- max tool calls;
- max runtime.

The model must not be able to override these limits.

---

## Failure recovery evaluation

Test behavior when dependencies fail.

Examples:

- model unavailable;
- retrieval unavailable;
- policy engine unavailable;
- tool timeout;
- unknown write outcome;
- provider rate limit.

Evaluate whether the system:

- fails safely;
- retries correctly;
- preserves state;
- avoids duplicate side effects.

---

## Fallback evaluation

If fallback models/providers exist, evaluate them independently.

Test:

- behavior difference;
- safety difference;
- cost;
- latency;
- structured output reliability.

Do not assume fallback is equivalent.

---

## Model comparison

When comparing models, evaluate on the same:

- dataset;
- prompt;
- tool definitions;
- retrieval state;
- parameters

unless the experiment intentionally changes them.

Document all changed variables.

---

## Prompt comparison

When comparing prompts, hold other variables stable where possible.

Prompt changes should be evaluated against:

- task quality;
- refusal behavior;
- security cases;
- token cost;
- latency if prompt length matters.

---

## Retrieval comparison

Compare retrieval changes on the same corpus and authorization context.

Evaluate both:

- quality;
- security.

Do not promote a retrieval strategy that improves recall by leaking protected
documents.

---

## Tool schema comparison

Tool schema changes may alter model behavior.

Evaluate:

- tool selection;
- argument validity;
- unnecessary calls;
- unsafe calls.

Treat tool descriptions as behavior-affecting artifacts.

---

## Evaluation harness design

A good harness should support:

- repeatable runs;
- scenario selection;
- concurrency control;
- retries only where appropriate;
- artifact/result output;
- metadata capture;
- machine-readable summaries;
- diffing against baseline.

Do not build an enormous framework if a simple script can satisfy requirements.

---

## Result format

Store structured results.

A result record may include:

```json
{
  "scenario": "cross-tenant-deny",
  "run_id": "...",
  "model": "...",
  "prompt_version": "...",
  "dataset_version": "...",
  "outcome": "pass",
  "metrics": {},
  "artifacts": {}
}
```

Do not rely only on terminal output.

---

## Evaluation provenance

Capture enough metadata to reproduce or interpret a run.

Examples:

- source commit;
- model;
- prompt hash/version;
- tool schema hash/version;
- dataset version;
- evaluator version;
- index version;
- timestamp;
- environment.

---

## Baselines

Use baselines to detect regression.

A baseline may be:

- previous release;
- production model;
- known stable prompt;
- deterministic reference implementation.

Do not treat the baseline as perfect.

Report both regressions and improvements.

---

## Regression thresholds

Define thresholds intentionally.

Examples:

```text
task_success must not drop > 2 percentage points
p95 latency must not rise > 15%
security critical failures must remain 0
```

These numbers are examples only.

Do not invent organizational thresholds without evidence.

---

## Release gating

A release gate may combine:

- deterministic tests;
- AI eval thresholds;
- zero critical security failures;
- latency/cost bounds.

Example:

```text
release allowed if:

    deterministic tests pass
    AND critical security scenarios pass
    AND task success >= accepted threshold
    AND no material regression in cost/latency
```

Keep gates understandable.

---

## Warning versus blocking evals

Not every eval should block release.

Classify:

```text
BLOCKING
WARNING
INFORMATIONAL
```

Use blocking gates for behavior that truly cannot regress.

Avoid making noisy metrics hard blockers before they are trustworthy.

---

## Flaky evals

A flaky evaluation can make release policy unusable.

For stochastic cases:

- use repeated runs;
- aggregate carefully;
- isolate deterministic failures;
- inspect variance.

Do not simply rerun until green.

---

## Eval failures

Distinguish:

```text
system failed scenario
```

from:

```text
evaluation infrastructure failed
```

Provider outage, broken judge, or corrupted dataset should not become a
behavioral score.

---

## Missing coverage

Report scenarios that were skipped or could not execute.

Do not present partial evaluation as complete.

---

## CI integration

Fast deterministic evals may run on every PR.

Expensive model evals may run:

- on model/prompt changes;
- on release branches;
- nightly;
- before production promotion.

Choose cadence based on cost and risk.

Do not run expensive evals on every trivial documentation change without need.

---

## Change-based eval selection

Trigger relevant eval suites based on changed components.

Examples:

```text
prompt changed      -> prompt regression + security evals
retriever changed   -> retrieval + grounding evals
tool schema changed -> tool-use + policy evals
model changed       -> broad regression suite
```

---

## Production evaluation

Offline evals are not enough.

Where appropriate, monitor production outcomes through:

- user feedback;
- escalation rate;
- correction rate;
- policy denials;
- task completion;
- incident reports.

Do not use sensitive production transcripts as eval data without governance.

---

## Shadow evaluation

A candidate model/system may run in shadow mode.

Compare outputs without affecting users.

Use this to evaluate:

- quality;
- latency;
- tool decisions;
- policy outcomes.

Do not allow shadow systems to perform real side effects.

---

## Canary evaluation

For staged rollout, compare candidate and baseline on live traffic where
appropriate.

Protect:

- user privacy;
- tenant isolation;
- security policy.

Critical security controls should not be experimental.

---

## Feedback loops

User feedback can inform eval development.

Distinguish:

- preference;
- factual correction;
- security incident;
- UX issue.

Do not equate thumbs-up with correctness.

---

## Production incidents

Every material AI incident should feed back into the evaluation corpus.

Examples:

- hallucinated high-impact answer;
- prompt injection;
- wrong tool;
- secret leak;
- cross-tenant disclosure;
- runaway cost.

Convert incidents into regression cases.

---

## Dataset governance

Evaluation datasets may contain sensitive content.

Apply:

- access control;
- redaction;
- retention;
- tenant isolation;
- provenance.

Do not copy production data casually into test repositories.

---

## Synthetic data

Synthetic eval data can be useful.

Use it for:

- adversarial cases;
- edge cases;
- privacy-preserving fixtures.

But validate that synthetic cases resemble real failure modes.

Do not assume synthetic performance equals production performance.

---

## Annotation quality

For labeled datasets:

- document labeling instructions;
- track annotator agreement where useful;
- review ambiguous examples;
- preserve source provenance.

Poor labels produce misleading evals.

---

## Benchmark contamination

Public benchmark data may be present in model training.

Treat public benchmark scores cautiously.

Prefer application-specific evals for release decisions.

---

## Domain expertise

For specialized systems, include domain experts in evaluation design.

Examples:

- security;
- medicine;
- law;
- finance;
- infrastructure.

Do not assume generic judges can assess specialist correctness reliably.

---

## Evaluation of explanations

Do not evaluate hidden chain-of-thought.

Evaluate observable artifacts such as:

- final answer;
- citations;
- plan summary;
- tool calls;
- action log;
- decision rationale.

Operational explainability does not require access to private reasoning.

---

## Auditability

For important evaluation runs, preserve:

- config;
- result summary;
- critical failures;
- metadata;
- artifacts necessary to reproduce findings.

Do not rely on ephemeral notebook output as release evidence.

---

## Reproducibility

Exact reproduction may be impossible with closed, evolving models.

Aim instead for interpretability:

- same scenario;
- same config;
- known model identity;
- known time;
- repeated sampling.

Document nondeterministic limits.

---

## Evaluation reports

A useful release report should summarize:

- what changed;
- what was evaluated;
- baseline;
- important regressions;
- important improvements;
- security failures;
- cost/latency changes;
- skipped tests;
- known uncertainty.

Avoid one giant score.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic evaluation structure may resemble:

```text
.
├── evals/
│   ├── datasets/
│   ├── scenarios/
│   ├── adversarial/
│   ├── judges/
│   ├── baselines/
│   └── reports/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── security/
├── scripts/
├── docs/
│   └── evaluation.md
└── <ecosystem-specific files>
```

Only create directories that contain meaningful content.

---

## Verification interface

A repository applying this profile should expose clear commands or equivalent
native interfaces for:

```text
test
eval
eval-regression
eval-security
eval-adversarial
eval-cost
eval-latency
eval-report
```

These names are illustrative.

Prefer the repository's existing task runner.

---

## Acceptance criteria

An AI evaluation system is not complete because it can run prompts against a
model.

Demonstrate the applicable subset of the following.

### Versioning

- model identity captured;
- prompt/config version captured;
- dataset version captured;
- evaluator version captured;
- retrieval/tool versions captured where relevant.

### Dataset quality

- representative cases exist;
- negative cases exist;
- adversarial cases exist where applicable;
- historical failures become regressions;
- sensitive data is handled appropriately.

### Deterministic controls

- schemas/policy/authz/tool limits tested conventionally;
- hard security boundaries have pass/fail assertions;
- evals do not replace deterministic tests.

### Behavioral eval

- task success measured;
- important failure modes measured;
- repeated sampling used where warranted;
- variance is visible.

### Security eval

- prompt injection cases exist where applicable;
- authorization cases exist;
- cross-tenant cases exist where applicable;
- secret leakage cases exist where applicable;
- tool/approval abuse cases exist for agents.

### Quality of judging

- automated judges are versioned;
- judge rubric is explicit;
- judge quality is checked against human labels where important.

### Release policy

- blocking versus non-blocking metrics are explicit;
- critical scenarios cannot be hidden by averages;
- regression thresholds are documented;
- skipped evals are visible.

### Operations

- latency is measured where relevant;
- token/cost usage is measured;
- provider/eval infrastructure failure is distinguishable from product failure.

---

## Optional composition

Common combinations:

```text
ai-service + ai-evaluation
```

For model-backed APIs with release-quality regression evidence.

```text
agentic-system + ai-evaluation
```

For tool selection, authorization, approval, prompt-injection, and side-effect
evaluation.

```text
rag-system + ai-evaluation
```

For retrieval, grounding, citation, poisoning, and authorization evaluation.

```text
mcp-tool-server + ai-evaluation
```

For evaluating whether agents select and use exposed tools correctly.

```text
ai-evaluation + high-assurance
```

For stronger independence, deeper adversarial coverage, and more rigorous
release gates.

---

## Anti-patterns

Avoid:

- one aggregate "AI quality" score;
- evaluating only happy paths;
- using exact string match for open-ended tasks;
- using only subjective judge scores for deterministic security controls;
- same model judging itself with no calibration;
- hiding critical failures in averages;
- rerunning flaky evals until green;
- treating provider failure as a product failure;
- treating product failure as evaluator failure;
- using production secrets in eval fixtures;
- allowing cross-tenant data into shared eval corpora;
- tuning indefinitely on the holdout set;
- claiming benchmark gains as product gains without application-specific evidence;
- comparing model runs with different prompts/datasets and calling it a model-only comparison;
- release gates based on unstable metrics;
- ignoring cost and latency regressions;
- assuming one successful run proves reliability;
- evaluating agents only on final answer quality while ignoring unauthorized actions.

Do not mistake measurement for understanding.

---

## Completion evidence

When this profile is applied, the final report should state:

1. evaluation objectives;
2. dataset/scenario inventory;
3. dataset versioning model;
4. model/prompt/tool/retrieval versions captured;
5. deterministic checks used;
6. behavioral metrics used;
7. security/adversarial scenarios;
8. judge model/rubric where used;
9. human-evaluation process where used;
10. sampling/repetition strategy;
11. baseline;
12. regression thresholds;
13. blocking release gates;
14. latency/cost metrics;
15. eval commands actually executed;
16. observed results;
17. skipped or failed eval infrastructure;
18. known coverage gaps;
19. known evaluator limitations;
20. deliberate omissions and why.

Never claim the system is "safe", "accurate", "robust", "grounded", or
"production-ready" solely because an aggregate evaluation score is high.

Describe the specific scenarios, gates, and evidence supporting the claim.

---

## Guiding principle

Good AI evaluation does not ask only:

> How good is the model?

It asks:

> Under which conditions does this system succeed, fail, refuse, hallucinate,
> leak, overreach, or become too expensive?

Measure the behavior that matters.

Keep hard security boundaries deterministic.

Use probabilistic evaluation for probabilistic behavior.

Preserve failures as regression cases.

And never let a flattering average hide a catastrophic edge case.
