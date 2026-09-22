# library-sdk.md

> Recipe for scaffolding or elevating a reusable library or SDK.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with profiles such as `software-supply-chain.md`,
> `high-assurance.md`, `hostile-input.md`, `zero-trust-service.md`, or
> `ai-security.md` where appropriate.
>
> This recipe defines API, compatibility, packaging, dependency, security,
> testing, documentation, release, and maintenance expectations for software
> intended to be consumed as a library or SDK.
>
> It does not require a specific language, package registry, build system,
> documentation generator, or release platform.

## Purpose

Use this recipe when the repository primarily produces reusable code consumed
by other software.

Typical examples:

- language libraries;
- client SDKs;
- API SDKs;
- security libraries;
- protocol libraries;
- data-format libraries;
- framework extensions;
- developer toolkits;
- reusable internal platform libraries;
- model/provider SDKs;
- authentication/authorization helpers.

Do **not** automatically apply this recipe to:

- services;
- command-line applications;
- standalone operators;
- one-off scripts;
- repositories whose primary artifact is a container image.

The goal is not to maximize abstraction.

The goal is to produce a library whose public contract is:

- small;
- stable;
- understandable;
- testable;
- versioned;
- secure by default;
- easy to integrate;
- hard to misuse accidentally;
- maintainable across releases.

---

## Core principle

A library is an API contract before it is an implementation.

Consumers depend on more than function signatures.

They depend on:

- names;
- types;
- error behavior;
- side effects;
- threading/concurrency behavior;
- serialization;
- compatibility;
- defaults;
- packaging;
- dependency graph;
- release semantics.

Treat public behavior as durable.

Internal structure may change.

Public behavior should change deliberately.

---

## Architectural invariants

A library or SDK SHOULD satisfy these invariants unless the repository has a
documented reason not to.

### 1. Public API is intentionally small

Expose only what consumers need.

Prefer:

```text
small stable public surface
    +
private implementation freedom
```

over exposing internal types and helpers accidentally.

### 2. Compatibility is a first-class constraint

Changes to public behavior require explicit compatibility consideration.

Do not treat compilation success as proof of compatibility.

### 3. Defaults are safe and unsurprising

A consumer using the obvious path should not accidentally:

- disable TLS verification;
- log secrets;
- use infinite retries;
- create unbounded concurrency;
- send telemetry externally;
- mutate global state;
- perform destructive actions.

### 4. Errors are part of the API

Consumers must be able to distinguish important failure classes.

Do not force callers to parse human-readable error strings.

### 5. Dependencies are a cost paid by consumers

Every dependency adds:

- supply-chain exposure;
- install size;
- compatibility risk;
- update burden;
- potential version conflicts.

Keep dependencies intentional.

### 6. Side effects are explicit

A constructor or import should not unexpectedly:

- make network calls;
- modify filesystem state;
- spawn threads;
- emit telemetry;
- read broad environment state;
- create external resources.

### 7. Configuration is explicit

Avoid hidden global configuration.

Prefer explicit client/options objects.

### 8. Versioning communicates compatibility

Release versioning should have meaningful semantics.

Do not publish breaking changes as invisible patch releases.

### 9. Packaging is part of correctness

A library that works from source but fails when installed is broken.

Test the actual published artifact shape.

### 10. Documentation is part of the interface

Consumers should not need to read implementation code to understand normal use.

---

## Library boundary definition

Before implementation, identify:

- target consumers;
- supported language/runtime versions;
- package format;
- public modules/namespaces;
- public types/functions;
- extension points;
- serialization contracts;
- error model;
- thread/concurrency model;
- network behavior;
- compatibility promise.

A useful boundary is:

```text
consumer code
    |
    v
public API
    |
    v
internal implementation
    |
    v
external dependencies / systems
```

The public API should shield consumers from unnecessary internal churn.

---

## Library versus SDK

Clarify which type of artifact this is.

### Library

Usually provides reusable local functionality.

Examples:

- parser;
- crypto wrapper;
- validation engine;
- data structure;
- policy helper.

### SDK

Usually wraps a remote service or protocol.

Examples:

- cloud API client;
- SaaS client;
- model provider SDK;
- internal platform client.

An SDK normally has additional concerns:

- authentication;
- network timeouts;
- retries;
- pagination;
- rate limits;
- transport errors;
- API versioning.

---

## Public API design

Design the public API around consumer tasks.

Prefer:

```text
client.create_resource(...)
client.get_resource(...)
```

over leaking transport primitives such as:

```text
client.request("POST", "/v1/resource", ...)
```

unless low-level control is explicitly part of the product.

Avoid exposing internal architecture through public names.

---

## Public versus private symbols

Use ecosystem conventions to distinguish public and internal code.

Do not accidentally export:

- test helpers;
- generated internals;
- transport implementation;
- temporary experimental APIs.

If consumers cannot import it reliably, do not document it as public.

---

## API ergonomics

Optimize for common use without hiding important behavior.

A typical flow should be obvious.

Example:

```text
configure
    ->
construct client
    ->
call operation
    ->
handle result/error
```

Avoid requiring consumers to understand internal dependency graphs.

---

## Constructors

Constructors should establish local object state.

Avoid network calls in constructors where possible.

Prefer explicit initialization methods if remote discovery is required.

Do not create hidden side effects during import or object construction.

---

## Options/configuration

Prefer explicit options structures over long positional argument lists.

Good options are:

- typed;
- documented;
- defaulted safely;
- forward-compatible where practical.

Avoid "options dictionaries" with arbitrary string keys when the language
supports stronger types.

---

## Global state

Avoid mutable process-global state.

Examples:

- global client;
- global token;
- global logger mutation;
- global retry settings.

Global state harms:

- testing;
- concurrency;
- embedding;
- multi-tenant use.

---

## Thread safety

Document whether public objects are:

- thread-safe;
- goroutine-safe;
- async-safe;
- single-threaded;
- immutable.

Do not leave concurrency semantics ambiguous.

---

## Async APIs

If the ecosystem uses async programming:

- follow ecosystem conventions;
- support cancellation;
- do not block event loops;
- document sync/async variants.

Do not provide fake async wrappers around blocking work if that harms behavior.

---

## Cancellation

Long-running SDK operations should support cancellation where the ecosystem
allows it.

Cancellation should propagate to:

- network requests;
- retries;
- background work.

---

## Context propagation

Where relevant, propagate:

- cancellation;
- deadlines;
- tracing context;
- request identifiers.

Do not hide a separate uncontrolled timeout model beneath caller-provided
deadlines.

---

## Error model

Define error categories intentionally.

Possible categories:

```text
InvalidArgument
AuthenticationError
AuthorizationError
NotFound
Conflict
RateLimited
Timeout
Unavailable
ProtocolError
InternalError
```

Use the language's normal error model.

Do not make consumers inspect strings.

---

## Error wrapping

Preserve root cause while adding context.

Do not leak:

- secrets;
- raw credentials;
- sensitive payloads.

Error messages should be useful but safe.

---

## Retry semantics

For SDKs, retries are part of the contract.

Retry only operations that are safe or explicitly idempotent.

Do not retry:

- invalid input;
- authentication failure;
- authorization denial;
- destructive operations with unknown outcome

without reconciliation or idempotency.

---

## Idempotency

If remote APIs support idempotency keys, expose them appropriately.

For high-level SDK operations, document whether retries are safe.

Do not hide duplicate side-effect risk.

---

## Timeouts

Networked SDKs should have finite timeouts.

Allow caller override within sane bounds.

Do not default to infinite waits.

---

## Connection management

Reuse connections where appropriate.

Document client lifecycle.

Consumers should know whether to:

- reuse one client;
- close client;
- create per request.

Do not create unnecessary connection pools per operation.

---

## Authentication

SDKs should integrate with established authentication mechanisms.

Examples:

- API token;
- OAuth/OIDC;
- workload identity;
- cloud-native credential chain;
- mTLS.

Do not invent custom authentication protocols.

---

## Credential handling

Credentials should:

- remain out of logs;
- remain out of error strings;
- be passed explicitly or discovered through documented mechanisms;
- support rotation where applicable.

Avoid environment-variable magic unless it follows ecosystem/provider
conventions.

---

## TLS

Enable certificate verification by default.

If insecure transport is supported for local development, make it explicit.

