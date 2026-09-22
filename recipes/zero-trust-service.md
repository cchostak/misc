# zero-trust-service.md

> Security profile for applying zero-trust principles to networked services.
>
> Apply under `AGENTS.md`, together with `SCAFFOLD.md` or `ELEVATE.md`, and
> normally compose with `secure-service.md`.
>
> This profile defines trust, identity, authorization, network, credential,
> policy, telemetry, and verification expectations for services operating in
> zero-trust environments.
>
> It does not require a specific cloud, service mesh, identity provider,
> policy engine, proxy, gateway, or Kubernetes platform.

## Purpose

Use this profile when a service crosses meaningful trust boundaries or handles
sensitive operations, identities, data, infrastructure, or production
workloads.

Typical examples:

- internet-facing services;
- internal APIs handling sensitive data;
- service-to-service platforms;
- multi-tenant systems;
- administrative/control-plane services;
- systems handling privileged operations;
- gateways and brokers;
- production services in segmented environments;
- workloads requiring explicit workload identity.

The goal is not to deploy a "zero trust product".

The goal is to ensure that trust is:

- explicit;
- narrow;
- continuously re-established where appropriate;
- based on verifiable identity and policy;
- not inferred from network location;
- observable;
- revocable;
- testable.

---

## Core principle

Do not trust a request merely because it originates from:

- an internal network;
- a VPN;
- a private subnet;
- a cluster namespace;
- a known hostname;
- a sidecar;
- a corporate device;
- another backend service;
- a previously authenticated session;
- a familiar source IP.

Network position may inform policy.

It must not silently replace identity and authorization.

The service should answer, for every protected operation:

```text
Who or what is calling?

How was that identity established?

What action is being requested?

On what resource?

Under what context?

Which policy authorizes it?

What evidence was used?

What happens if identity or policy evaluation fails?
```

---

## Zero-trust invariants

A service applying this profile SHOULD satisfy these invariants unless a
documented requirement justifies otherwise.

### 1. Verify explicitly

Protected operations require an authenticated identity and an explicit
authorization decision.

Do not rely solely on:

- source network;
- source IP;
- namespace membership;
- process location;
- possession of a route;
- presence behind a reverse proxy.

### 2. Least privilege

Identities should receive only the permissions required for their role.

Privileges should be scoped by:

- action;
- resource;
- environment;
- tenant;
- service;
- time;
- workflow context

where the platform and application model support it.

Avoid broad wildcard permissions.

### 3. Assume breach

Design as though one component, credential, workload, or network segment may
eventually be compromised.

A compromised component should not automatically grant unrestricted lateral
movement.

### 4. Workload identity is explicit

Service-to-service calls should identify the calling workload where the
environment supports it.

Prefer workload identity and short-lived credentials over static shared
secrets.

### 5. Authentication and authorization are distinct

Authentication establishes identity.

Authorization decides permitted action.

Do not treat successful authentication as permission to access every protected
resource.

### 6. Deny is the safe default

When identity, policy, context, or required security state cannot be
established, protected operations should fail closed.

Do not silently convert:

- identity-provider failure;
- token-validation failure;
- policy-engine failure;
- malformed policy response;
- missing attributes

into allow decisions.

### 7. Trust is scoped, not transitive

If service A trusts service B, that does not mean A trusts every identity,
request, or payload B can originate.

Avoid accidental trust chains.

### 8. Credentials are short-lived where practical

Prefer short-lived, automatically rotated credentials.

Avoid long-lived bearer secrets distributed broadly across workloads.

### 9. Authorization is resource-aware

Authorization should consider the resource being accessed, not only the route
or operation name.

Where relevant, include:

- tenant;
- owner;
- environment;
- object ID;
- classification;
- sensitivity;
- operation.

### 10. Policy decisions are observable

Important allow/deny decisions should be diagnosable.

Do not log sensitive policy inputs unnecessarily, but preserve enough context
to understand why a request was allowed or denied.

### 11. Network controls are defense in depth

Segmentation, firewalls, service meshes, and network policy can reduce attack
surface.

They do not replace application authorization.

### 12. Break-glass access is exceptional

Emergency access may exist, but it must be explicit, attributable, bounded,
and observable.

---

## Trust model

Before implementation, identify all trust boundaries.

Document at minimum:

- human identities;
- workload identities;
- machine identities;
- administrative identities;
- external users;
- external services;
- data stores;
- message brokers;
- control planes;
- identity providers;
- policy decision points;
- policy enforcement points.

A simple model may look like:

```text
User
  |
  | authenticated request
  v
Edge / Gateway
  |
  | identity context
  v
Service A
  |
  | workload identity
  v
Service B
  |
  | scoped database identity
  v
Datastore
```

The diagram is not the control.

The implementation must enforce the represented boundaries.

---

## Identity taxonomy

Do not collapse all principals into "users".

Identify the actor type.

Typical classes:

### Human identity

A person authenticated through an identity provider.

### Workload identity

A running service, job, function, or workload instance.

### Automation identity

CI/CD, scheduled jobs, controllers, deployment automation, or platform
services.

### Device identity

A managed machine, workstation, hardware device, or service appliance.

### External system identity

A third-party API, webhook producer, partner system, or federated workload.

Different identity classes may require different authentication methods and
authorization policies.

---

## Human authentication

If humans call the service directly or indirectly:

- use an established identity provider where available;
- use standard protocols such as OAuth 2.x / OpenID Connect where appropriate;
- validate signatures;
- validate issuer;
- validate audience;
- validate expiration;
- validate not-before where used;
- handle key rotation;
- reject unsupported or unexpected token algorithms.

Do not implement custom password authentication when an established identity
provider should own authentication.

Do not log raw access or refresh tokens.

---

## Workload identity

For service-to-service calls, prefer identity tied to the workload runtime.

Examples may include:

- SPIFFE/SPIRE identities;
- cloud workload identity;
- Kubernetes service-account-backed federation;
- short-lived client certificates;
- short-lived OAuth/OIDC credentials;
- platform-native workload credentials.

The profile does not mandate one implementation.

Prefer:

```text
workload
   ->
short-lived identity
   ->
scoped authorization
```

over:

```text
workload
   ->
shared static API key
```

Shared secrets may be acceptable in constrained environments, but should not be
the default for mature service-to-service trust.

---

## Identity propagation

Propagate identity only when downstream authorization actually needs it.

Distinguish:

- caller identity;
- authenticated end-user identity;
- service/workload identity;
- delegated identity.

Do not blindly forward user tokens across arbitrary service boundaries.

Do not allow one service to manufacture identity claims for another unless
that delegation model is intentional and verifiable.

Where delegation is required, scope it.

---

## Token validation

For bearer tokens, validate all security-relevant claims required by the
protocol and architecture.

Potential checks include:

- cryptographic signature;
- issuer;
- audience;
- expiration;
- not-before;
- token type;
- scope;
- authorized party/client ID;
- tenant;
- subject;
- nonce where applicable.

Do not accept a token merely because it parses.

Do not accept tokens intended for another audience.

Do not rely on unsigned token fields for authorization.

---

## Token lifetime

Keep privileged access tokens short-lived where practical.

Do not make token expiration so long that revocation becomes operationally
meaningless.

Long-running processes should refresh identity through a secure mechanism
rather than relying indefinitely on one static bearer token.

---

## Session security

Where browser or interactive sessions exist:

- use secure session identifiers;
- rotate identifiers after privilege changes where appropriate;
- configure secure cookie attributes;
- define idle and absolute session lifetime;
- protect against CSRF where relevant;
- provide session revocation where required.

Do not treat possession of a browser session as authorization for every
backend action.

---

## Authorization model

Choose an authorization model appropriate to the system.

Possible models include:

- RBAC;
- ABAC;
- ReBAC;
- capability-based authorization;
- ownership-based policy;
- policy-as-code.

The model should represent the real security requirements.

Do not introduce a policy engine merely to avoid writing a handful of clear,
well-tested authorization rules.

---

## Policy decision and enforcement

Identify:

- **Policy Decision Point (PDP)** — where authorization is evaluated;
- **Policy Enforcement Point (PEP)** — where the result is enforced.

These may be inside the application or externalized.

Examples:

```text
Request
  |
  v
PEP
  |
  +--> PDP
  |      |
  |      +--> allow / deny
  |
  v
Protected operation
```

The protected operation must not proceed before the applicable authorization
decision is enforced.

Do not perform expensive or sensitive side effects before authorization.

---

## Application-level authorization

Application authorization is normally required even when infrastructure
performs identity-aware access control.

