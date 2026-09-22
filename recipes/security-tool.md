# security-tool.md

> Recipe for scaffolding or elevating a security-focused developer or platform tool.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with profiles such as `software-supply-chain.md`,
> `zero-trust-service.md`, `hostile-input.md`, or `high-assurance.md`
> where appropriate.
>
> This recipe is intentionally implementation-neutral. A security tool may be
> a CLI, service, library, scanner, policy engine, analyzer, proxy, plugin, or
> automation component.

## Purpose

Use this recipe when the repository exists primarily to inspect, validate,
enforce, transform, or report on security-relevant state.

Typical examples:

- static analyzers;
- dependency scanners;
- secret scanners;
- IaC scanners;
- policy validators;
- admission/policy tools;
- configuration auditors;
- repository scanners;
- artifact verifiers;
- SBOM/provenance tools;
- security-focused CLIs;
- security automation services;
- protocol analyzers;
- fuzzing harnesses;
- compliance evidence collectors;
- authorization/policy test tools.

The goal is not merely to produce "security software".

The goal is to produce a tool whose:

- findings are trustworthy;
- behavior is deterministic enough to automate;
- failure modes are explicit;
- output can be consumed by humans and machines;
- hostile input is handled safely;
- security claims are testable;
- false positives and false negatives are treated seriously;
- exit semantics are unambiguous;
- evidence can be traced back to the analyzed input.

---

## Core operating principle

A security tool sits in a dangerous position:

```text
untrusted or security-relevant input
            |
            v
      security logic
            |
            v
   finding / decision / action
```

That means the tool itself must be treated as security-sensitive.

Do not assume the input is benign.

Do not assume the environment is trustworthy.

Do not assume "no findings" means "safe".

Do not turn parser crashes, scanner failures, incomplete analysis, or missing
coverage into silent success.

---

## Architectural invariants

A security tool SHOULD satisfy these invariants unless the repository has a
documented reason not to.

### 1. Findings are evidence, not truth

A finding should be attributable to:

- the exact input;
- the exact rule/check;
- the exact tool version;
- relevant configuration;
- relevant source location or artifact identity.

Avoid vague output such as:

```text
Security issue found
```

Prefer output that explains what was detected and where.

### 2. Scanner failure is not clean scan

Distinguish:

```text
analysis succeeded, zero findings
```

from:

```text
analysis did not complete
```

A failed parser, timeout, missing dependency, unsupported format, or internal
error must not silently become "pass".

### 3. Exit codes are part of the API

Exit behavior must be stable and documented.

At minimum distinguish where appropriate:

- success/no blocking findings;
- findings detected;
- usage/configuration error;
- analysis/internal failure.

Do not use one generic exit code for every failure mode if downstream
automation needs to distinguish them.

### 4. Machine-readable output is first-class

Where the tool is intended for CI or automation, provide structured output.

Prefer a stable machine-readable format such as:

- JSON;
- SARIF;
- NDJSON;
- another documented schema.

Human-readable terminal output is useful, but should not be the only
integration interface.

### 5. Determinism matters

Given the same input, version, and configuration, the tool should produce
equivalent results unless nondeterminism is inherent and documented.

Avoid hidden dependence on:

- current directory state;
- local caches;
- wall-clock time;
- mutable remote data;
- network availability;
- environment-specific defaults.

### 6. Rules are explicit

Security rules should be reviewable and testable.

Where rules are configurable, define:

- rule identifier;
- severity;
- description;
- rationale;
- detection logic;
- remediation guidance where appropriate.

Do not encode critical policy only in opaque code paths if a clearer
representation is practical.

### 7. Unknown is not safe

When the tool cannot confidently evaluate something, represent that state
explicitly.

Possible states include:

```text
PASS
FAIL
UNKNOWN
ERROR
NOT_APPLICABLE
```

Do not coerce `UNKNOWN` or `ERROR` into `PASS`.

### 8. Safe defaults

Default configuration should favor safe analysis.

Avoid defaults that:

- ignore errors;
- disable important checks;
- follow arbitrary symlinks;
- execute analyzed content;
- reach arbitrary network locations;
- scan sensitive filesystem areas unintentionally.

### 9. Analysis should be side-effect free by default

A scanner/analyzer should normally inspect, not modify.

If the tool supports remediation/fix mode:

- make it explicit;
- preview changes where practical;
- preserve backups or reversibility where appropriate;
- never silently mutate input during analysis mode.