Do not expose a deceptively named flag that silently disables verification.

---

## Endpoint configuration

Allow custom endpoints only when consumers need them.

Validate URL/scheme.

Do not let endpoint overrides accidentally bypass expected TLS or trust policy
without explicit intent.

---

## Proxy support

Follow standard proxy conventions where applicable.

Do not implement custom proxy behavior unnecessarily.

---

## Serialization

Serialization formats are contracts.

If consumers persist serialized values, version them.

Do not casually change field names, enum values, or wire format.

---

## Unknown fields

Choose behavior intentionally:

- preserve;
- ignore;
- reject.

For forward-compatible APIs, ignoring unknown response fields is often useful.

For security-sensitive local config, stricter behavior may be appropriate.

---

## Enum evolution

Public enums are compatibility-sensitive.

Adding an enum value can break exhaustive consumer switches.

Where the language/ecosystem has this risk, design for unknown future values.

---

## Nullability / optionality

Be explicit about:

- missing;
- null;
- empty;
- zero.

Do not collapse distinct API states accidentally.

---

## Pagination

SDKs should provide ergonomic pagination.

Possible patterns:

- explicit page API;
- iterator;
- async iterator;
- generator.

Do not silently fetch an unbounded entire dataset by default.

---

## Streaming

If the API supports streaming:

- expose cancellation;
- expose partial errors;
- close resources correctly;
- document thread/async behavior.

Do not buffer unbounded streams into memory.

---

## Retries and streaming

Do not transparently retry a partially consumed stream unless semantics are
well-defined.

---

## Rate limiting

Surface rate-limit information where useful.

Possible fields:

- retry-after;
- quota remaining;
- limit reset.

Do not hide server throttling behind generic errors.

---

## Backoff

Use bounded exponential backoff with jitter where appropriate.

Allow consumers to configure retry behavior if required.

Do not make retry timing unobservable.

---

## Logging

Libraries should not take over application logging.

Prefer:

- injectable logger;
- standard ecosystem logging;
- disabled debug logging by default.

Do not configure global log format or level on import.

---

## Telemetry

Do not emit telemetry externally without explicit product design and
documentation.

If instrumentation exists:

- make it predictable;
- avoid sensitive values;
- allow opt-out where appropriate.

A library should not surprise consumers with network calls.

---

## Metrics

Libraries generally should not expose process-global metrics implicitly.

If metrics hooks are useful, expose integration points.

---

## Tracing

For network SDKs, tracing hooks may be useful.

Follow ecosystem standards.

Do not hard-code one observability vendor.

---

## Extension points

Add extension points only when real use cases require them.

Examples:

- custom transport;
- authentication provider;
- serializer;
- retry policy;
- middleware.

Avoid abstracting every internal component behind interfaces preemptively.

---

## Middleware/hooks

Hooks should have clear ordering and error semantics.

Do not let hooks silently bypass core security checks.

---

## Plugins

Plugin systems create long-term compatibility and security obligations.

Do not introduce one unless the library genuinely needs third-party extension.

---

## Dependency policy

Prefer the standard library where it is sufficient.

For each dependency ask:

- Is it necessary?
- Is it maintained?
- Is it widely used?
- Is the API stable?
- What transitive dependencies arrive?
- Does it constrain consumer versions?

Libraries should be especially conservative with dependencies.

---

## Dependency exposure

Avoid leaking dependency-specific types in the public API unless intentional.

Otherwise replacing the dependency later becomes a breaking change.

---

## Optional dependencies

Use optional dependencies for genuinely optional features.

Do not make every capability optional if it creates an untestable matrix.

---

## Version conflicts

For ecosystems prone to dependency conflicts, minimize restrictive pins on
consumer-facing runtime dependencies while keeping reproducible development
locks where appropriate.

Follow ecosystem norms.

---

## Vendoring

Vendor only when justified.

Vendoring can improve reproducibility but increases maintenance burden.

---

## Package size

Keep package size reasonable.

Do not ship:

- test fixtures;
- build caches;
- credentials;
- large docs artifacts;
- unnecessary binaries.

Test the packaged artifact contents.

---

## Generated code

Generated code should be reproducible.

Document the generator and version.