Examples:

A gateway may establish:

```text
caller = Alice
```

but the application may still need to decide:

```text
Can Alice delete resource X belonging to tenant Y?
```

Infrastructure access control should not replace domain authorization.

---

## Tenant isolation

For multi-tenant systems:

- derive tenant context from trusted identity/policy where practical;
- scope data access by tenant;
- scope caches by tenant;
- scope authorization by tenant;
- scope background work by tenant;
- prevent tenant ID confusion;
- test cross-tenant denial.

Never trust a caller-supplied tenant identifier merely because it exists in a
header or request body.

---

## Administrative access

Administrative operations require stronger boundaries.

Consider:

- separate administrative roles;
- stronger authentication;
- narrower network exposure;
- explicit approval;
- short-lived elevated privilege;
- detailed audit events.

Do not hide administrative privilege behind an undocumented boolean such as:

```text
is_admin = true
```

without a defined authorization model.

---

## Just-in-time privilege

Where supported, prefer temporary elevation for sensitive operations over
permanent standing privilege.

Possible controls:

- time-bound role activation;
- approval workflow;
- short-lived credentials;
- scoped administrative sessions.

Do not add complex JIT infrastructure where the risk does not justify it.

---

## Break-glass access

Emergency access should be:

- separately identifiable;
- strongly authenticated;
- narrowly scoped;
- time-bounded where possible;
- logged;
- reviewed after use.

Do not use the break-glass path for routine operations.

Do not make emergency access depend on the exact control plane that may be
unavailable during an emergency unless that dependency is acceptable.

---

## Service-to-service authorization

Do not stop at workload authentication.

A valid service identity should receive only the operations it needs.

Example:

```text
payments-api
    may:
        charge-payment
        read-payment-status

payments-api
    may not:
        administer-users
        modify-policy
```

Avoid generic "internal-service" roles with broad permissions.

---

## Mutual TLS

mTLS may be useful for workload identity and channel protection.

Use it when the platform and threat model justify it.

mTLS can provide:

- peer authentication;
- transport confidentiality;
- transport integrity.

mTLS does **not** automatically provide application authorization.

A valid certificate does not mean the caller may perform every operation.

Do not deploy a service mesh solely to obtain the label "zero trust".

---

## Service mesh

A service mesh MAY provide useful capabilities such as:

- workload identity;
- mTLS;
- traffic policy;
- telemetry;
- authorization enforcement.

It also introduces:

- control-plane complexity;
- certificate lifecycle;
- proxy behavior;
- policy complexity;
- debugging overhead;
- new failure modes.

Use a mesh when its capabilities solve concrete requirements.

Do not use one as a prerequisite for this profile.

---

## Network segmentation

Apply network controls where they materially reduce attack surface.

Examples:

- firewall rules;
- security groups;
- namespace/network policy;
- subnet boundaries;
- ingress/egress policy;
- private service endpoints.

Prefer explicit allowed paths.

But remember:

```text
network allowed
```

does not imply:

```text
operation authorized
```

Network controls are supplementary.

---

## Egress control

Outbound connectivity is part of the trust model.

Where risk justifies it:

- restrict destination networks;
- restrict protocols/ports;
- use explicit proxies;
- allowlist required services;
- block cloud metadata endpoints;
- prevent direct internet egress for workloads that do not need it.

Egress restrictions can reduce:

- command-and-control access;
- secret exfiltration;
- SSRF impact;
- dependency confusion;
- accidental external dependencies.

Do not block required functionality without providing an intentional path.

---

## SSRF

Services making outbound requests based on caller input require explicit SSRF
controls.

Consider:

- URL parsing with established libraries;
- allowed schemes;
- hostname allowlists;
- DNS rebinding considerations;
- private/reserved address restrictions;
- redirect validation;
- cloud metadata protection;
- timeout and response-size limits.

Identity-aware networking does not remove SSRF risk.

---

## Datastore identity

Prefer datastore access tied to the workload or service role rather than one
shared administrator credential.

Grant only required operations.

Examples:

- read-only service account;
- table/schema-scoped role;
- separate migration identity;
- separate backup identity.

Do not give runtime services schema-admin permissions merely because it is
convenient.

---

## Control-plane access

Separate runtime workload identity from platform administration.

A service should not have access to:

- orchestration admin APIs;
- CI/CD secrets;
- signing infrastructure;
- cluster-admin credentials;
- identity-provider administration

unless that is part of its explicit purpose.

Compromising an application workload should not automatically compromise the
delivery/control plane.

---

## Secrets

Zero trust does not mean "put everything in a secret manager" and stop.

For each secret, determine:

- why it exists;
- who needs it;
- how it is delivered;
- lifetime;
- rotation;
- revocation;
- auditability.

Prefer eliminating static secrets where workload identity can replace them.

Where secrets remain:

- scope narrowly;
- rotate;
- do not commit;
- do not log;
- do not bake into images;
- do not share broadly.

---

## Cryptographic trust

Use established cryptographic libraries and protocols.

Do not invent:

- custom token formats;
- custom encryption schemes;
- custom certificate validation;
- custom signature algorithms.

Trust roots must be explicit.

Examples:

- trusted CA;
- OIDC issuer;
- signing identity;
- policy trust bundle.

Protect trust-root modification more strongly than ordinary configuration.

---

## Trust bootstrap

Every identity system has a bootstrap trust point.

Document it.

Examples:

- configured OIDC issuer;
- workload identity federation;
- root CA;
- SPIFFE trust domain;
- cloud identity provider;
- signing trust root.

Do not describe a system as trustless.

Zero trust means eliminating implicit trust, not eliminating all trust.

---

## Policy-as-code

Where policy complexity justifies it, policy-as-code can improve:

- reviewability;
- consistency;
- testing;
- versioning;
- auditability.

Potential systems include:

- OPA/Rego;
- Cedar;
- CEL;
- application-native policy;
- another suitable engine.

Do not adopt a policy language merely because the platform supports one.

Policy must remain understandable enough to review.

---

## Policy inputs

Authorization policy should consume trusted inputs.

Classify policy inputs by source.

Examples:

```text
trusted:
    validated identity claims
    service-owned resource metadata
    platform-provided workload identity

untrusted:
    request headers
    body fields
    caller-supplied role names
```

Do not allow untrusted input to assert trusted authorization attributes.

---

## Context-aware authorization

Where justified, authorization may consider context such as:

- device state;
- network zone;
- authentication strength;
- time;
- environment;
- resource sensitivity;
- operation risk.

Context can strengthen authorization.

Avoid policies that become impossible to reason about because they combine too
many volatile signals.

---

## Continuous verification

"Continuous" does not mean running full authentication from scratch on every
CPU instruction.

It means trust is not granted permanently because one check succeeded in the
past.

Use mechanisms appropriate to the system, such as:

- short token lifetimes;
- session expiry;
- policy re-evaluation on protected operations;
- revocation;
- credential rotation;
- device/workload state refresh;
- connection re-establishment.

---

## Revocation

Define how trust can be removed.

Examples:

- disable identity;
- revoke session;
- rotate credential;
- revoke certificate;
- change authorization policy;
- remove workload permission;
- deny artifact;
- quarantine workload.

Where immediate revocation is not technically supported, document the maximum
remaining credential/session lifetime.

---

## Failure behavior

Identity and policy dependencies are security-critical dependencies.

Define behavior for:

- identity provider unavailable;
- JWKS/key endpoint unavailable;
- policy engine unavailable;
- certificate rotation failure;
- clock skew;
- stale cached policy;
- revoked identity;
- partial network partition.

Do not improvise fallback to unauthenticated or unauthorized operation.

Caching identity keys or policy may be appropriate, but expiration and stale
behavior must be explicit.

---

## Availability versus fail-closed behavior

Security and availability can conflict.

Determine which operations may continue safely during identity/policy
dependency failure.

Examples:

- public health endpoint may remain available;
- cached read-only data may be safe;
- privileged mutation may require fail-closed behavior.

Do not use one global fallback for every endpoint.

Document intentional degraded modes.

---

## Observability

Zero-trust controls must be observable.

Record, where appropriate:

- authentication success/failure;
- authorization allow/deny;
- policy errors;
- invalid tokens;
- expired credentials;
- identity-provider failures;
- certificate errors;
- break-glass usage;
- privilege elevation;
- suspicious cross-tenant access attempts.

Do not log:

- raw tokens;
- passwords;
- private keys;
- session secrets;
- unnecessary sensitive claims.

---

## Audit events

