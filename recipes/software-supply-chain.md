# software-supply-chain.md

> Security profile for software supply-chain integrity.
>
> Apply under `AGENTS.md`, together with `SCAFFOLD.md` or `ELEVATE.md`, and
> compose with an appropriate recipe such as `secure-service.md`,
> `platform-control-plane.md`, `library-sdk.md`, `cli.md`, or
> `agentic-system.md`.
>
> This profile defines how source, dependencies, builds, artifacts, evidence,
> promotion, and verification should be handled. It is not a mandate to deploy
> every available supply-chain security product.

## Purpose

Use this profile when the repository produces software or artifacts that may be:

- released externally;
- deployed into production;
- consumed by other teams;
- used as a dependency;
- distributed through an artifact registry;
- executed with meaningful privilege;
- installed into customer or production environments;
- used as part of a security-sensitive build or deployment path.

Typical artifacts include:

- container images;
- binaries;
- packages;
- archives;
- libraries;
- CLI releases;
- Helm charts;
- OCI artifacts;
- firmware;
- plugins;
- models;
- agent bundles;
- generated deployment manifests.

The goal is to make it possible to answer:

> What source and dependencies produced this artifact, in what environment,
> under what identity, with what evidence, and why should I trust it?

The goal is **not** to maximize the number of SBOMs, signatures, scanners,
attestations, or badges.

---

## Core invariants

A software supply chain SHOULD satisfy these invariants unless the project has
a documented reason not to.

### 1. Source identity is immutable

Every release artifact must map to an immutable source revision.

Do not use branch names alone as release identity.

Prefer:

- commit SHA;
- immutable VCS revision;
- signed release tag plus underlying commit;
- another content-addressed source identifier.

The release record should preserve the relationship:

```text
source revision
    ->
build
    ->
artifact digest
```

### 2. Dependencies are explicit

Runtime and build dependencies should be declared rather than discovered
implicitly from the build host.

Use lockfiles, manifests, pinned toolchains, or equivalent mechanisms.

Avoid:

- unpinned downloads;
- mutable remote scripts;
- `curl | sh`;
- floating package versions in release paths;
- implicit global tools;
- mutable base images.

### 3. Builds are isolated and reproducible

Prefer ephemeral, isolated build environments.

A build should not depend on undocumented mutable host state.

Reproducibility should be improved as risk justifies, using mechanisms such
as:

- lockfiles;
- pinned compiler/runtime versions;
- pinned container bases;
- Nix/Guix;
- Bazel or similar controlled build environments;
- vendored dependencies where justified;
- content-addressed caches.

Do not claim a build is hermetic if it can reach untracked mutable inputs.

### 4. Artifacts are immutable

Published artifacts should have an immutable identity.

Examples:

- OCI digest;
- package checksum;
- signed release archive digest;
- content hash;
- immutable package version.

Tags and human-readable versions may exist, but security policy should bind to
immutable identity where possible.

### 5. Build once, promote the same artifact

Do not rebuild an application separately for each environment.

The artifact tested in one stage should be the artifact promoted to the next.

Environment-specific configuration must remain separate from artifact identity.

### 6. Evidence is bound to the artifact

SBOMs, provenance, signatures, test results, and security metadata should be
associated with the exact immutable artifact they describe.

Evidence that cannot be tied to an artifact digest is weaker and may be
misleading.

### 7. Signing without verification is incomplete

A signature that is never verified provides little protection.

Where signing is used, define:

- signer identity;
- trusted identity policy;
- signing point;
- storage location;
- verification point;
- failure behavior.

### 8. Security policy is enforced before trust

Do not treat successful publication as equivalent to approval for deployment
or consumption.

Artifacts may need to satisfy policy before promotion.

Examples:

- signature valid;
- provenance valid;
- source revision allowed;
- vulnerability threshold satisfied;
- required tests passed;
- trusted builder used.

### 9. Exceptions are explicit

Supply-chain controls need an exception mechanism.

Exceptions should be:

- narrow;
- attributable;
- time-bounded where practical;
- reviewable;
- tied to a specific artifact or risk;
- visible in audit history.

Avoid permanent global bypasses.

### 10. Trust is minimized

Do not rely on one privileged CI worker, shared credential, or mutable registry
tag as the root of trust for the entire release process.