Do not require ordinary consumers to run code generation after install unless
that is the explicit model.

---

## Native extensions

Native code increases:

- packaging complexity;
- platform matrix;
- security risk;
- ABI concerns.

Use only when justified.

Provide pure-language fallback where practical, not automatically.

---

## Platform support

Define supported:

- operating systems;
- architectures;
- runtime versions;
- language versions.

Do not claim support for environments not tested.

---

## Runtime support policy

Document minimum supported runtime/compiler versions.

Avoid dropping support accidentally through dependency updates.

---

## API compatibility

Treat as potentially breaking:

- removing symbols;
- renaming symbols;
- changing parameters;
- changing return types;
- changing defaults;
- changing error types;
- changing exception behavior;
- changing serialization;
- changing thread safety;
- changing side effects.

---

## Semantic versioning

Use semantic versioning where it fits the ecosystem.

Typical interpretation:

```text
MAJOR = incompatible public change
MINOR = backward-compatible capability
PATCH = backward-compatible fix
```

If using another versioning scheme, document compatibility semantics.

---

## Pre-1.0 versions

Pre-1.0 does not mean compatibility is irrelevant.

Be explicit about expected stability.

Do not use perpetual `0.x` as an excuse for arbitrary breaking changes.

---

## Deprecation

Deprecate before removing when practical.

A deprecation should provide:

- deprecated API;
- replacement;
- migration guidance;
- removal timeline or release boundary where appropriate.

Do not silently remove widely used APIs.

---

## Compatibility tests

Where public API stability matters, use automated compatibility checks.

Possible checks:

- exported symbol diff;
- ABI check;
- API snapshot;
- schema diff.

Use ecosystem-appropriate tooling.

---

## Behavioral compatibility

Symbol compatibility is not enough.

A change can be breaking if it alters:

- retry count;
- timeout default;
- ordering;
- error class;
- serialization;
- authentication behavior.

Add regression tests for public behavior.

---

## ABI compatibility

For native/shared libraries, define whether ABI compatibility is promised.

Do not imply ABI stability if only source compatibility is tested.

---

## Source compatibility

Define whether consumers should expect existing source to compile after minor
updates.

---

## Binary compatibility

Relevant in some ecosystems.

Test it if promised.

---

## Protocol compatibility

For SDKs implementing a wire protocol, maintain protocol fixtures or contract
tests.

---

## API service versioning

If the SDK wraps a remote service, distinguish:

- SDK version;
- service API version.

Do not couple them unnecessarily.

---

## Service evolution

SDKs should tolerate additive service changes where practical.

Examples:

- unknown response fields;
- unknown enum values;
- new optional fields.

Do not hard-fail on harmless forward-compatible additions unless strictness is
required.

---

## Feature detection

Prefer capability detection or explicit API versioning over brittle version
string comparisons.

---

## Experimental APIs

Experimental APIs should be clearly marked.

Keep them out of the stable namespace where ecosystem conventions support it.

Do not let experimental behavior accidentally become de facto stable.

---

## Security-sensitive APIs

Security libraries require especially careful API design.

Avoid easy-to-misuse primitives.

Prefer safe high-level operations.

Example:

```text
verify_token(token, expected_issuer, expected_audience)
```

over exposing raw signature checks alone when the higher-level validation is
the intended secure operation.

---

## Cryptography

If the library uses cryptography:

- use established primitives/libraries;
- avoid custom crypto;
- validate parameters;
- use safe defaults.

Do not expose weak algorithms by default.

---

## Randomness

Use cryptographically secure randomness for security-sensitive values.

Do not use deterministic pseudo-random functions for secrets unless
intentionally seeded by a secure design.

---

## Secret handling

Avoid storing secrets in long-lived object fields unnecessarily.

Redact:

- `repr`;
- `toString`;
- logs;
- debug dumps.

Do not make secret-bearing objects trivially printable.

---

## Constant-time behavior

For low-level cryptographic/security code, use constant-time operations where
relevant.

Do not claim side-channel resistance without evidence.

---

## Input validation

Validate all external input.

Libraries often receive hostile input from consumers indirectly.

Consider:

- size limits;
- recursion depth;
- malformed encodings;
- archive expansion;
- integer overflow;
- path traversal.