### 10. Security claims must be bounded

Do not claim:

```text
secure
safe
compliant
vulnerability-free
```

when the tool only evaluated a subset of conditions.

State exactly what was checked.

---

## Tool archetype

Before implementation, classify the tool.

Possible archetypes include:

- CLI scanner;
- library;
- CI plugin/action;
- daemon/service;
- policy engine;
- admission controller;
- repository bot;
- protocol proxy;
- IDE/editor integration;
- artifact verifier;
- fuzzing tool;
- evidence generator.

A repository may combine several archetypes.

Identify:

- primary consumer;
- input format;
- output format;
- execution environment;
- trust boundary;
- side effects;
- expected automation integration;
- performance constraints.

Do not design a distributed service when a local CLI would solve the problem.

---

## Input threat model

Assume analyzed input may be malicious.

Examples:

- source repositories;
- archives;
- manifests;
- binaries;
- documents;
- container images;
- package metadata;
- YAML/JSON;
- Git history;
- SBOMs;
- URLs;
- certificates;
- logs;
- policy files.

Potential attacks include:

- parser crashes;
- algorithmic complexity attacks;
- decompression bombs;
- archive traversal;
- symlink abuse;
- path traversal;
- oversized input;
- malformed encodings;
- recursive structures;
- embedded secrets;
- hostile filenames;
- code execution through parsers/plugins;
- malicious remote references.

Treat parsing as a security boundary.

---

## Parsing and hostile input

Use safe parsers.

Avoid unsafe deserialization.

Set limits where practical for:

- input size;
- nesting depth;
- archive size;
- decompression ratio;
- file count;
- recursion depth;
- token count;
- regex complexity;
- processing time.

For archives:

- reject path traversal;
- normalize paths;
- avoid extracting outside the intended directory;
- handle symlinks explicitly;
- bound expansion.

For binary parsing:

- validate offsets and lengths;
- reject malformed structures;
- fuzz parsers where feasible.

---

## Filesystem handling

If scanning files or repositories:

- do not follow symlinks blindly;
- define whether hidden files are included;
- define ignore semantics;
- normalize paths;
- avoid escaping the requested scan root;
- avoid reading device files or special files unintentionally;
- handle permission errors explicitly.

Do not assume filenames are valid UTF-8 or safe to print.

Escape output where required.

---

## Remote inputs

If the tool can fetch URLs, repositories, artifacts, or metadata:

- validate schemes;
- apply timeouts;
- bound response size;
- handle redirects deliberately;
- consider SSRF;
- protect metadata endpoints;
- verify TLS;
- validate expected hostnames where appropriate.

Do not allow arbitrary remote fetching merely because a field contains a URL.

Offline mode should exist where useful.

---

## Rules and checks

Each rule should have a stable identifier.

Example:

```text
SEC001
```

A rule should document:

- ID;
- title;
- severity;
- category;
- description;
- evidence;
- remediation guidance where appropriate.

Stable rule IDs enable:

- suppression;
- baselining;
- dashboards;
- CI policies;
- regression tests.

Do not use mutable prose strings as the only finding identity.

---

## Severity

Severity must have defined semantics.

Do not assign severity arbitrarily.

Possible models include:

- informational;
- low;
- medium;
- high;
- critical.

Or another project-specific scale.

Where CVSS or another external scoring system is used, distinguish:

- tool-assigned severity;
- upstream severity;
- environmental severity.

Do not pretend severity equals exploitability.

---

## Confidence

Where detection is heuristic, consider separate confidence from severity.

For example:

```text
severity: high
confidence: low
```

This is often more informative than collapsing both concepts.

Do not hide uncertainty.

---

## False positives

False positives are product defects.

Provide enough evidence for a user to determine whether a finding is valid.

Where suppression is supported:

- make it explicit;
- scope narrowly;
- require a reason where practical;
- preserve rule ID;
- avoid broad global ignores.

Do not force users to disable entire rule families for one known exception.

---

## False negatives

False negatives are often harder to observe.

Use:

- regression tests;
- known-vulnerable fixtures;
- adversarial corpora;
- mutation testing where useful;
- independent test cases;
- fuzzing;
- real-world samples.

Do not evaluate scanner quality only on clean inputs.

---

## Baselines

Where a repository has existing findings, baseline mode may help adoption.

A baseline should:

- identify existing findings;
- fail only on newly introduced findings;
- remain reviewable;
- support gradual cleanup.

Do not let baselines become permanent universal suppression files.

Baseline identity should be tied to stable finding attributes where possible.

---

## Suppression model

Suppression must be deliberate.

Possible forms:

- inline comment;
- config file;
- external policy;
- baseline.

Each suppression should preferably capture:

- rule ID;
- scope;
- justification;
- expiration where useful.

Do not support hidden magic strings that bypass controls without auditability.

---

## Output model

A finding should include, where applicable:

- rule ID;
- title;
- severity;
- confidence;
- location;
- evidence;
- message;
- remediation;
- fingerprint;
- references;
- metadata.

Use stable fingerprints if findings must be tracked across runs.

Do not include secrets in finding evidence.

Redact sensitive values while retaining enough context to diagnose the issue.

---

## SARIF

If the tool targets source-code security workflows, SARIF may be useful.

If SARIF is supported:

- validate output;
- use stable rule IDs;
- provide locations;
- provide meaningful messages;
- avoid malformed or incomplete results.

Do not add SARIF merely as a checkbox if downstream systems do not need it.

---

## JSON schema

For JSON output, publish or document a stable schema.

Version the schema if compatibility matters.

Avoid breaking machine consumers casually.

If fields are deprecated, preserve compatibility long enough for realistic
migration.

---

## CLI behavior

For CLI tools:

- use predictable subcommands;
- provide `--help`;
- provide `--version`;
- support non-interactive use;
- write findings to stdout or an explicit output file;
- write diagnostic errors to stderr;
- define exit codes;
- avoid prompts in CI mode.

Prefer explicit flags over environment-dependent magic.

---

## Configuration

Configuration should be explicit and validated.

Define precedence, for example:

```text
defaults
    <
config file
    <
environment
    <
CLI flags
```

if that matches the tool.

Do not silently accept unknown security-sensitive config keys.

Report invalid rule names and invalid suppressions.

---

## Offline and online modes

If the tool uses remote intelligence, vulnerability feeds, or APIs, separate:

- analysis logic;
- data acquisition;
- local cache.

Document when network access affects results.

Where possible, provide an offline/reproducible mode.

Do not imply two scans are comparable if one used fresh remote intelligence
and the other used stale data.

---

## External vulnerability data

If consuming vulnerability databases or threat intelligence:

- identify source;
- track data freshness;
- validate format;
- handle source outage;
- distinguish stale data from no vulnerability.

Do not say "no vulnerabilities" when the vulnerability database failed to
update.

---

## Data freshness

Security data can expire.

Expose relevant metadata such as:

- feed version;
- last update;
- generated time;
- source database revision.

Allow policy to reject excessively stale intelligence where required.

---

## Performance

Security tools often scan large repositories or artifact sets.

Measure performance on representative workloads.

Avoid:

- unbounded concurrency;
- loading entire large artifacts into memory unnecessarily;
- catastrophic regex behavior;
- repeated parsing of identical content.

Performance optimizations must not weaken correctness silently.

---

## Incremental scanning

Incremental/diff scanning can improve developer experience.

But define clearly what it does and does not cover.

Do not label a changed-files-only scan as equivalent to a full repository
scan.

Use full scans at appropriate boundaries where required.

---

## Concurrency

If scanning concurrently:

- bound workers;
- preserve deterministic aggregation;
- avoid races in caches/output;
- handle cancellation;
- maintain stable exit semantics.

Do not make output ordering nondeterministic if machine consumers depend on it.

Sort results before emission where practical.

---

## Caching

Caches must not alter security correctness.

Cache keys should include relevant:

- input hash;
- rule version;
- tool version;
- configuration;
- external data version.

Do not reuse stale cached results after rule or intelligence changes unless
that behavior is intentional and visible.

---

## Plugin systems

Plugins increase extensibility and attack surface.

Only introduce a plugin system when there is a clear need.

If plugins exist:

- define trust model;
- isolate untrusted plugins where practical;
- version plugin API;
- restrict permissions;
- validate plugin metadata;
- avoid automatic execution of arbitrary repository plugins.

Do not load executable plugins from an analyzed repository by default.

---

## Rule extensions

Prefer declarative rule extensions where possible.

If custom rules can execute code, treat them as trusted code.

Do not execute user-supplied rule code inside a privileged scanner process
without isolation.

---