Use independent controls where they materially improve security.

---

## Threat model

Before applying controls, identify the relevant threats.

Consider:

- malicious dependency;
- dependency takeover;
- compromised maintainer account;
- typo-squatted package;
- malicious pull request;
- compromised build runner;
- poisoned build cache;
- mutable base image;
- stolen publishing credentials;
- artifact registry compromise;
- signing-key compromise;
- forged provenance;
- tampered artifact;
- compromised release workflow;
- malicious insider;
- vulnerable but legitimate dependency;
- unauthorized promotion;
- artifact substitution between build and deploy;
- stale exception remaining active indefinitely.

Do not assume every repository has the same threat profile.

---

## Source control integrity

Protect release-relevant source history.

Where supported, consider:

- protected default/release branches;
- required reviews;
- status checks;
- restricted force-push;
- restricted tag creation;
- signed commits/tags where the organization requires them;
- CODEOWNERS or equivalent when ownership is known;
- immutable release references.

Do not invent organizational ownership metadata merely to enable a control.

Automation should record the exact source revision used for each artifact.

---

## Trusted and untrusted contributions

Treat pull-request or contribution builds according to their trust level.

Untrusted builds SHOULD NOT automatically receive:

- production credentials;
- signing identities;
- artifact release credentials;
- deployment authority;
- secret-backed integration credentials.

Where release validation requires running code from an untrusted contribution,
use isolated workers and restricted credentials.

Do not assume build scripts are safe merely because they are stored in the
repository.

The repository content itself may be hostile input to the CI system.

---

## Dependency management

Dependencies should be:

- declared;
- versioned;
- reviewable;
- reproducible;
- attributable to an upstream source.

Use the ecosystem's standard lockfile or equivalent where one exists.

Avoid broad dependency updates mixed into unrelated changes.

### Direct dependencies

Before adding a dependency, consider:

- necessity;
- maintenance;
- release history;
- licensing;
- security posture;
- transitive cost;
- update burden;
- ecosystem maturity.

### Transitive dependencies

Do not assume transitive dependencies are harmless because they were not
selected directly.

Where practical:

- inspect dependency trees;
- identify unexpectedly large transitive closure;
- remove unused dependencies;
- monitor vulnerable transitive dependencies.

### Dependency sources

Prefer authenticated, expected package sources.

Avoid implicit fallback to untrusted registries.

Where supported, configure:

- registry allowlists;
- repository pinning;
- checksums;
- lockfile integrity;
- namespace protections.

---

## Dependency confusion and namespace safety

If private package names exist, ensure public registries cannot silently
override them.

Consider:

- explicit registry configuration;
- scoped namespaces;
- source priority;
- package source mapping;
- internal proxy repositories.

Do not allow a build system to search arbitrary public registries before the
intended internal source for private packages.

---

## Toolchain pinning

Release builds should pin or otherwise control critical build tools.

Examples:

- compiler;
- runtime;
- package manager;
- build tool;
- container builder;
- SBOM generator;
- signer;
- scanner;
- reusable CI action/task.

Do not use floating `latest` versions in a release-critical path.

Pinning should be balanced with maintainable upgrade automation.

---

## Base image integrity

For containerized workloads:

- prefer minimal trusted bases;
- pin versions sufficiently for reproducibility;
- use immutable digests for release-critical builds where practical;
- monitor for vulnerabilities;
- rebuild when security updates require it.

Do not treat a pinned digest as "secure forever".

Immutability preserves identity; it does not remove vulnerabilities.

---

## Build isolation

Build workers should minimize persistent trust.

Prefer:

- ephemeral workers;
- clean workspaces;
- isolated credentials;
- isolated caches;
- resource limits;
- restricted host access;
- restricted metadata service access;
- minimal network access where practical.

Avoid sharing broad host mounts or Docker sockets with untrusted build jobs.

If privileged builds are unavoidable, document and isolate them.

---

## Network access during builds

Where practical, control network access during release builds.

Benefits include:

- reducing hidden dependency acquisition;
- preventing unexpected exfiltration;
- improving reproducibility.

Possible models:

- no network after dependency resolution;
- allowlisted package mirrors;
- internal artifact proxies;
- staged dependency fetch followed by offline build.

Do not require full network isolation if the ecosystem cannot support it
without disproportionate complexity.

---

## Build cache security