Compose with `hostile-input.md` for parser-heavy libraries.

---

## Parsing

Parsers should fail safely.

Do not:

- panic/crash on malformed input;
- allocate unbounded memory;
- recurse without limits.

Fuzz security-sensitive parsers.

---

## Memory safety

Prefer memory-safe language/runtime facilities where available.

For unsafe/native sections:

- isolate them;
- minimize surface;
- test aggressively;
- document invariants.

---

## Resource ownership

Public APIs should make resource lifetime clear.

Examples:

- file handles;
- sockets;
- streams;
- threads;
- native memory.

Use language-native lifecycle patterns.

---

## Cleanup

Support deterministic cleanup where necessary.

Do not rely solely on garbage collection for scarce external resources.

---

## Test strategy

A reusable library needs deeper compatibility evidence than a single
application.

### Unit tests

Test public and internal behavior.

### Public API tests

Test from the consumer perspective.

Avoid reaching into internals for every assertion.

### Integration tests

For SDKs, exercise real or representative network behavior.

### Contract tests

Validate protocol/API compatibility.

### Regression tests

Every public-behavior defect should become a regression test where practical.

### Packaging tests

Install the built artifact into a clean environment and import/use it.

---

## Consumer tests

Create small consumer fixtures if useful.

Verify typical installation and usage.

Do not test only from the repository source tree.

---

## Matrix testing

Test supported runtime/language versions according to actual support policy.

Do not maintain an enormous matrix for unsupported environments.

---

## Fuzzing

Consider fuzzing:

- parsers;
- codecs;
- protocol decoders;
- security-sensitive validation.

Crashes should become regressions.

---

## Property-based testing

Useful properties may include:

- encode/decode round-trip;
- normalization idempotency;
- parse never crashes;
- verify rejects mutated signatures;
- serialization preserves semantic identity.

---

## Mutation testing

For high-assurance/security libraries, mutation testing may expose weak test
suites.

Use where value justifies runtime cost.

---

## Concurrency tests

For thread-safe/async-safe APIs, test:

- concurrent calls;
- cancellation;
- shutdown;
- race conditions.

Use race detectors where the ecosystem provides them.

---

## Leak testing

For native/resource-heavy libraries, test:

- file descriptors;
- memory;
- threads;
- sockets.

---

## Network simulation

SDKs should test:

- timeout;
- connection reset;
- malformed response;
- partial response;
- rate limit;
- server error;
- retries.

Do not test only HTTP 200.

---

## Unknown outcome

If an SDK wraps side-effecting APIs, represent ambiguous outcomes.

Example:

```text
request timed out after server may have committed write
```

Do not transparently retry unless idempotency guarantees safety.

---

## Mocking

Mocks are useful for consumer logic, but do not let the entire SDK test suite
mock away the protocol.

Keep contract/integration coverage.

---

## Documentation

At minimum document:

- installation;
- supported versions;
- quickstart;
- authentication;
- configuration;
- errors;
- retries/timeouts;
- thread/async safety;
- examples;
- migration/deprecation;
- security considerations.

---

## API reference

Generate API reference from public symbols where ecosystem tooling supports it.

Review generated docs for clarity.

Do not publish internal symbols accidentally.

---

## Examples

Examples should:

- compile/run;
- use current API;
- avoid insecure shortcuts;
- avoid fake production credentials.

Test examples where practical.

---

## README

The README should quickly answer:

- what the library does;
- how to install;
- minimal usage;
- support/version policy;
- documentation location.

Do not bury basic usage under architecture discussion.

---

## Changelog

Maintain release notes/changelog when consumers need upgrade visibility.

Call out:

- breaking changes;
- deprecations;
- security fixes;
- behavior changes.

Do not generate meaningless commit dumps as release notes.

---

## Migration guides

Provide migration guidance for major/breaking releases.

Show before/after code where useful.

---

## License

Do not invent or change the project license without explicit project evidence
or instruction.

Ensure package metadata matches the repository's actual license.

---

## Package metadata

Set accurate metadata such as:

- name;
- version;
- description;
- license;
- repository;
- supported runtime versions.

Do not invent author/maintainer information.

---

## Build reproducibility

The package build should be repeatable.