## Fix/remediation mode

If the tool can modify input:

- separate scan and fix commands;
- make changes reviewable;
- avoid unsafe auto-fixes;
- preserve formatting where possible;
- rerun validation after changes.

A fix must not create a new vulnerability while removing another.

Do not auto-fix security-sensitive configuration blindly when intent matters.

---

## Policy mode

A security tool may support policy decisions such as:

```text
allow
deny
warn
```

If so, keep policy logic separate from detection where practical.

Detection answers:

> What did we observe?

Policy answers:

> Is this acceptable here?

This separation improves reuse.

---

## CI integration

CI integration should:

- use deterministic versions;
- expose clear exit codes;
- preserve reports as artifacts;
- avoid leaking secrets;
- fail according to documented policy;
- distinguish tool failure from finding failure.

Do not use `continue-on-error` for mandatory security checks unless there is a
documented rollout mode.

---

## Pre-commit integration

Use pre-commit checks only for fast developer feedback.

Do not force heavyweight full-repository scans on every commit unless runtime
is acceptable.

Prefer:

```text
fast local check
    +
full CI/security scan
```

over making developers bypass an unusable local hook.

---

## Service mode

If the security tool runs as a service:

- apply `secure-service.md`;
- authenticate clients if required;
- authorize sensitive operations;
- validate uploaded/analyzed content;
- isolate scan jobs;
- bound resources;
- avoid cross-tenant data leakage;
- expose health and metrics.

Do not assume uploaded artifacts are safe.

---

## Multi-tenancy

If scanning for multiple tenants:

- isolate inputs;
- isolate findings;
- isolate caches;
- isolate temporary files;
- scope credentials;
- prevent one tenant from inferring another tenant's data.

Do not use shared temp paths with predictable names.

---

## Temporary data

Security analysis may handle sensitive source and artifacts.

Temporary data should:

- live in controlled locations;
- use safe permissions;
- be cleaned up;
- avoid predictable collision-prone names;
- not survive indefinitely after failure.

Consider encrypted storage where risk justifies it.

---

## Secret handling

A scanner may encounter secrets even when it is not a secret scanner.

Do not log full sensitive values.

If a secret must be reported:

- redact;
- fingerprint;
- show only minimal context.

Do not include raw credentials in JSON/SARIF reports by default.

---

## Secret scanning tools

If the tool itself detects secrets:

- minimize retention of matched values;
- prefer fingerprints/hashes;
- avoid sending findings to third parties without explicit design;
- clearly distinguish verified and heuristic detections.

Do not print an entire private key to prove you found it.

---

## Security boundaries

Document which components are trusted.

Examples:

```text
trusted:
    scanner binary
    signed rule bundle

untrusted:
    repository input
    archive input
    remote manifest

conditionally trusted:
    vulnerability feed
    plugin bundle
```

Do not let untrusted content cross into the trusted execution boundary
implicitly.

---

## Sandboxing

For tools that execute analyzed code or dynamic checks, use isolation.

Possible approaches:

- container sandbox;
- VM;
- restricted subprocess;
- seccomp;
- namespace isolation;
- capability dropping;
- network restrictions.

Do not execute repository code directly on a privileged CI host merely because
dynamic analysis requires execution.

---

## Dynamic analysis

If dynamic analysis is required:

- isolate target;
- bound runtime;
- bound network;
- bound filesystem;
- collect logs safely;
- clean environment after execution.

Assume analyzed code may attempt to escape or exfiltrate.

---

## Fuzzing

Security-sensitive parsers and rule engines should consider fuzzing.

Good fuzz targets include:

- parsers;
- decoders;
- archive handlers;
- policy evaluators;
- serialization;
- protocol framing;
- file readers.

Crashes found by fuzzing should become regression tests.

---

## Property-based testing

Use property tests for invariants such as:

- malformed input never causes panic;
- normalization is idempotent;
- serialization round-trips safely;
- deny rules cannot become allow under irrelevant field changes;
- scanner result ordering is deterministic.

---

## Corpus testing

Maintain representative fixtures.

Include:

- clean samples;
- known-vulnerable samples;
- malformed samples;
- edge cases;
- real-world structures;
- regression cases.

Label fixtures clearly.

Avoid embedding real credentials or sensitive customer data.

---

## Golden tests

Golden/snapshot outputs may help validate stable report formats.

Use them carefully.