Caches should improve performance without becoming trusted mutable input.

Prefer:

- content-addressed caches;
- cache keys bound to dependency/build inputs;
- separate trust domains for untrusted branches;
- integrity verification.

Do not let untrusted pull requests poison a cache used by trusted release
builds without validation.

A cache hit must not bypass artifact integrity checks.

---

## Build provenance

Where provenance is required, it should record enough information to identify:

- source revision;
- builder identity;
- build invocation;
- artifact digest;
- relevant build parameters;
- build environment identity where applicable.

Provenance should be generated by the build system, not manually authored.

Do not claim provenance proves more than the build environment actually
guarantees.

---

## SLSA-style maturity

Where useful, use SLSA concepts as a framework for reasoning about:

- build provenance;
- isolated builds;
- trusted builders;
- source integrity;
- artifact integrity.

Do not chase a maturity label solely for marketing.

Use the controls that materially reduce risk for the system.

If a maturity claim is documented, ensure the actual process supports it.

---

## SBOM generation

Generate an SBOM where the artifact's distribution or risk justifies it.

Use a standard format such as:

- SPDX;
- CycloneDX.

The SBOM should:

- correspond to the actual artifact;
- include direct and transitive dependencies where tooling supports it;
- be stored with the release evidence;
- be attributable to the artifact digest/version.

Do not create an SBOM from stale source metadata and imply it describes the
built artifact if it does not.

---

## SBOM validation

Where SBOMs influence policy or customer assurance, validate them.

Check:

- parseability;
- artifact association;
- expected package coverage;
- generation success;
- format/schema validity.

An empty or obviously incomplete SBOM should not count as successful evidence.

---

## Vulnerability scanning

Select scanners appropriate to the artifact and ecosystem.

Potential scan targets include:

- source;
- dependencies;
- container images;
- OS packages;
- IaC;
- secrets;
- licenses.

Avoid redundant tools that provide no materially different coverage.

### Failure policy

Define what happens when vulnerabilities are detected.

Consider:

- severity;
- exploitability;
- reachability;
- fix availability;
- environment exposure;
- compensating controls;
- exception expiration.

Do not encode arbitrary severity policy if organizational requirements are
unknown.

Do not silently ignore scanner failures.

A scanner that failed to run is different from a scan that found zero issues.

---

## License and policy scanning

Where licensing matters:

- detect dependency licenses;
- compare against organizational policy;
- preserve attribution requirements;
- surface unknown/unclassified licenses.

Do not invent license policy.

If organizational policy is absent, report findings rather than making legal
decisions.

---

## Secret scanning

Scan source history and changes for likely credentials.

Where practical:

- scan pre-commit;
- scan pull requests;
- scan repository history periodically;
- block verified high-confidence secrets.

Do not assume secret scanning makes committed secrets safe.

If a real secret is discovered, rotation/revocation is required.

Deleting the string from Git does not invalidate the credential.

---

## Artifact signing

Sign artifacts where integrity and publisher identity matter.

Possible mechanisms include:

- Sigstore/Cosign;
- GPG/PGP;
- package-ecosystem-native signatures;
- hardware-backed signing systems;
- cloud KMS-backed signing.

Prefer short-lived or workload-backed identities over long-lived private keys
where supported.

Signing identity should be scoped to the release process.

Do not store private signing keys in the repository.

---

## Keyless signing

Where the environment supports trusted OIDC/workload identity, keyless
signing can reduce long-lived key management.

Verify:

- expected issuer;
- expected subject/identity;
- workflow/build context where applicable;
- artifact digest.

Do not treat "signed by some valid OIDC identity" as sufficient trust.

Verification policy must define which identities are acceptable.

---

## Key-based signing

If key-based signing is necessary:

- generate keys securely;
- protect private keys with an appropriate secret/KMS/HSM mechanism;
- scope access tightly;
- support rotation;
- document recovery;
- record trust anchors.

For local demos, generated ephemeral keys are acceptable if clearly marked as
non-production.

---

## Signature verification

Verification should occur before the artifact is trusted.

Possible enforcement points:

- release promotion;
- artifact download/install;
- package manager hook;
- deployment admission;
- runtime startup;
- installer/updater.

A verifier should check the immutable artifact identity, not only a mutable
tag or filename.