Pin build tooling sufficiently.

Use clean builds for release.

---

## Source distributions

If the ecosystem publishes source packages, test them.

Do not test only prebuilt binaries/wheels.

---

## Binary distributions

If publishing binary artifacts, test representative target platforms.

Do not claim platform support from cross-compilation alone without runtime
verification where risk warrants it.

---

## Package signing

If ecosystem/package signing is required, define verification.

Compose with `software-supply-chain.md`.

Signing without consumer verification or repository enforcement may add little
value.

---

## Checksums

Publish checksums for downloadable binary/archive artifacts where useful.

Bind them to release identity.

---

## SBOM / provenance

For distributed security-sensitive libraries, consider:

- SBOM;
- build provenance;
- signed artifacts.

Use according to risk and consumer requirements.

---

## Release process

A release should be reproducible and intentional.

A typical flow may be:

```text
version change
    ->
tests
    ->
compatibility checks
    ->
package build
    ->
package install test
    ->
sign/provenance if required
    ->
publish
    ->
verify published artifact
```

Do not treat successful upload as proof of a valid release.

---

## Publish credentials

Scope package publishing credentials narrowly.

Prefer short-lived/federated identities where supported.

Do not store registry tokens in the repository.

---

## Release immutability

Published versions should be immutable.

Do not overwrite an existing version with different contents.

If a release is bad, publish a new version or yank according to ecosystem
rules.

---

## Yank/deprecate

Know how the package ecosystem handles compromised/broken releases.

Use yank/deprecation mechanisms instead of mutating history.

---

## Security advisories

For security-sensitive libraries, document how vulnerabilities are reported
and how fixes are communicated.

Do not invent security contacts if the project has none.

---

## CVEs/advisories

If the project participates in vulnerability disclosure, ensure affected
version ranges are accurate.

Do not overstate impact.

---

## Consumer trust

A consumer should be able to determine:

- what version they installed;
- what dependencies it includes;
- whether the release is authentic where relevant;
- what compatibility to expect.

---

## Breaking change review

Before releasing a breaking change, explicitly review:

- API;
- behavior;
- serialization;
- errors;
- defaults;
- dependencies;
- supported runtimes.

Breaking changes should be deliberate.

---

## Dependency upgrades

For libraries, dependency upgrades can break consumers even when your own tests
pass.

Review:

- minimum versions;
- upper bounds;
- transitive conflicts;
- runtime support.

---

## Minimum dependency versions

If claiming support for minimum dependency versions, test them.

Do not specify theoretical lower bounds that have never been exercised.

---

## Upper bounds

Avoid unnecessary tight upper bounds that block consumers from upgrading.

Use them only when compatibility is known to break.

---

## Lockfiles

Follow ecosystem norms.

Applications often commit full lockfiles.

Libraries may need separate reproducible dev locks while allowing consumer
resolution flexibility.

Do not impose application-style dependency pinning blindly on libraries.

---

## Build-time versus runtime dependencies

Keep build/test dependencies out of consumer runtime where possible.

---

## Optional features

Feature flags/extras should:

- be documented;
- have clear dependency impact;
- be tested.

Avoid combinatorial feature matrices.

---

## Serialization compatibility tests

If consumers persist library-generated data, keep fixtures from older versions
and test new readers/writers against them.

---

## Backward-readable formats

Where practical, new versions should read old durable data.

Do not silently strand consumer data.

---

## Forward compatibility

If forward compatibility is a requirement, define what older versions do with
new fields/variants.

---

## Stable identifiers

If public APIs expose IDs, fingerprints, hashes, or rule names used by
consumers, treat them as compatibility-sensitive.

Do not repurpose an existing identifier to mean something new.

---

## Environment behavior

Avoid environment-dependent behavior unless documented.

Examples:

- proxy variables;
- credentials;
- locale;
- timezone;
- filesystem paths.

---

## Locale/timezone

Normalize or document timezone behavior.

Avoid parsing dates using machine-local defaults unexpectedly.

---

## Deterministic output

Where consumers compare outputs, keep ordering stable.

Examples:

- serialized maps;
- findings;
- generated config.

Sort where useful.

---

## Feature discovery