High-impact actions should produce attributable audit records.

Examples:

- privilege changes;
- policy changes;
- secret access;
- administrative operations;
- tenant-boundary changes;
- destructive actions;
- break-glass access.

An audit record should capture enough context to answer:

- who/what acted;
- what action occurred;
- on which resource;
- when;
- whether it succeeded;
- relevant policy/authorization context.

---

## Correlation

Use correlation identifiers where useful to follow an operation across trust
boundaries.

Correlation identifiers are diagnostic context.

They are not authentication credentials.

Do not grant permission based on possession of a request ID or trace ID.

---

## Rate and abuse controls

Identity does not eliminate abuse.

Authenticated callers can still:

- overload the service;
- enumerate resources;
- abuse expensive operations;
- perform credential-stuffing attacks;
- exploit authorization gaps.

Where risk justifies it, apply:

- per-identity limits;
- per-tenant limits;
- operation-specific limits;
- concurrency limits;
- quotas.

Do not rely solely on source IP for identity-aware rate controls.

---

## Sensitive data

Classify sensitive data where the system needs differentiated handling.

Consider:

- access policy;
- logging;
- transport;
- persistence;
- cache behavior;
- retention;
- replication.

Authorization should follow the resource sensitivity model where applicable.

Do not send sensitive data to downstream systems solely because they are
"internal".

---

## Encryption in transit

Use authenticated encryption for traffic crossing relevant trust boundaries.

TLS is the normal baseline for HTTP-style network services beyond controlled
local development.

For internal service traffic, use platform-appropriate secure transport where
the threat model requires it.

Do not disable certificate verification for convenience.

---

## Encryption at rest

Use platform-provided encryption at rest where appropriate.

Application-level encryption may be justified for especially sensitive fields
or trust boundaries.

Do not add bespoke field encryption unless key management and operational
recovery are also designed.

Encryption without key-management discipline can make systems less
recoverable without materially improving security.

---

## Environment boundaries

Development, staging, and production should not implicitly trust one another.

Separate, where appropriate:

- identities;
- credentials;
- databases;
- trust roots;
- deployment permissions;
- network access;
- administrative roles.

A compromised development environment should not automatically grant
production access.

---

## Non-production data

Do not copy production secrets or sensitive customer data into lower
environments merely for convenience.

Use:

- synthetic data;
- redacted data;
- minimized datasets;
- dedicated test identities.

If production-like data is required, apply equivalent access controls.

---

## CI/CD identity

Build and deployment automation are identities too.

Apply zero-trust principles to them.

Separate:

- test identity;
- artifact publisher identity;
- signer identity;
- deployment identity;
- production promotion identity.

Do not give a pull-request workflow production deployment authority.

Prefer short-lived federated CI credentials where available.

---

## Artifact trust

A valid workload identity does not make an artifact trustworthy.

Compose this profile with `software-supply-chain.md` where artifact integrity
matters.

Runtime identity and software provenance solve different trust problems.

For example:

```text
workload identity proves:
    what is running / who is calling

artifact provenance proves:
    where the software came from
```

Both may be required.

---

## Deployment policy

Where applicable, require workloads to satisfy deployment security controls
such as:

- approved artifact source;
- immutable artifact reference;
- signed/verified artifact;
- least-privilege runtime identity;
- non-root execution;
- constrained privileges;
- permitted network access.

Do not create policy requirements that the platform cannot currently satisfy.

Roll out enforcement progressively.

---

## Public endpoints

Some endpoints may intentionally allow anonymous access.

Examples:

- public content;
- health probes;
- login initiation;
- webhook endpoints authenticated by signature rather than user login.

Anonymous does not mean unprotected.

Public endpoints still require:

- input validation;
- resource limits;
- abuse controls where appropriate;
- explicit authorization semantics;
- safe error handling.

Document intentional anonymous access.

---

## Health endpoints

Health probes may need different access policy than application APIs.

Do not expose:

- secrets;
- identity configuration;
- internal topology;
- stack traces;
- detailed dependency credentials/status.

If probes require authentication, ensure the orchestrator can supply it
reliably.

Do not make operational recovery impossible by over-securing basic liveness
signals without need.

---

## Webhooks

Webhook producers are external identities.

Verify them through the provider's supported mechanism.

Possible controls:

- request signature;
- shared-secret MAC;
- mTLS;
- source identity;
- timestamp validation;
- replay protection.

Do not trust a webhook because it arrives from an expected IP range alone.

---

## Message brokers

For asynchronous systems:

- authenticate producers and consumers;
- authorize topic/queue access;
- avoid broad wildcard subscriptions;
- scope identities by function;
- validate message content;
- preserve tenant/security context intentionally.

Do not assume a message is trusted merely because it came through an internal
broker.

---

## Background jobs

Background workers should use dedicated identities.

Do not reuse front-end service credentials automatically.

Scope worker permissions to required operations.

If jobs act on behalf of users, distinguish:

- workload identity;
- delegated user authority;
- system authority.

Do not silently transform a user request into unrestricted system privilege.

---

## Caches

Caches cross trust boundaries when they contain identity-dependent data.

Ensure cache keys include relevant security context.

Consider:

- tenant;
- user/subject;
- authorization scope;
- resource version.

Do not serve cached privileged content to another identity because a cache key
ignored authorization context.

---

## Proxies and gateways

Reverse proxies and gateways may establish or validate identity.

If the application trusts identity headers from a proxy:

- ensure direct bypass is impossible or controlled;
- strip untrusted client-supplied copies;
- authenticate the proxy-to-service channel;
- define exactly which headers are trusted.

Do not blindly trust headers such as:

```text
X-User
X-Role
X-Forwarded-User
```

when callers can reach the service directly.

---

## Sidecars and local agents

A local sidecar is not automatically trusted because it shares a pod, host, or
loopback interface.

Define:

- its identity;
- its permissions;
- its control API exposure;
- trust of localhost traffic;
- secret access.

Avoid unauthenticated privileged admin endpoints on loopback when other local
processes may be compromised.

---

## Metadata services

Cloud/container metadata endpoints can expose powerful credentials.

Protect them through platform controls and SSRF defenses.

Do not assume metadata endpoints are unreachable from application code.

Use newer hardened metadata protocols where the platform provides them.

---

## DNS

DNS answers are not identity proof.

Do not authorize solely because a hostname resolved to an expected address.

Use authenticated protocols and identity verification.

Consider DNS rebinding in SSRF-sensitive code.

---

## Error handling

Do not expose security-sensitive distinctions unnecessarily.

For example, authentication errors should not reveal more account information
than required.

Internally, preserve enough detail for diagnosis.

Externally, return consistent and safe error semantics.

Avoid leaking:

- policy internals;
- token contents;
- secret identifiers;
- infrastructure topology.

---

## Policy testing

Authorization policy requires automated tests.

Test at minimum, where applicable:

- explicit allow;
- explicit deny;
- missing identity;
- invalid identity;
- expired identity;
- wrong audience;
- insufficient role/scope;
- cross-tenant access;
- resource ownership mismatch;
- policy backend failure;
- stale/revoked credential;
- administrative boundary.

Policy tests should validate expected behavior, not only parser syntax.

---

## Negative testing

Zero-trust controls are defined by what they reject.

At least one deny/failure path should be exercised end to end.

Examples:

- service with valid identity but insufficient permission is denied;
- workload from unauthorized namespace/account is denied;
- token for wrong audience is rejected;
- invalid certificate is rejected;
- user cannot access another tenant;
- direct gateway bypass is rejected;
- revoked identity no longer succeeds;
- policy-engine failure blocks privileged action;
- untrusted header cannot assert identity.

A system tested only for successful authentication has not demonstrated
zero-trust enforcement.

---

## Adversarial testing

For higher-risk services, consider testing:

- token manipulation;
- claim substitution;
- confused deputy behavior;
- privilege escalation;
- authorization bypass;
- tenant ID manipulation;
- gateway/header spoofing;
- SSRF;
- credential replay;
- stale authorization cache;
- downgrade paths;
- break-glass abuse.

Preserve these as regression tests where practical.

---

## Confused deputy prevention

A service with broader authority than its caller can become a confused deputy.

When a service performs actions on behalf of another principal:

- preserve caller context where needed;
- authorize the delegated action;
- scope downstream credentials;
- distinguish system actions from delegated actions.

Do not use the service's broad privilege to bypass the caller's authorization
constraints.

---

## Replay resistance

Where captured credentials or requests could be replayed harmfully, consider:

- short lifetimes;
- nonces;
- timestamps;
- idempotency keys;
- proof-of-possession;
- one-time tokens.

Do not introduce replay infrastructure when ordinary TLS plus short-lived
credentials is sufficient for the risk.

---

## Credential storage

If credentials must be stored:

- use platform secret storage;
- limit filesystem permissions;
- avoid world-readable env dumps;
- avoid crash-report leakage;
- clear temporary files;
- prefer in-memory handling where practical.

Environment variables are convenient but are not automatically the safest
secret mechanism in every runtime.

Use the platform's strongest practical option.

---

## Credential rotation

Design rotation without downtime where practical.

Support overlapping validity during controlled rotation if the protocol needs
it.

Do not make credential rotation require rebuilding the application artifact.

---

## Identity-provider dependency

Authentication infrastructure is a critical dependency.

Plan for:

- key rotation;
- key-cache behavior;
- temporary network failure;
- issuer migration;
- clock skew;
- malformed discovery metadata.

Do not fetch remote key material on every request when a safe bounded cache is
appropriate.

Do not cache indefinitely.

---

## Policy engine dependency

If authorization depends on an external policy engine:

- define timeout;
- define failure behavior;
- bound retries;
- define cache policy;
- observe latency and failures;
- avoid infinite dependency chains.

A policy engine outage should not accidentally become allow-all.

For non-sensitive public operations, a different degraded behavior may be
intentional and documented.

---

## Trust-domain changes

Changes to the following are security-sensitive architecture changes:

- identity provider;
- trust root;
- certificate authority;
- workload trust domain;
- token audience;
- authorization model;
- policy engine;
- gateway identity contract;
- tenant identity model.

Document consequential changes with an ADR or equivalent record.

Test migrations.

---

## Least-privilege review

Periodically review permissions.

Look for:

- unused roles;
- wildcard scopes;
- stale service accounts;
- orphaned secrets;
- deprecated endpoints;
- broad datastore access;
- environment-crossing permissions.

Do not assume permissions remain minimal because they were minimal at launch.

---

## Access review

Where organizational requirements justify it, support review of who or what
has access to sensitive operations.

Do not invent compliance processes without requirements.

The system should at least make access relationships discoverable.

---

## Policy exceptions

Zero-trust policy exceptions should be:

- explicit;
- scoped;
- attributable;
- time-bounded where practical;
- logged;
- reviewed.

Examples:

- temporary legacy client access;
- emergency administrative route;
- temporary broad egress;
- temporary policy bypass for migration.

Avoid hidden exceptions embedded as hard-coded identities or IP addresses.

---

## Migration from implicit trust

For an existing service, do not attempt a flag-day zero-trust rewrite unless
required.

Prefer staged migration:

```text
1. inventory trust paths
2. establish identity
3. observe
4. add authorization
5. test deny paths
6. enforce
7. remove legacy trust
```

Shadow/audit modes can be useful before enforcement.

Do not leave shadow mode permanently and claim the control is enforced.

---

## Developer experience

Security controls should have a usable local workflow.

Developers should be able to:

- obtain a local/test identity;
- run authorized test calls;
- run denied test calls;
- inspect policy decisions;
- rotate local credentials if applicable;
- execute security regression tests.

Do not require developers to possess production credentials.

Provide safe fixtures or local identity emulation where useful.

---

## Documentation

Document the trust model clearly.

At minimum explain:

- identity providers;
- workload identity;
- authentication flow;
- authorization model;
- trust boundaries;
- policy decision/enforcement points;
- credential lifecycle;
- network controls;
- break-glass path;
- expected failure behavior.

Do not publish secret values or sensitive operational details that create
unnecessary attack surface.

---

## Architecture decision records

Use ADRs for consequential zero-trust decisions such as:

- workload identity technology;
- token/credential model;
- authorization model;
- policy engine;
- service mesh adoption;
- trust-domain design;
- tenant isolation model;
- administrative access pattern.

Do not create ADRs for every individual permission.

---

## Recommended verification interface

A repository applying this profile should expose clear verification commands
through its existing task runner.

Potential commands include:

```text
test
test-authn
test-authz
test-security
test-integration
verify-policy
```

These names are illustrative.

Prefer the repository's established tooling.

---

## Acceptance criteria

A zero-trust service is not complete because it uses mTLS, a VPN, a service
mesh, or an identity provider.

