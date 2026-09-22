# platform-control-plane.md

> Recipe for scaffolding or elevating a software delivery control plane.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> This recipe defines the architectural and operational expectations for a
> secure, reproducible, policy-driven platform control plane. It does not
> require a specific source-control system, CI engine, registry, GitOps
> product, cloud provider, or orchestration platform.

## Purpose

Use this recipe when the repository owns or defines a platform that turns
source changes into verified, immutable artifacts and then promotes those
artifacts into runtime environments.

Typical examples:

- internal developer platforms;
- CI/CD control planes;
- build farms;
- artifact promotion systems;
- GitOps delivery platforms;
- release orchestration systems;
- supply-chain security platforms;
- deployment/promotion pipelines;
- platform engineering monorepos.

Do **not** apply this recipe to a single application repository merely because
it contains a CI workflow.

The goal is not to recreate a fashionable platform diagram.

The goal is to establish a control plane with clear authority boundaries,
reproducible execution, immutable artifacts, explicit promotion, auditable
policy, and end-to-end verification.

---

## Core architectural principle

Separate **build** from **deploy**.

The build plane answers:

- What source revision was built?
- With what dependencies and toolchain?
- What tests/checks passed?
- What immutable artifact was produced?
- What provenance can be attached to it?
- Is the artifact signed and verifiable?

The deploy plane answers:

- What artifact is desired in this environment?
- Who or what is allowed to promote it?
- What policy must be satisfied before rollout?
- How is rollout observed?
- How is failure stopped or rolled back?
- What evidence proves what happened?

The immutable artifact identifier is the contract between the two planes.

A reference flow may look like:

```text
Developer
   |
   v
Source Control
   |
   v
Build Trigger
   |
   v
Build / Test / Package
   |
   v
Artifact Registry
   |
   v
Sign / Attest / Verify
   |
   v
Desired-State Change
   |
   v
Deployment Reconciler
   |
   v
Progressive Delivery
   |
   v
Runtime Telemetry / Policy
```

Do not collapse these responsibilities merely for convenience.

---

## Architectural invariants

A platform control plane SHOULD satisfy these invariants unless a documented
constraint justifies otherwise.

### 1. Source is identified immutably

Builds must be tied to an immutable source revision.

Do not rely on branch names alone as build identity.

The build record should be able to answer exactly which commit/revision was
used.

### 2. Build inputs are explicit

Toolchains, dependencies, base images, reusable CI tasks, and build
configuration should be pinned sufficiently for reproducibility.

Avoid hidden host dependencies.

Prefer hermetic or strongly isolated build environments where practical.

### 3. Build and deployment authority are separate

The build system should not receive broad production deployment credentials
unless the architecture explicitly requires direct deployment.

Prefer:

```text
build system
    ->
artifact + evidence
    ->
desired-state update / promotion request
    ->
deployment system
```

over:

```text
build system
    ->
kubectl / ssh / cloud-admin
```

### 4. Artifacts are immutable

Promotion must identify an immutable artifact.

For container images, prefer a digest form such as:

```text
registry.example/service@sha256:...
```

over mutable labels such as:

```text
latest
main
production
```

Tags may exist for human convenience, but should not be the sole promotion
identity.

### 5. Build once, promote many

Do not rebuild the application separately for staging, pre-production, and
production.

Promote the same artifact through environments with environment-specific
configuration applied separately.

### 6. Signing requires verification

Artifact signing is incomplete unless there is a defined verification point.

If the platform generates signatures, attestations, or provenance, specify:

- who creates them;
- what identity is trusted;
- where they are stored;
- where they are verified;
- what happens when verification fails.

### 7. Desired state is auditable

Deployment intent should be represented in an auditable system.

For GitOps systems, Git is the desired-state record.

For non-GitOps systems, use an equivalent durable, reviewable promotion record.

Do not rely on undocumented imperative deployment state.

### 8. Promotion is explicit

Moving an artifact between environments is a distinct action with explicit
policy.