If the SDK talks to evolving backends, expose server capability detection where
needed.

Do not force consumers to infer capability from server version strings alone.

---

## Deprecation warnings

Warnings should be useful and not excessively noisy.

Emit once where appropriate.

Do not break machine output with unexpected warning text.

---

## Exception safety

In languages with exceptions, document which public calls can throw and what
resource state remains after failure.

---

## Panic policy

For languages with panics/assertions, public APIs should not panic on ordinary
invalid external input.

Reserve panics/assertions for programmer invariants where ecosystem norms allow.

---

## Unsafe escape hatches

Sometimes consumers need low-level access.

If exposing an unsafe/advanced API:

- mark it clearly;
- document invariants;
- keep it separate from the normal path.

Do not make the insecure path easier than the safe path.

---

## Test doubles

SDKs may provide official test helpers/fakes when valuable.

Keep them separate from production API.

Do not make fake behavior diverge wildly from real protocol semantics.

---

## Sandbox/test endpoints

If the remote service has sandbox environments, document how the SDK selects
them safely.

Avoid accidentally defaulting to production or sandbox contrary to user
expectation.

---

## Local emulation

Provide local emulation only if it materially helps consumers.

Do not build a fake service that becomes a second product to maintain without
need.

---

## Observability hooks

Expose hooks for:

- tracing;
- request IDs;
- metrics;
- logging

when consumers need them.

Keep them optional and vendor-neutral where practical.

---

## Performance

Benchmark only when performance matters to the library's contract.

Measure representative workloads.

Avoid microbenchmarks that do not reflect consumer usage.

---

## Allocation behavior

For low-level/high-throughput libraries, monitor:

- allocations;
- memory growth;
- copying.

Do not optimize prematurely if correctness/clarity suffer.

---

## Caching

If the library caches:

- bound the cache;
- document lifecycle;
- define thread safety;
- avoid caching secrets longer than necessary.

Do not introduce hidden unbounded process-global caches.

---

## Background threads

Avoid spawning background threads/tasks unless required.

If used:

- document lifecycle;
- allow shutdown;
- propagate cancellation;
- avoid preventing process exit.

---

## Fork/process behavior

For ecosystems where process forking is relevant, document client safety across
fork boundaries.

---

## Signal handling

Libraries should generally not install global signal handlers.

Leave process lifecycle to applications unless signal management is the
library's explicit purpose.

---

## Filesystem

Avoid writing files automatically unless the API promises it.

Use caller-provided paths/directories.

Do not write to current working directory implicitly.

---

## Permissions

If writing sensitive files:

- use restrictive permissions;
- avoid predictable insecure temp paths;
- clean up appropriately.

---

## Temporary files

Use secure temporary-file primitives.

Do not construct predictable temporary filenames manually.

---

## Network discovery

Do not perform network discovery on import.

If service discovery is needed, make it explicit and bounded.

---

## DNS

Do not treat DNS identity as authentication.

Use TLS/identity controls where required.

---

## Proxying and SSRF

SDKs accepting caller-controlled endpoints may become SSRF primitives.

Validate and document endpoint controls where the SDK runs in privileged
services.

---

## Compliance claims

Do not claim a library makes consumers compliant with a framework by itself.

Security libraries provide controls, not organizational compliance.

---

## Threat model

For security-sensitive libraries, document:

- trusted inputs;
- untrusted inputs;
- secret material;
- unsafe operations;
- external dependencies;
- known non-goals.

---

## Security review triggers

Require stronger review for changes to:

- authentication;
- authorization helpers;
- cryptography;
- serialization;
- parsers;
- certificate validation;
- secret handling;
- public defaults;
- retry semantics;
- unsafe/native code.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic library/SDK repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── src/
│   ├── public API
│   └── internal implementation
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── compatibility/
│   ├── packaging/
│   └── security/
├── examples/
├── docs/
│   ├── api/
│   ├── guides/
│   └── adr/
├── CHANGELOG.md
└── <ecosystem package/build files>
```

Only create directories that contain meaningful content.

---

## Verification interface

A library/SDK repository should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-integration
test-compatibility
test-packaging
build
package
verify-package
```

These names are illustrative.

Use the ecosystem's native tooling where appropriate.

---