Review semantic differences rather than blindly updating snapshots after every
change.

Do not allow snapshot regeneration to hide security regression.

---

## Regression rule

Every security defect in the tool should result in a regression test where
practical.

Examples:

- bypass;
- missed finding;
- false positive;
- parser crash;
- unsafe file handling;
- output corruption;
- wrong exit code.

---

## Compatibility

If the tool has machine consumers, preserve compatibility.

Version:

- rule IDs;
- output schema;
- API;
- CLI flags;
- config format.

Breaking changes require explicit migration documentation.

Do not silently repurpose existing rule IDs.

---

## Rule versioning

A rule's semantics may evolve.

If a change materially alters what a rule detects, consider:

- versioning;
- release notes;
- changed fingerprint semantics;
- baseline impact.

Do not make historical findings impossible to interpret.

---

## Documentation

The README should explain:

- what the tool detects or enforces;
- what it explicitly does not detect;
- supported inputs;
- installation;
- quickstart;
- exit codes;
- output formats;
- configuration;
- suppressions;
- CI usage;
- security model;
- known limitations.

Avoid marketing claims not supported by tests.

---

## Rule documentation

For each major rule or rule family, document:

- why it matters;
- examples;
- false-positive considerations;
- remediation;
- severity rationale where useful.

Security findings should teach the user enough to act.

---

## Threat model documentation

Security tools should document their own trust model.

Include:

- hostile inputs;
- trusted components;
- remote dependencies;
- plugin risk;
- dynamic execution risk;
- data retention;
- network requirements.

A security tool without a threat model should not assume its own safety.

---

## Telemetry

If the tool emits telemetry:

- make it explicit;
- avoid sending analyzed source/content by default;
- avoid sending secrets;
- document what is collected;
- allow disabling where appropriate.

Do not surprise users with network calls from a local security scanner.

---

## Privacy

Security tools often inspect sensitive repositories.

Minimize retained data.

Do not upload source, findings, or credentials externally unless explicitly
required and documented.

If remote analysis is part of the architecture, make the boundary obvious.

---

## Supply chain

A security tool's own supply chain matters.

Compose with `software-supply-chain.md` where the tool is distributed.

At minimum consider:

- pinned dependencies;
- reproducible builds;
- signed releases;
- checksums;
- SBOM;
- provenance.

A compromised security tool can become a privileged attack vector.

---

## Least privilege

If the tool needs elevated permissions, justify each one.

Examples:

- filesystem read;
- container socket;
- cluster access;
- registry access;
- cloud API access.

Prefer read-only credentials for scanners.

Do not require admin credentials where narrower permissions work.

---

## Kubernetes/security platform integrations

If the tool integrates with Kubernetes or another platform:

- use least-privilege RBAC;
- prefer read-only access for audit tools;
- scope namespaces/resources;
- avoid cluster-admin;
- handle partial permissions explicitly.

Do not silently skip inaccessible resources and report a clean scan.

Surface incomplete coverage.

---

## Coverage reporting

If the tool may only partially inspect a target, report coverage.

Examples:

```text
scanned: 94 resources
skipped: 3 resources
errors: 2 resources
```

Do not collapse partial analysis into success.

---

## Confidence in negative results

A "no findings" result should be qualified by what was actually checked.

Where useful, include:

- enabled rule count;
- skipped rules;
- unsupported files;
- parse errors;
- excluded paths;
- stale intelligence state.

Negative results need context.

---

## Compliance mode

If the tool maps findings to frameworks such as CIS, NIST, PCI DSS, or other
controls:

- distinguish technical checks from full compliance;
- preserve framework/control references;
- avoid saying a passing scan proves compliance.

A technical scanner can provide evidence toward compliance.

It does not replace organizational, procedural, or legal assessment.

---

## Reporting

Reports should prioritize actionability.

For each issue, include:

- what is wrong;
- where;
- why it matters;
- evidence;
- how to remediate.

Avoid dumping raw scanner internals on users.

Support summary and detailed modes where useful.

---

## Exit-code example

A security CLI may use semantics such as:

```text
0 = scan completed; no blocking findings
1 = scan completed; blocking findings present
2 = invalid invocation or configuration
3 = scan incomplete/internal error
```

This is illustrative.

Choose a scheme appropriate to the ecosystem and document it.

Do not change exit semantics casually after users automate around them.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic security tool may resemble:

```text
.
├── AGENTS.md
├── README.md
├── src/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── regression/
│   ├── corpus/
│   └── fuzz/
├── rules/
├── schemas/
├── docs/
│   ├── rules/
│   ├── architecture/
│   └── adr/
├── scripts/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

A security-tool repository should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-regression
test-integration
fuzz
build
scan-fixtures
verify-output
```

These names are illustrative.

Use the project's native tooling where stronger conventions exist.

---

## Acceptance criteria

A security tool is not complete because it produces findings.

Demonstrate the applicable subset of the following.

### Functional

- clean fixture produces expected clean result;
- vulnerable fixture produces expected finding;
- malformed input does not crash;
- unsupported input is reported explicitly;
- exit codes match documentation.

### Output

- human-readable output is understandable;
- machine-readable output validates;
- stable rule identifiers exist;
- findings include actionable locations/evidence where applicable;
- secrets are redacted.

### Security

- hostile paths cannot escape scan root;
- archive extraction is safe where applicable;
- arbitrary remote fetch is constrained where applicable;
- untrusted plugins/code are not executed implicitly;
- dynamic analysis is isolated where applicable.

### Reliability

- scan failure is distinguishable from clean scan;
- partial coverage is surfaced;
- timeouts/resource exhaustion are handled;
- output is deterministic enough for automation.

### Detection quality

- known-positive fixtures are detected;
- known-negative fixtures remain clean;
- regression corpus passes;
- false-positive suppressions work as documented;
- at least one important bypass/negative-path test passes.

### Automation

- CI can consume exit status;
- structured report is available where required;
- no interactive input is required in CI mode.

### Documentation

- supported inputs documented;
- unsupported/unknown behavior documented;
- exit codes documented;
- suppressions documented;
- known limitations documented.

---

## Optional composition

Common combinations:

```text
security-tool + hostile-input
```

For parsers, file scanners, archive analyzers, protocol inspectors, or tools
that routinely process attacker-controlled content.

```text
security-tool + software-supply-chain
```

For distributed scanners, agents, binaries, plugins, and signed rule bundles.

```text
security-tool + high-assurance
```

For deeper fuzzing, independent verification, adversarial corpora, and stronger
release evidence.

```text
security-tool + zero-trust-service
```

For multi-user or remote scanning services with explicit identity and
authorization boundaries.

```text
security-tool + ai-security
```

For AI-assisted analysis where model output is advisory and must not silently
become an enforcement decision.

---

## Anti-patterns

Avoid:

- "no findings" when scanning failed;
- parser exceptions treated as clean results;
- executing analyzed repositories by default;
- printing secrets in findings;
- unstable rule IDs;
- undocumented exit codes;
- machine output generated by scraping human text;
- severity with no semantics;
- global suppression files with no ownership;
- network calls hidden inside local analysis;
- plugins auto-loaded from the target repository;
- compliance claims based on partial technical checks;
- scanners that require admin credentials for read-only work;
- full-repository mutation during normal scan mode;
- ignoring inaccessible files/resources;
- regex-heavy detection without adversarial tests;
- output schemas changed silently;
- scanner results depending on wall-clock time without disclosure;
- false-positive handling based only on "disable the rule";
- AI-generated findings treated as deterministic proof without verification.

Do not mistake the word "security" in the project name for evidence that the
tool itself is secure.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. tool archetype;
2. supported input types;
3. hostile-input assumptions;
4. rule/check model;
5. output formats;
6. exit-code semantics;
7. configuration model;
8. suppression/baseline model;
9. external data/network dependencies;
10. sandboxing/isolation model where applicable;
11. structured-output validation;
12. corpus/regression coverage;
13. fuzz/property tests performed;
14. positive-path fixture results;
15. negative/bypass-path fixture results;
16. commands actually executed;
17. observed results;
18. known limitations;
19. unresolved false-positive/false-negative risks;
20. controls deliberately not implemented and why.

Never claim the tool "detects", "blocks", "verifies", or "proves" a security
condition unless that behavior was actually implemented and tested.

---

## Guiding principle

A good security tool is conservative about what it knows.

It should distinguish:

```text
clean
```

from:

```text
could not determine
```

It should treat hostile input as hostile.

It should make findings reproducible.

It should make automation predictable.

It should make failures visible.

It should make suppressions explicit.

It should make evidence actionable.

And it should never let its own uncertainty quietly become someone else's
false sense of security.