Verification failure should block trust unless an explicit exception exists.

---

## Attestations

Use attestations for specific claims that policy needs to evaluate.

Examples:

- build provenance;
- SBOM;
- test result;
- security scan result;
- policy decision.

Do not create attestations with vague meaning.

Each attestation should have:

- a clear predicate/claim;
- an identified producer;
- an artifact binding;
- a verification policy.

---

## Artifact registry controls

Artifact registries should support the release trust model.

Consider:

- immutable tags or retention protections;
- RBAC;
- separate read/write roles;
- audit logs;
- replication;
- retention;
- malware/vulnerability scanning;
- signature/attestation storage.

Do not give ordinary consumers publish rights.

Do not share one registry-admin credential across CI jobs.

---

## Promotion

Promotion should reference the same immutable artifact that was built and
verified.

Promotion may involve:

- moving metadata;
- updating desired state;
- changing an environment manifest;
- assigning an immutable release channel;
- copying between registries without rebuild.

Do not rebuild as part of promotion.

Preserve the artifact digest across stages where possible.

---

## Environment trust boundaries

Different environments may have different trust requirements.

For example:

```text
dev
    ->
integration
    ->
staging
    ->
production
```

For each environment, define:

- allowed artifact sources;
- required evidence;
- allowed signer identities;
- approval policy;
- vulnerability policy;
- deployment authority.

Do not assume development policy must equal production policy.

---

## Admission policy

Where the deployment platform supports it, consider admission-time checks for:

- approved registry;
- immutable image reference;
- valid signature;
- valid provenance;
- approved builder;
- vulnerability policy;
- deployment security posture.

Admission policy should fail clearly.

Test both allowed and denied artifacts.

Do not deploy policy that rejects all legitimate workloads because the build
pipeline does not yet produce required evidence.

---

## Release manifest

For important releases, consider producing a machine-readable release record
containing:

- source revision;
- artifact digest;
- version;
- SBOM reference;
- provenance reference;
- signature reference;
- test summary;
- scan summary;
- promotion history.

This can simplify audit and incident response.

Do not duplicate data unnecessarily if the artifact registry and provenance
system already provide an equivalent reliable record.

---

## Rebuild strategy

Security fixes in dependencies may require rebuilding unchanged source.

The release process should support:

- scheduled rebuilds;
- base-image refresh;
- dependency refresh;
- emergency rebuild;
- reproducibility checks.

A source commit being unchanged does not mean the artifact remains secure
forever.

---

## Dependency update automation

Use Dependabot, Renovate, or equivalent where appropriate.

Configure only ecosystems actually present.

Prefer:

- bounded update frequency;
- grouped low-risk updates where useful;
- automatic CI verification;
- explicit review for major updates.

Do not enable auto-merge for high-impact updates without sufficient tests and
policy.

---

## Reproducibility verification

Where reproducible builds are a meaningful goal, periodically test them.

A reproducibility test may:

1. build the same revision independently;
2. compare artifact digests or normalized outputs;
3. investigate differences.

Do not claim deterministic reproducibility based only on a lockfile.

---

## Independent verification

For higher-assurance systems, consider separating build and verification.

Examples:

- independent artifact rebuild;
- independent signature verification;
- separate policy service;
- admission check performed by a different trust domain.

Use this where the risk justifies the operational cost.

---

## Release credentials

Separate credentials by capability.

Examples:

- dependency read;
- artifact publish;
- signing;
- promotion;
- deployment.

Do not use one shared credential for all release operations.

Prefer short-lived credentials minted for a specific job.

---

## Least privilege

CI/release identities should have only required permissions.

Examples:

- test jobs should not publish artifacts;
- pull-request jobs should not sign production releases;
- artifact publishers should not administer the registry;
- signers should not modify source;
- deployment systems should not modify build history.

Review privilege boundaries explicitly.

---

## Branch and tag policy

Release triggers should be explicit.

Examples:

- protected branch;
- signed release tag;
- approved release workflow;
- version manifest change.

Do not publish production artifacts from arbitrary branch pushes unless that is
an intentional design.

Do not trust a tag name without checking the underlying immutable revision.

---

## Release versioning

Use versioning appropriate to the artifact ecosystem.

Possible schemes:

- semantic versioning;
- calendar versioning;
- commit-derived versioning;
- monotonic build numbers.