Promotion should not silently rebuild or substitute another artifact.

### 9. Policy is enforceable

Security, quality, approval, and environment requirements should be enforced
by executable controls where practical.

Documentation alone is not a policy engine.

### 10. Rollout is observable

Deployment progression must have sufficient telemetry to determine whether it
is safe to continue.

A progressive rollout without meaningful health signals is only delayed
deployment.

### 11. Rollback/recovery is designed

The platform should define what happens when:

- build fails;
- artifact publication fails;
- signature/provenance verification fails;
- desired-state update fails;
- rollout health degrades;
- deployment controller is unavailable;
- registry is unavailable;
- source-control integration fails.

### 12. The control plane itself is declarative

Platform configuration should be version controlled.

Bootstrap and upgrades should be reproducible.

Manual console configuration should be minimized and documented when
unavoidable.

---

## Control-plane boundaries

Before implementation, identify the following planes explicitly.

### Source plane

Responsible for:

- source repositories;
- review;
- branch protection;
- webhook/event generation;
- immutable revision identity.

### Build plane

Responsible for:

- source checkout;
- dependency resolution;
- tests;
- static checks;
- compilation;
- packaging;
- artifact creation;
- build evidence.

### Artifact plane

Responsible for:

- artifact storage;
- immutable addressing;
- retention;
- integrity;
- metadata;
- signatures/attestations.

### Promotion plane

Responsible for:

- environment progression;
- release approvals;
- desired-state changes;
- policy checks;
- rollout initiation.

### Runtime reconciliation plane

Responsible for:

- reconciling desired state;
- rollout execution;
- health observation;
- rollback/abort.

### Observability plane

Responsible for:

- deployment metrics;
- logs;
- traces;
- rollout health;
- control-plane diagnostics.

### Policy plane

Responsible for:

- signature verification;
- provenance requirements;
- admission policy;
- approval rules;
- environment constraints.

These may be implemented by fewer physical systems, but the logical
responsibilities should remain clear.

---

## Source-control integration

The platform must detect and integrate with the actual source-control system.

Do not assume GitHub.

Possible systems include:

- Gitea;
- GitHub;
- GitLab;
- Bitbucket;
- Gerrit;
- self-hosted Git;
- another repository platform.

Where event-driven CI is used:

- verify webhook/event authenticity;
- bind events to an exact repository and revision;
- reject malformed events;
- prevent unauthorized repositories from triggering privileged pipelines;
- avoid build loops caused by automation commits;
- make trigger filtering explicit.

Do not let user-controlled repository metadata become trusted pipeline
configuration without validation.

---

## Build trigger model

Define exactly what causes a build.

Examples:

- pull request;
- push to protected branch;
- release tag;
- manual release request;
- scheduled rebuild;
- dependency update.

Different triggers may have different privilege levels.

A pull-request build should not automatically receive the same secrets or
publication authority as a protected release build.

Separate untrusted contribution builds from trusted release builds where the
repository threat model requires it.

---

## Build execution

Build jobs should be isolated from one another.

Prefer ephemeral execution environments.

As applicable:

- use clean workspaces;
- avoid persistent mutable build state;
- scope credentials per job;
- restrict network access where feasible;
- limit CPU/memory;
- clean temporary data;
- prevent cross-build artifact contamination.

Do not assume the build worker is trustworthy merely because it is internal.

---

## Hermeticity and reproducibility

Prefer builds whose inputs can be identified and reconstructed.

Depending on ecosystem and maturity, this may include:

- dependency lockfiles;
- pinned toolchains;
- pinned base images;
- reproducible package repositories;
- Nix/Guix/Bazel-style dependency closure;
- vendoring where justified;
- cached artifacts addressed by content.

Hermeticity is a build property.

Do not describe the entire CI/CD control plane as "hermetic" unless that claim
is technically accurate.

The platform itself should instead be reproducible, declarative, and pinned.

---

## Build outputs