Demonstrate the applicable subset of the following.

### Identity

- protected requests have explicit caller identity;
- service-to-service calls use an intentional workload identity model;
- invalid/expired/wrong-audience credentials are rejected;
- long-lived shared secrets are minimized.

### Authorization

- protected actions require explicit authorization;
- denied actions are tested;
- cross-resource/cross-tenant boundaries are tested where applicable;
- policy failures do not become implicit allows.

### Privilege

- runtime identity is least-privileged;
- datastore access is scoped;
- CI/CD identities are separate from runtime identities;
- administrative privilege is distinct from normal operation.

### Network

- network access is constrained where useful;
- internal network location is not the sole authorization mechanism;
- outbound access is deliberate where risk justifies it.

### Credentials

- secrets are not committed;
- credentials can be rotated/revoked;
- privileged credentials are short-lived where practical;
- trust roots are explicit.

### Failure

- identity-provider failure behavior is defined;
- policy-engine failure behavior is defined;
- revocation behavior is understood;
- break-glass behavior is explicit where present.

### Observability

- authentication failures are observable;
- authorization denies are observable;
- administrative/break-glass actions are auditable;
- sensitive credentials are not logged.

### Testing

- allow path passes;
- deny path passes;
- malformed/invalid identity path passes;
- at least one meaningful security-boundary negative test passes.

---

## Optional composition

This profile is intended to compose with other recipes and profiles.

Common combinations:

```text
secure-service + zero-trust-service
```

For a general production service with explicit identity and authorization
boundaries.

```text
platform-control-plane + zero-trust-service
```

For build/deploy systems with strongly separated automation identities,
promotion authority, and control-plane boundaries.

```text
zero-trust-service + software-supply-chain
```

For both runtime identity and artifact provenance.

```text
zero-trust-service + internet-facing
```

For stronger public-edge abuse controls and exposure hardening.

```text
zero-trust-service + high-assurance
```

For deeper authorization testing, stronger trust isolation, and independent
verification.

```text
agentic-system + zero-trust-service
```

For agents or tool-using services where model/tool authority must be separated
from caller identity and delegated permissions.

---

## Anti-patterns

Do not mistake any of the following for zero trust by themselves:

- VPN;
- private subnet;
- firewall;
- mTLS;
- service mesh;
- SSO;
- OAuth;
- Kubernetes NetworkPolicy;
- an API gateway;
- an identity-aware proxy;
- a secrets manager.

Avoid:

- "internal = trusted";
- shared admin API keys;
- wildcard service roles;
- one identity for every workload;
- authorization based only on IP address;
- identity headers accepted from arbitrary callers;
- trusting all workloads in one namespace;
- production credentials in developer environments;
- long-lived tokens with no revocation strategy;
- service mesh deployment with no authorization model;
- allow-all when the policy engine is unavailable;
- permanent break-glass credentials;
- hidden policy bypasses;
- logging raw tokens;
- using trace IDs as identity;
- granting access because a caller can reach the port.

Zero trust is an authorization and trust architecture, not a network topology.

---

## Completion evidence

When this profile is applied, the final report should state:

1. protected trust boundaries;
2. human identity model;
3. workload identity model;
4. authentication mechanism;
5. authorization model;
6. policy decision point(s);
7. policy enforcement point(s);
8. credential lifecycle;
9. network segmentation/egress controls;
10. datastore privilege model;
11. CI/CD identity separation;
12. administrative/break-glass model;
13. failure behavior for identity/policy dependencies;
14. audit/telemetry controls;
15. security tests executed;
16. deny-path tests executed;
17. commands actually run;
18. observed results;
19. exceptions and unresolved gaps;
20. controls deliberately not introduced and why.

Never claim the service is "zero trust" merely because one identity-aware
technology is installed.

Describe the concrete trust boundaries and enforcement that were actually
implemented and verified.

---

## Guiding principle

Zero trust does not mean trusting nothing.

It means trusting deliberately.

A secure service should not ask:

> Is this request coming from inside?

It should ask:

> Who or what is making this request, what exactly are they allowed to do,
> under what verifiable context, and what stops them when the answer is no?

Trust should be explicit.

Privilege should be narrow.

Identity should be verifiable.

Authorization should be enforced.

Failures should be safe.

Exceptions should be visible.

And network location should never quietly become identity.