Whatever scheme is used, map human-readable version to immutable artifact
identity.

Do not rely on version strings alone as integrity controls.

---

## Metadata integrity

Do not allow mutable metadata to override artifact trust silently.

Examples:

- artifact labels;
- registry annotations;
- release notes;
- package descriptions.

Security decisions should bind to immutable evidence where possible.

---

## CI configuration as code

Build/release workflows are security-sensitive code.

Treat changes to them accordingly.

Consider stronger review for:

- CI workflow files;
- release scripts;
- signing configuration;
- registry publication logic;
- dependency source configuration;
- policy configuration.

Do not assume "pipeline YAML" is operational glue rather than privileged code.

---

## Reusable CI components

Reusable actions/tasks/templates should be pinned and versioned.

Avoid consuming mutable references such as:

```text
@main
@master
latest
```

for release-critical dependencies.

If commit-SHA pinning is used, retain a human-readable version comment or
update automation where helpful.

---

## Third-party CI extensions

Before adding third-party CI actions/plugins/tasks, consider:

- publisher trust;
- maintenance;
- permissions;
- network access;
- execution model;
- update policy.

Minimize privileged third-party code in release workflows.

---

## Artifact retention

Define retention according to operational and audit requirements.

Important releases may need longer retention than ephemeral CI artifacts.

Retain enough information to support:

- rollback;
- incident investigation;
- reproducibility analysis;
- vulnerability response.

Do not invent compliance retention periods without requirements.

---

## Revocation and compromise response

The supply chain must support responding to compromise.

Consider procedures for:

- compromised signing identity;
- compromised artifact publisher;
- malicious dependency;
- tampered artifact;
- compromised CI worker;
- poisoned cache;
- vulnerable released artifact.

Response may require:

- credential revocation;
- key rotation;
- artifact quarantine;
- release withdrawal;
- rebuild;
- republish;
- policy update;
- downstream notification.

---

## Artifact quarantine

Where the registry/platform supports it, consider a quarantine or blocked state
for known-bad artifacts.

Do not silently delete evidence needed for incident response.

Prevent quarantined artifacts from being promoted.

---

## Build logs

Build logs are part of operational evidence but may contain sensitive data.

Do not log:

- credentials;
- signing keys;
- tokens;
- secret environment variables;
- private package credentials.

Use masking where supported.

Do not treat logs as the only source of provenance.

---

## Auditability

Important events should be attributable.

Examples:

- release initiated;
- artifact published;
- signature created;
- exception granted;
- promotion approved;
- policy overridden;
- release revoked.

Audit records should identify:

- actor;
- artifact;
- action;
- time;
- result.

---

## Policy exceptions

When an artifact violates policy but must be released, require an explicit
exception.

An exception should include:

- artifact identity;
- failed control;
- justification;
- approver;
- expiration;
- remediation plan where appropriate.

Do not create a generic "skip security" option.

---

## Local development

Do not make local development depend on production signing infrastructure.

Developers should be able to build and test locally.

Where local artifact verification is useful, provide:

- local ephemeral signing;
- test trust roots;
- mock registry;
- local verification commands.

Clearly separate local/demo trust from production trust.

---

## CI verification interface

A repository using this profile should expose obvious supply-chain checks,
either directly or through its existing task runner.

Potential commands:

```text
check
test
build
sbom
scan
sign
verify
provenance
release
```

These names are illustrative.

Do not force a command if the ecosystem has a stronger native mechanism.

---

## Tests

Supply-chain controls must be tested, not only configured.

### Positive-path tests

Where practical, demonstrate:

- expected source revision builds;
- artifact publishes;
- artifact digest is recorded;
- SBOM/provenance is generated;
- signature is valid;
- verifier accepts trusted artifact;
- promotion succeeds.

### Negative-path tests

At least one important failure should be demonstrated.

Examples:

- unsigned artifact rejected;
- artifact signed by wrong identity rejected;
- tampered artifact rejected;
- untrusted branch cannot publish;
- vulnerable artifact blocked under configured policy;
- missing provenance blocks promotion;
- poisoned cache does not bypass verification.

A control that has never been tested in its deny/failure mode is weaker
evidence.

---

## Acceptance criteria

A repository applying this profile is not complete merely because supply-chain
tools are present.

Demonstrate the applicable subset of the following.