Every successful build should produce an identifiable result.

Depending on the workload:

- container image;
- package;
- binary;
- archive;
- library;
- Helm chart;
- OCI artifact;
- firmware image;
- model artifact.

The platform should capture:

- source revision;
- build identifier;
- artifact digest/checksum;
- build timestamp;
- toolchain identity where relevant;
- test/check result;
- provenance reference where used.

Do not treat CI logs as the only record of build identity.

---

## Artifact registry

Use the artifact store appropriate to the artifact type.

The registry/store should support, where applicable:

- immutable or content-addressed references;
- retention policy;
- access control;
- auditability;
- metadata;
- signature/attestation storage;
- vulnerability metadata;
- replication/backup where required.

Avoid using source-control attachments or arbitrary object-storage paths as a
substitute for an artifact registry when stronger artifact semantics are
required.

---

## Supply-chain identity

For releasable artifacts, consider attaching:

- checksum/digest;
- SBOM;
- provenance;
- signature;
- vulnerability scan result;
- policy decision.

Do not generate every possible attestation by default.

Use the applied profiles and risk model to determine required evidence.

Where signing is used, prefer short-lived or workload-backed identities over
long-lived private keys where the environment supports them.

If local demonstration requires a key pair, generate it during bootstrap and
treat it as non-production material.

---

## Provenance

Where provenance is required, it should answer at minimum:

- what source revision was built;
- what builder produced the artifact;
- what artifact digest resulted;
- what invocation/build definition was used.

Do not claim high-assurance provenance if the build environment allows
untracked mutable inputs.

---

## SBOM

Where an SBOM is required:

- generate it from the actual artifact or resolved dependency graph;
- use a standard format;
- retain it alongside release evidence;
- ensure it corresponds to the promoted artifact.

An SBOM that cannot be tied to a specific immutable artifact is weaker
evidence.

---

## Vulnerability scanning

Apply scanning at the appropriate stages.

Potential stages include:

- source/dependency scanning before build;
- artifact scanning after build;
- registry rescanning as vulnerability databases evolve;
- deployment policy evaluation.

Scanning must have a defined failure policy.

Do not create a permanently ignored red dashboard.

For production policy, define:

- severity threshold;
- fix availability handling;
- exception process;
- expiration of exceptions;
- ownership of remediation.

Do not encode arbitrary severity policy without project or organizational
requirements.

---

## Promotion model

Promotion should be represented explicitly.

A promotion record should identify:

- artifact;
- source revision;
- source environment;
- target environment;
- actor/automation identity;
- policy result;
- approval where required;
- timestamp;
- resulting deployment state.

For GitOps, this may be represented by a pull request or commit that changes
an artifact digest in environment configuration.

For another delivery model, preserve equivalent auditability.

---

## GitOps

If GitOps is used:

- deployment desired state belongs in Git;
- the deployment reconciler observes Git;
- the build pipeline should not secretly deploy around Git;
- environment changes should be reviewable;
- controller credentials should be scoped;
- drift behavior should be explicit;
- prune/self-heal behavior should be chosen deliberately.

Avoid CI calling the GitOps controller API solely to imitate a direct deploy.

Prefer changing desired state and allowing reconciliation to occur.

Prevent automation commits from re-triggering unintended build loops.

---

## Progressive delivery

Use progressive delivery where deployment risk justifies it.

Possible strategies:

- rolling update;
- canary;
- blue/green;
- staged environment promotion;
- ring-based rollout.

Do not add canary infrastructure merely because the platform supports it.

If canary weighting is claimed, distinguish:

- replica-based approximation;
- actual request-level traffic routing.

Do not describe replica ratios as precise traffic percentages.

Progression decisions should use meaningful signals.

Examples:

- success/error rate;
- latency;
- saturation;
- domain-specific business metrics;
- crash/restart rate.

---

## Automated analysis

Automated rollout analysis must define:

- metric source;
- query;
- evaluation interval;
- sample count;
- success condition;
- failure condition;
- inconclusive behavior;
- abort behavior.

Do not create tautological health checks that can never fail.

Where possible, test the negative path by deliberately producing a failing
analysis result in a safe environment.

---

## Manual approvals

Manual approval is appropriate when:

- risk warrants human judgment;
- regulatory process requires it;
- automated evidence is insufficient;
- production promotion is intentionally gated.

Approval should be tied to a specific immutable artifact.

Do not ask a human to approve "whatever is currently on main".

---

## Deployment identity and authorization

Use distinct identities for distinct control-plane functions.

Examples:

- source webhook identity;
- build worker identity;
- artifact publisher identity;
- signer identity;
- GitOps writer identity;
- deployment reconciler identity;
- admission/policy identity.

Do not share one platform-admin token across the entire delivery path.

Use least privilege.

Build workers should not automatically receive production-cluster admin
rights.

---

## Secrets management

Control-plane secrets require stronger discipline because compromise can
affect many workloads.

Requirements:

- do not commit secrets;
- scope secrets by function;
- prefer short-lived credentials;
- rotate credentials;
- avoid injecting secrets into untrusted PR builds;
- prevent secret values from entering logs;
- avoid shared credentials across environments;
- audit access where the platform supports it.

Local bootstrap secrets may be generated automatically and stored in the
local runtime's secret mechanism.

---

## Environment separation

Define environments intentionally.

Examples:

```text
development
staging
production
```

or:

```text
dev
integration
preprod
prod
```

The names matter less than the policy differences.

For each environment define:

- artifact source;
- promotion requirements;
- identity/credential boundary;
- approval model;
- configuration source;
- rollback behavior;
- data sensitivity;
- observability expectations.

Do not create multiple environments that behave identically merely for
appearance.

---

## Configuration promotion

Separate application artifact promotion from environment configuration.

A release should not require rebuilding the application because one
environment uses a different endpoint or feature configuration.

Treat configuration as versioned desired state where practical.

Secrets should not be embedded in the same plain-text configuration.

---

## Database/schema changes

If workloads use persistent schemas, the delivery platform must account for
migration compatibility.

Prefer:

- backward-compatible migrations;
- expand/migrate/contract patterns;
- explicit migration jobs;
- observability of migration state.

Do not assume application rollback is safe after an irreversible database
migration.

Promotion policy should account for data compatibility where required.

---

## Rollback

Rollback must mean something precise.

Possible rollback mechanisms include:

- revert desired-state commit;
- promote previous immutable artifact;
- abort canary;
- switch blue/green service;
- restore compatible configuration.

A rollback must not silently rebuild old source.

Document cases where rollback is unsafe because of data or external side
effects.

---

## Disaster recovery

For control planes whose loss would materially block engineering or
production operations, identify:

- source backup;
- artifact registry backup/replication;
- configuration backup;
- signing-key recovery model;
- GitOps state recovery;
- cluster/control-plane restoration;
- recovery ordering.

Do not over-engineer HA/DR for a local reference environment.

For production platforms, define recovery objectives based on actual
requirements rather than invented RPO/RTO numbers.

---

## Observability

The control plane itself must be observable.

Track, where appropriate:

- build duration;
- build success/failure;
- queue time;
- worker saturation;
- artifact publication failures;
- signature/provenance failures;
- promotion frequency;
- deployment success/failure;
- rollback/abort events;
- policy denials;
- reconciliation lag;
- controller health.

Use correlation identifiers that allow tracing:

```text
source revision
    ->
build
    ->
artifact
    ->
promotion
    ->
deployment
```

Avoid logging credentials or sensitive build inputs.

---

## Auditability

Important control-plane actions should leave an attributable record.

Examples:

- build invocation;
- artifact publication;
- signature;
- exception approval;
- promotion;
- policy override;
- production deployment;
- rollback.

Do not rely solely on ephemeral terminal output.

Where overrides exist, record:

- who;
- what;
- why;
- artifact;
- environment;
- expiration where applicable.

---

## Policy exceptions

No production policy is complete without an exception mechanism.

Exceptions should be:

- explicit;
- narrow;
- attributable;
- time-bounded where practical;
- reviewable;
- observable.

Avoid permanent global bypass switches.

Do not make emergency access impossible, but make it visible.

---

## Bootstrap

A platform repository should provide a reproducible bootstrap path.

For local/reference environments, prefer an obvious command such as:

```text
make bootstrap
```

or an ecosystem-native equivalent.

Bootstrap should be idempotent where practical.

It should create only the infrastructure required by the selected architecture.

Do not rely on undocumented manual UI configuration.

If manual configuration is unavoidable, document it as a gap.

---

## Local reference environment

A local control-plane implementation may use a lightweight local runtime such
as:

- containers;
- kind;
- k3d;
- minikube;
- another disposable environment.

Use the simplest platform capable of demonstrating the architecture.

Do not claim local topology is production architecture.

The purpose of the local environment is to exercise integration and control
flow.

---

## CI/CD implementation

Detect and use the actual platform selected by the repository.

Possible build orchestration systems include:

- Tekton;
- GitHub Actions;
- GitLab CI;
- Jenkins;
- Buildkite;
- Argo Workflows;
- another system.

The recipe does not mandate one.

Regardless of implementation, the pipeline should make phases explicit.

A common release path is:

```text
receive immutable source revision
    ->
checkout
    ->
validate
    ->
test
    ->
build
    ->
package
    ->
publish
    ->
resolve immutable artifact identity
    ->
generate evidence
    ->
sign
    ->
verify
    ->
request/update promotion
```

Do not reorder signing around mutable artifact identity.

---

## Dependency caching

Caching may improve build performance, but must not undermine reproducibility.

Caches should be treated as accelerators, not trusted sources of truth.

Prefer content-addressed caches where supported.

A poisoned cache must not be able to silently substitute unverifiable release
content.

---

## Untrusted contribution builds

If external or lower-trust contributors can trigger builds:

- do not expose release credentials;
- do not expose production secrets;
- separate trusted and untrusted worker pools where justified;
- avoid privileged container execution;
- restrict artifact publication;
- require trusted re-execution before release if needed.

Do not assume a pull request's build scripts are benign.

The repository itself may be attacker-controlled input during CI.

---

## Privileged build features

Avoid privileged build execution where possible.

Where container image builds require special capabilities, prefer safer
mechanisms such as rootless or daemonless builders when compatible with the
ecosystem.

If privileged execution is unavoidable, isolate it and document the trust
implications.

---

## Admission and deployment policy

If the runtime supports admission policy, consider enforcing:

- approved registries;
- immutable image references;
- signature verification;
- provenance requirements;
- non-root execution;
- forbidden privilege;
- environment-specific constraints.

Do not enforce policies that the platform cannot satisfy itself.

Policy rollout should be staged to avoid breaking all deployments at once.

---

## Artifact promotion policy

Production promotion may require evidence such as:

- tests passed;
- artifact vulnerability policy passed;
- signature valid;
- provenance valid;
- source branch/revision allowed;
- approval obtained;
- staging verification passed.

Keep policy rules machine-readable where practical.

Avoid checks that exist only in a wiki.

---

## Version pinning

Pin external platform components sufficiently to support reproducibility and
safe upgrades.

Examples:

- Helm chart versions;
- container image versions/digests;
- CI actions/tasks;
- operators/controllers;
- package repositories;
- toolchains.

Do not use `latest` for control-plane components in reproducible
environments.

Track upgrades deliberately.

---

## Upgrades

The control plane needs an upgrade model.

For major components:

- document current version;
- document upgrade mechanism;
- test upgrades in a lower-risk environment;
- account for CRD/schema changes;
- account for rollback limitations;
- preserve state backups where needed.

Do not silently auto-upgrade critical control-plane components across major
versions.