## Acceptance criteria

A library or SDK is not complete because its own source tree tests pass.

Demonstrate the applicable subset of the following.

### Public API

- public surface is explicit;
- normal usage is ergonomic;
- internal symbols are not exported accidentally;
- errors are structured/documented;
- side effects are explicit.

### Compatibility

- versioning semantics are documented;
- compatibility-sensitive behavior is tested;
- deprecation path exists where needed;
- durable serialization/protocol compatibility is tested where relevant.

### Dependencies

- runtime dependencies are justified;
- unnecessary transitive dependencies are avoided;
- consumer version constraints are intentional.

### Packaging

- package builds;
- package contents are correct;
- clean-environment install works;
- minimal consumer example works from installed artifact;
- supported runtime metadata is accurate.

### Security

- TLS verification is safe by default for network SDKs;
- secrets do not appear in logs/errors;
- hostile input is rejected safely;
- auth credentials are scoped/handled intentionally;
- dangerous low-level APIs are clearly separated.

### Reliability

- timeouts exist for network calls;
- retry semantics are documented;
- idempotency behavior is understood;
- cancellation works where supported.

### Documentation

- install instructions work;
- quickstart uses current API;
- supported versions are documented;
- error/retry/auth behavior is documented;
- breaking changes are visible.

### Release

- artifact version is immutable;
- release can be reproduced;
- published artifact is verified after publish where practical;
- signing/provenance is verified where required.

---

## Optional composition

Common combinations:

```text
library-sdk + software-supply-chain
```

For signed/verifiable package releases, provenance, SBOMs, and trusted
publishing.

```text
library-sdk + high-assurance
```

For deeper compatibility testing, mutation testing, fuzzing, and stronger
release evidence.

```text
library-sdk + hostile-input
```

For parsers, decoders, archive libraries, protocol libraries, and libraries
processing attacker-controlled data.

```text
library-sdk + zero-trust-service
```

For client SDKs that implement workload identity, authn/authz, or sensitive
service interactions.

```text
library-sdk + ai-security
```

For AI SDKs exposing model, tool, prompt, or agent capabilities where model
output and user content are untrusted.

---

## Anti-patterns

Avoid:

- huge public APIs by default;
- exposing internal dependency types accidentally;
- constructors that make hidden network calls;
- process-global mutable clients;
- disabling TLS verification by default;
- infinite network timeouts;
- retries on unknown-outcome writes;
- errors that require string parsing;
- one release changing serialization silently;
- tight dependency upper bounds without evidence;
- unnecessary heavyweight runtime dependencies;
- publishing packages that were never installed from the built artifact;
- testing only from source checkout;
- mutable package versions;
- overwriting bad releases;
- examples that use deprecated APIs;
- hidden telemetry;
- logging credentials;
- arbitrary environment-variable magic;
- plugin systems with no concrete need;
- perpetual pre-1.0 instability used as an excuse for careless breaking changes;
- claiming semantic versioning while shipping breaking patch/minor releases.

Do not mistake abstraction for a good API.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. library/SDK purpose;
2. target consumers;
3. supported runtime/platform versions;
4. public API boundary;
5. error model;
6. configuration model;
7. concurrency/async guarantees;
8. dependency strategy;
9. compatibility/versioning policy;
10. serialization/protocol compatibility model;
11. authentication/credential model where applicable;
12. timeout/retry/idempotency model where applicable;
13. packaging/publishing model;
14. deprecation/migration model;
15. unit/integration/compatibility tests executed;
16. packaging/install tests executed;
17. security/fuzz/property tests executed where applicable;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a library is "stable", "backward compatible", "secure", or
"production ready" solely because it compiles or because its internal unit
tests pass.

Describe the public-contract, packaging, compatibility, and security evidence
that was actually exercised.

---

## Guiding principle

A good library gives consumers leverage without giving them maintenance debt.

Its public API should be smaller than its implementation.

Its defaults should be safer than its escape hatches.

Its errors should be machine-usable.

Its side effects should be explicit.

Its dependencies should be justified.

Its package should work outside its own repository.

Its releases should communicate compatibility honestly.

And consumers should be able to upgrade with confidence because the library
treats its public behavior as a contract, not an implementation detail.