### Source and dependencies

- release maps to immutable source revision;
- dependency manifests are explicit;
- lockfiles or equivalent are present where supported;
- release-critical tools are pinned.

### Build

- build runs in a controlled environment;
- hidden host dependencies are minimized;
- artifact identity is immutable;
- build result can be traced back to source.

### Evidence

- SBOM generated where required;
- provenance generated where required;
- signature generated where required;
- evidence binds to artifact digest.

### Verification

- signature/provenance verification runs at a defined trust boundary;
- wrong or missing evidence causes rejection;
- trusted identities are explicitly configured.

### Promotion

- same artifact is promoted without rebuild;
- promotion is auditable;
- environment policy is explicit.

### Security

- untrusted contribution builds do not receive release secrets;
- release identities use least privilege;
- secrets are not committed or logged;
- policy exceptions are explicit.

### Operations

- rebuild is possible;
- compromise/revocation path is documented where risk requires it;
- critical release evidence is retained sufficiently for rollback/investigation.

---

## Optional composition

This profile is intended to compose with multiple recipes and profiles.

Common combinations:

```text
secure-service + software-supply-chain
```

For signed, verifiable deployable services.

```text
platform-control-plane + software-supply-chain
```

For end-to-end provenance, artifact policy, promotion, and admission controls.

```text
library-sdk + software-supply-chain
```

For package signing, dependency hygiene, provenance, and release integrity.

```text
cli + software-supply-chain
```

For trusted binary releases, checksums, signing, and installer verification.

```text
agentic-system + software-supply-chain
```

For signed agent bundles, tool manifests, policy files, prompts, models, and
deployment artifacts.

```text
software-supply-chain + high-assurance
```

For stronger isolation, independent verification, reproducibility checks, and
strict release policy.

---

## Anti-patterns

Do not introduce these without a concrete requirement:

- SBOM generation with no artifact binding;
- signatures that are never verified;
- provenance produced manually;
- provenance claims stronger than the build environment supports;
- multiple scanners with overlapping output and no policy;
- vulnerability dashboards that never block or drive remediation;
- mutable `latest` tags as deployment identity;
- rebuilding per environment;
- long-lived shared release credentials;
- release secrets exposed to pull requests;
- unpinned third-party CI actions;
- `curl | sh` in release pipelines;
- public-registry fallback for private package namespaces;
- build caches shared across trust domains without integrity controls;
- permanent global security bypasses;
- auto-merging major dependency changes without meaningful tests;
- custom cryptography for artifact signing;
- treating a digest as proof of publisher identity;
- treating a signature as proof of vulnerability absence;
- treating an SBOM as proof of secure software.

Do not mistake evidence generation for trust.

Trust comes from verified claims, constrained identities, and enforceable
policy.

---

## Completion evidence

When this profile is applied, the final report should state:

1. immutable source identity used;
2. dependency-locking strategy;
3. toolchain pinning strategy;
4. build isolation model;
5. artifact type and immutable identifier;
6. artifact registry/store;
7. SBOM status;
8. provenance status;
9. signing model;
10. trusted signer identity policy;
11. verification point;
12. vulnerability scanning policy;
13. promotion model;
14. release credential boundaries;
15. untrusted-contribution handling;
16. tests and negative-path checks executed;
17. commands actually run;
18. observed results;
19. exceptions or unresolved gaps;
20. controls deliberately not implemented and why.

Never claim an artifact is "trusted", "verified", "reproducible", "signed",
"provenanced", or "secure" unless the corresponding control was actually
executed and observed, or the claim is explicitly marked as unverified.

---

## Guiding principle

A secure software supply chain should make artifact trust explainable.

A consumer should not need to trust:

- a mutable tag;
- an opaque CI job;
- a shared secret;
- a developer laptop;
- an undocumented manual release step;
- or the assumption that "the registry probably contains what we built."

The chain should preserve a verifiable relationship:

```text
trusted source
    +
declared dependencies
    +
controlled build
    +
immutable artifact
    +
bound evidence
    +
trusted identity
    +
verification
    +
explicit promotion
```

The important question is not:

> Did we generate an SBOM and sign the image?

The important question is:

> Can we prove that this exact artifact came from the expected source through
> the expected build process, and will our systems refuse it when that proof is
> missing or invalid?