---

## Platform API and self-service

If developers consume the platform through a self-service interface, expose a
small stable contract.

Possible interfaces:

- repository conventions;
- pipeline templates;
- reusable tasks;
- service catalog actions;
- CLI;
- API;
- declarative config.

Avoid forcing application teams to understand internal platform mechanics.

Prefer paved roads over mandatory bespoke tickets.

Do not create a portal solely because "internal developer platforms have
portals".

---

## Templates and reusable components

Reusable pipeline tasks/templates should:

- have explicit inputs/outputs;
- use immutable versions;
- avoid hidden secrets;
- document required permissions;
- fail clearly;
- support independent testing;
- avoid unnecessary coupling to one application.

Do not centralize every line of pipeline logic if it makes application
ownership opaque.

---

## Repository layout

Follow project conventions first.

A generic platform-control-plane repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── platform/
│   ├── source/
│   ├── build/
│   ├── registry/
│   ├── promotion/
│   ├── deployment/
│   ├── policy/
│   └── observability/
├── bootstrap/
├── scripts/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/
│   ├── architecture/
│   ├── adr/
│   └── runbooks/
└── <platform-specific configuration>
```

This is illustrative, not mandatory.

Do not create empty directories.

Organize by control-plane responsibility, not by vendor name, when that
improves long-term clarity.

---

## Documentation

The platform documentation should explain:

- system purpose;
- supported workload types;
- source-to-production flow;
- trust boundaries;
- build/deploy separation;
- artifact identity model;
- promotion model;
- credential model;
- signing/provenance model;
- rollout strategy;
- policy enforcement;
- bootstrap;
- upgrade process;
- disaster recovery assumptions;
- common failure modes.

A new engineer should be able to answer:

> What happens after I push a change?

and:

> What exact artifact is running in production, and why was it allowed there?

---

## Runbooks

For production control planes, create runbooks for high-impact failure modes.

Examples:

- registry unavailable;
- build workers unavailable;
- GitOps controller stuck;
- signing identity unavailable;
- admission policy blocking valid releases;
- broken promotion;
- failed rollout;
- source webhook outage;
- corrupted desired-state configuration.

Runbooks should prioritize diagnosis and safe recovery.

Do not write speculative runbooks for components that are not present.

---

## Testing strategy

A control plane requires more than YAML/schema validation.

### Unit/component tests

Test custom logic such as:

- policy;
- promotion rules;
- manifest generation;
- configuration parsing;
- helper tools.

### Configuration validation

Validate:

- syntax;
- schemas;
- rendered manifests;
- policy configuration;
- references;
- permissions where tooling permits.

### Integration tests

Exercise important integration seams:

- source event -> build trigger;
- build -> registry;
- build -> signature/provenance;
- promotion -> desired state;
- desired state -> reconciler;
- rollout -> telemetry;
- policy -> allow/deny result.

### End-to-end tests

Where practical, prove:

```text
source change
    ->
automated build
    ->
immutable artifact
    ->
evidence/signature
    ->
promotion
    ->
reconciliation
    ->
healthy rollout
```

### Negative-path tests

At least one important failure path should be exercised.

Examples:

- unit test failure prevents publication;
- unsigned artifact is rejected;
- invalid provenance is rejected;
- untrusted branch lacks release credentials;
- rollout metric failure causes abort;
- unauthorized promotion is denied.

A platform that demonstrates only the happy path is incomplete.

---

## Acceptance test

The recipe is not complete because control-plane files exist.

Where the execution environment permits, demonstrate:

1. bootstrap the platform;
2. register or create a sample workload;
3. push or submit a source change;
4. observe automatic build triggering;
5. observe tests/checks execute;
6. produce an immutable artifact;
7. publish the artifact;
8. capture its digest/checksum;
9. generate required supply-chain evidence;
10. verify that evidence;
11. create/request a promotion;
12. observe desired state change;
13. observe deployment reconciliation;
14. observe rollout health evaluation;
15. observe successful promotion.

Also demonstrate at least one negative path.

Do not claim end-to-end operation if only manifests were rendered.

---

## Verification interface

A platform repository should expose obvious commands or equivalent native
interfaces for:

```text
bootstrap
check
test
test-integration
test-e2e
verify
demo
destroy
```

These names are illustrative.

The important property is that the platform has repeatable verification and
cleanup.

`verify` should validate static configuration and integration prerequisites.

`demo` should exercise the main control flow with minimal manual intervention.

`destroy` should clean disposable local infrastructure safely.

---

## Destructive operations

Destructive operations require explicit scoping.

A local `destroy` command must not be capable of deleting arbitrary clusters,
registries, repositories, or production resources because of an unset
variable or ambiguous context.

Use:

- explicit environment names;
- confirmation for high-impact operations where appropriate;
- namespace/project scoping;
- safe defaults;
- dry-run capability where practical.

Never make production destruction the default behavior.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. control-plane architecture;
2. source-control integration;
3. build system;
4. artifact store;
5. immutable artifact identity;
6. supply-chain evidence generated;
7. verification/enforcement point;
8. promotion mechanism;
9. deployment reconciler;
10. rollout model;
11. observability signals;
12. credential/identity boundaries;
13. tests executed;
14. end-to-end path actually demonstrated;
15. negative path demonstrated;
16. commands actually run;
17. observed results;
18. unverified assumptions;
19. remaining risks;
20. deliberate omissions and why.

Never claim an integration, security control, deployment, signature,
provenance check, rollout, or recovery mechanism works unless it was actually
executed and observed or clearly marked as unverified.

---

## Optional profile hooks

This recipe is intended to compose with additional profiles.

Common combinations:

```text
platform-control-plane + software-supply-chain
```

For deeper provenance, SBOM, signing, verification, artifact policy, and
promotion evidence.

```text
platform-control-plane + zero-trust
```

For stronger workload identity, control-plane trust segmentation, least
privilege, policy enforcement, and service-to-service identity.

```text
platform-control-plane + high-assurance
```

For stricter release evidence, independent verification, fault injection,
recovery testing, and stronger change controls.

```text
platform-control-plane + hostile-input
```

For platforms that execute untrusted repositories, plugins, build scripts, or
externally supplied configuration.

```text
platform-control-plane + ai-security
```

For platforms that build or deploy models, agents, prompts, tool manifests,
evaluation bundles, or other AI system artifacts.

---

## Anti-patterns

Do not introduce these without a concrete requirement:

- Kubernetes solely because this is a platform project;
- GitOps solely because it is fashionable;
- a service mesh for control-plane traffic;
- multiple CI engines doing the same job;
- multiple artifact registries without a boundary reason;
- multiple policy engines with overlapping responsibility;
- mutable production artifact references;
- direct production-admin credentials in CI;
- long-lived shared signing keys;
- release logic embedded only in shell scripts;
- manual UI-only bootstrap;
- undocumented policy overrides;
- scanners whose failures are always ignored;
- signatures that are never verified;
- SBOMs that cannot be tied to an artifact;
- provenance claims with untracked mutable build inputs;
- deployment workflows that rebuild artifacts;
- "canary" deployments without meaningful health analysis;
- fake HA/DR claims based on a local demo environment;
- dashboards as a substitute for enforceable policy.

Do not mistake quantity of platform components for maturity.

---

## Guiding principle

A delivery platform should make the safe path the easy path.

Source should become an artifact through a reproducible process.

Artifacts should be immutable.

Evidence should travel with the artifact.

Promotion should be explicit.

Deployment authority should be narrower than build authority.

Policy should be enforceable.

Rollouts should be observable.

Failures should stop safely.

Overrides should be attributable.

And at any point, the platform should be able to answer:

> What exact source became this exact artifact, who or what allowed it to
> reach this environment, and what evidence justified that decision?
