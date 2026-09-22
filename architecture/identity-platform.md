# identity-platform.md

> Recipe for scaffolding or elevating a shared identity platform.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md`, `control-plane.md`,
> `developer-platform.md`, `secure-service.md`, `kubernetes-workload.md`,
> `operator-controller.md`, `library-sdk.md`, and profiles such as
> `zero-trust-service.md`, `software-supply-chain.md`, `high-assurance.md`, or
> `ai-security.md` where appropriate.
>
> This recipe defines architecture expectations for platforms that establish,
> federate, issue, validate, authorize, delegate, rotate, revoke, and audit
> identity for humans, workloads, devices, automation, and platform components.
>
> It does not require a specific identity provider, PKI, service mesh, SPIFFE
> implementation, cloud IAM product, policy engine, directory service, or
> authentication protocol.

## Purpose

Use this recipe when the platform provides shared identity or trust capability
to multiple systems, workloads, users, or administrative domains.

Typical examples:

- enterprise identity platforms;
- workforce identity;
- workload identity;
- machine-to-machine identity;
- service identity;
- federated identity;
- platform IAM;
- certificate/credential issuance;
- identity-aware access infrastructure;
- authorization context platforms;
- credential brokering;
- short-lived token services;
- multi-cloud identity federation;
- developer-platform identity layers;
- AI platform identity and delegated authority.

The goal is not to centralize every access decision into one product.

The goal is to establish a coherent identity architecture where systems can
answer:

- Who or what is this principal?
- Who issued that identity?
- For which audience is it valid?
- What trust domain does it belong to?
- Is it still valid?
- What authority was delegated?
- What resource/action is being requested?
- Which policy applies?
- What evidence supports the decision?
- How is access revoked?

---

## Core principle

Identity is not authorization.

A useful model is:

```text
identity
    !=
authentication
    !=
authorization
    !=
entitlement
    !=
session
```

A secure identity platform should make those boundaries explicit.

A high-level flow is:

```text
Principal
   |
   v
Authentication / Attestation
   |
   v
Identity Assertion
   |
   v
Token / Credential Issuance
   |
   v
Policy / Authorization Context
   |
   v
Enforcement Point
   |
   v
Resource
```

The identity platform establishes trustworthy identity and context.

The resource or policy layer decides whether a specific action is allowed.

---

## Architectural invariants

An identity platform SHOULD satisfy these invariants unless there is a
documented reason not to.

### 1. Principal type is explicit

Distinguish at least where applicable:

- human;
- workload;
- automation;
- device;
- platform component;
- external partner;
- service account.

Do not collapse all principals into generic "users".

### 2. Identity has a trust domain

Every identity should be attributable to a trust domain and issuer.

Do not accept an identity merely because its format looks familiar.

### 3. Issuer and audience are explicit

Credentials should be validated for:

- issuer;
- audience;
- subject;
- expiry;
- signature;
- relevant claims.

Do not accept bearer tokens outside the audience for which they were issued.

### 4. Authentication and authorization remain distinct

A valid identity proves who or what the caller is.

It does not prove the caller may perform a specific action.

### 5. Workload identity is short-lived where practical

Prefer renewable, scoped credentials over long-lived static secrets.

Do not distribute permanent service credentials when federation or workload
identity is available.

### 6. Delegation is explicit

When one principal acts on behalf of another, preserve:

- original subject;
- delegate;
- scope;
- target;
- expiry;
- chain of delegation where relevant.

Do not erase the initiating identity behind a powerful intermediary.

### 7. Trust is not transitive by accident

Trusting an issuer does not mean trusting every downstream system that received
one of its tokens.

Define scope and audience deliberately.

### 8. Revocation behavior is known

Know how long revoked access may remain usable.

Do not claim immediate revocation if credentials are cached until expiry.

### 9. Bootstrap trust is explicit

Root trust, first administrators, signing keys, and initial identity issuance
need a documented bootstrap path.

### 10. Identity lifecycle is auditable

Important lifecycle events should be attributable:

- creation;
- issuance;
- rotation;
- privilege change;
- delegation;
- revocation;
- break-glass use.

### 11. Administrative privilege is separated

Identity-platform administrators should not automatically become application
administrators or data owners.

### 12. Identity compromise has bounded blast radius

Use:

- short lifetimes;
- scoped claims;
- audience restriction;
- tenant separation;
- environment separation;
- narrow credentials.

Do not make one stolen credential universally useful.

---

## Identity taxonomy

Document principal classes.

A useful taxonomy may include:

```text
Human
  |
  +--> employee
  +--> contractor
  +--> administrator
  +--> external partner

Workload
  |
  +--> service
  +--> job
  +--> controller
  +--> agent

Automation
  |
  +--> CI
  +--> deployment system
  +--> bot

Device
  |
  +--> managed workstation
  +--> server/node
  +--> edge device
```

Different principal classes may require different authentication mechanisms.

---

## Human identity

Human identity may originate from:

- enterprise directory;
- workforce IdP;
- external identity federation;
- customer identity provider.

Define:

- source of truth;
- joiner/mover/leaver process;
- MFA policy;
- session model;
- recovery process.

Do not let application teams independently recreate workforce identity unless
there is a concrete reason.

---

## Workload identity

Workload identity should bind identity to an execution context.

Possible evidence sources:

- orchestrator identity;
- cloud workload identity;
- instance metadata;
- TPM/device attestation;
- signed workload certificate;
- SPIFFE-style workload identity;
- service account federation.

The identity platform should validate the evidence before issuing credentials.

---

## Device identity

If devices influence access decisions, define:

- device enrollment;
- ownership;
- attestation;
- posture freshness;
- revocation.

Do not treat a device identifier alone as proof of trust.

---

## Automation identity

CI, deployment systems, bots, and controllers should have dedicated identities.

Do not run automation under personal user credentials.

Separate:

```text
build identity
deployment identity
runtime identity
administrative identity
```

---

## Service accounts

Service accounts are identities, not generic shared secrets.

Use them deliberately.

Avoid:

- one shared service account for many workloads;
- static credentials embedded in repositories;
- indefinite credential lifetime.

---

## Trust domains

Define trust domains.

Examples:

- enterprise;
- production;
- development;
- partner;
- customer;
- cloud account/project;
- cluster;
- region.

Trust domain boundaries should influence:

- issuer trust;
- certificate trust;
- federation;
- authorization.

---

## Trust anchors

Identify root trust anchors.

Examples:

- CA root;
- OIDC issuer;
- signing key;
- hardware root;
- enterprise IdP.

Protect them according to their blast radius.

Do not duplicate root keys casually across environments.

---

## Federation

Federation allows one identity domain to trust another.

Define:

- issuer;
- relying party;
- audience;
- accepted claims;
- mapping;
- trust duration.

Do not federate based solely on email domain or mutable display attributes.

---

## Federation mapping

Map external identities to local subjects explicitly.

Avoid ambiguous mappings.

Prefer stable immutable identifiers.

Do not use mutable usernames as the sole long-term key if the upstream provides
a stable subject identifier.

---

## Identity normalization

If multiple issuers exist, define a canonical subject representation.

Example:

```text
issuer + subject
```

is often safer than:

```text
email
```

Avoid identity collisions across providers.

---

## Authentication protocols

Use established protocols where appropriate:

- OIDC;
- OAuth 2.x;
- SAML;
- mTLS;
- WebAuthn/passkeys;
- SSH certificates;
- signed workload tokens.

Do not invent custom authentication protocols without a compelling need.

---

## OIDC

For OIDC-based identity:

Validate:

- issuer;
- signature;
- audience;
- expiration;
- nonce/state where appropriate;
- authorized party/client where relevant.

Do not use access tokens and ID tokens interchangeably.

---

## OAuth

OAuth is authorization delegation, not a generic identity protocol.

Use scopes and audiences intentionally.

Do not treat any OAuth access token as proof of human identity without
understanding issuer semantics.

---

## SAML

Where SAML is required for enterprise federation:

- validate signature;
- audience;
- recipient;
- expiry;
- assertion replay.

Do not depend on mutable attributes for permanent identity if stable identifiers
exist.

---

## mTLS identity

mTLS can authenticate peers.

It does not by itself answer:

- what action is allowed;
- what resource is targeted;
- which tenant applies.

Separate identity from authorization.

---

## Certificate identity

If certificates carry identity:

- define subject naming;
- issuance authority;
- TTL;
- renewal;
- revocation;
- key protection.

Avoid long-lived leaf certificates where automated rotation is practical.

---

## Workload certificate issuance

A workload certificate should bind to trusted workload evidence.

Do not allow arbitrary workloads to request arbitrary identities.

---

## Token issuance

Token issuance should be explicit and policy-bound.

A token should identify:

- issuer;
- subject;
- audience;
- expiry;
- relevant scopes/claims.

Keep tokens narrowly useful.

---

## Token lifetime

Short-lived tokens reduce compromise duration.

Choose TTL based on:

- revocation needs;
- availability;
- refresh cost;
- runtime constraints.

Do not choose long lifetime solely for operational convenience.

---

## Refresh

Refresh mechanisms should not silently create effectively permanent access.

Protect refresh tokens or equivalent long-lived credentials more strongly than
short-lived access tokens.

---

## Token exchange

Token exchange/delegation can reduce credential sprawl.

Use it to mint narrower credentials.

Do not allow exchange into broader privilege than the original subject is
authorized to obtain.

---

## Audience restriction

A credential intended for service A should not automatically work at service B.

Validate audience at every enforcement point.

This is one of the strongest controls against credential replay across systems.

---

## Scope

Scopes should represent meaningful delegated authority.

Avoid giant catch-all scopes such as:

```text
admin
all
full_access
```

unless genuinely necessary.

---

## Claims

Keep identity claims:

- necessary;
- stable;
- well-defined.

Do not stuff entire authorization policy into identity tokens if it will become
stale or oversized.

---

## Group claims

Group membership can be useful but may create:

- large tokens;
- stale authorization;
- ambiguous nesting.

For complex authorization, use resource-aware policy rather than token groups
alone.

---

## Entitlements

Entitlements answer what access a principal may have.

Do not conflate entitlement storage with authentication.

Entitlements may live in:

- directory;
- policy system;
- application;
- authorization graph.

---

## Role model

Use roles where they fit.

Avoid role explosion.

Prefer roles that represent stable job/function boundaries.

Do not encode every resource instance into a new role.

---

## RBAC

RBAC is useful for coarse-grained permissions.

Document:

- role;
- permissions;
- scope;
- inheritance.

Do not use global roles where resource scope matters.

---

## ABAC

Attribute-based policy can incorporate:

- identity;
- resource;
- environment;
- device;
- risk.

Ensure attributes come from trusted sources.

Do not let requesters self-assert security-relevant attributes.

---

## ReBAC

Relationship-based access can model:

- owner;
- member;
- parent/child;
- delegated administrator.

Define graph authority and consistency.

---

## Capability-based authorization

Capabilities can encode specific delegated authority.

Use when narrow transferability is useful.

Protect capability tokens as bearer authority.

---

## Policy decision and enforcement

A useful model:

```text
PEP -> PDP -> Policy -> Decision
```

Where:

- PEP = policy enforcement point;
- PDP = policy decision point.

The architecture may be centralized or embedded.

Do not require one universal PDP if local enforcement is safer or simpler.

---

## Authorization context

A decision may depend on:

- subject;
- action;
- resource;
- tenant;
- environment;
- device;
- risk;
- time;
- approval.

Keep context explicit.

---

## Resource-aware authorization

Authorization should answer:

> May subject S perform action A on resource R?

Do not stop at:

```text
subject has role developer
```

for sensitive resources.

---

## Application authorization

The identity platform should provide primitives and trusted identity context.

Applications may still own domain-specific authorization.

Do not centralize business-specific authorization blindly into the identity
platform.

---

## Delegated authorization

If a platform performs actions for users:

- preserve user context;
- apply platform policy;
- narrow delegated scope.

Do not let the platform's own broad credential bypass user policy.

---

## Impersonation

Impersonation is high-risk.

If supported:

- explicit permission;
- reason;
- scope;
- audit;
- time limit.

Do not make impersonation a normal troubleshooting mechanism.

---

## Acting-as versus acting-for

Distinguish:

```text
acting as subject
```

from:

```text
acting for subject using delegated authority
```

Prefer delegated authority where possible because it preserves system identity.

---

## Session model

For humans, define:

- login;
- session duration;
- idle timeout;
- refresh;
- MFA step-up;
- logout;
- revocation.

Do not rely on indefinite browser sessions.

---

## Step-up authentication

Use stronger authentication for high-risk operations where appropriate.

Examples:

- production admin;
- secret export;
- root policy change;
- break-glass.

---

## MFA

MFA should be phishing-resistant where risk justifies it.

Do not treat SMS or email as equivalent to stronger authenticators in
high-assurance contexts.

---

## Recovery

Account recovery can become a bypass path.

Apply controls comparable to the authentication strength being recovered.

---

## Device posture

Device posture may be useful context.

Examples:

- managed device;
- disk encryption;
- patch state;
- device certificate.

Treat posture as contextual evidence, not infallible truth.

---

## Risk-based access

Risk signals may influence:

- step-up;
- session restriction;
- denial.

Do not make opaque probabilistic risk scores the sole authorization basis for
critical operations without deterministic guardrails.

---

## Revocation

Define what can be revoked:

- session;
- refresh token;
- signing key;
- certificate;
- workload identity;
- group/role;
- delegated grant.

---

## Revocation latency

Document realistic revocation latency.

For short-lived tokens without introspection:

```text
maximum effective delay ~= token TTL
```

unless another revocation mechanism exists.

Do not claim "instant revocation" inaccurately.

---

## Token introspection

Introspection can support central revocation but adds runtime dependency.

Decide whether the availability/security tradeoff is worth it.

---

## Cached authorization

Caching authorization decisions can reduce latency.

Bound:

- TTL;
- scope;
- invalidation.

Do not cache high-risk grants indefinitely.

---

## Key rotation

Signing and encryption keys need lifecycle management.

Define:

- generation;
- activation;
- overlap;
- retirement;
- emergency rotation.

---

## JWKS / key discovery

If using discoverable signing keys:

- cache safely;
- honor rotation;
- validate issuer;
- handle key rollover.

Do not trust arbitrary remote key URLs from untrusted token content.

---

## Certificate rotation

Automate leaf certificate rotation where practical.

Avoid outages caused by manual expiry.

Monitor upcoming expiration.

---

## Root rotation

Root trust rotation is more complex.

Plan:

- overlap;
- bundle update;
- phased rollout;
- rollback.

Do not assume root replacement is a normal leaf-certificate rotation.

---

## PKI

If the platform owns PKI, define:

- root CA;
- intermediate CAs;
- issuance;
- revocation;
- TTL;
- audit;
- key protection.

Keep root keys offline or strongly protected where risk justifies it.

---

## Environment separation

Separate trust between:

- development;
- test;
- staging;
- production.

Avoid one signing root or token issuer with unrestricted cross-environment trust
unless that is a deliberate architecture.

---

## Tenant isolation

For multi-tenant identity:

- tenant membership;
- issuer/subject mapping;
- policy;
- admin delegation;
- logs;
- token audience

must preserve tenant boundaries.

Do not trust tenant ID claims supplied by clients without authoritative binding.

---

## Cross-tenant access

Cross-tenant access should require explicit grants.

Do not infer access from shared organization membership alone.

---

## B2B / partner federation

For partner identity:

- isolate partner trust;
- scope audiences;
- validate mapping;
- define offboarding.

One compromised partner should not automatically gain enterprise-wide access.

---

## Customer identity

If the platform handles customer identity, separate it from workforce identity
unless the architecture explicitly requires convergence.

Different lifecycle and privacy requirements often apply.

---

## Admin identities

Use dedicated administrative roles or identities for sensitive platform
operations.

Avoid everyday browsing/development sessions carrying global admin authority.

---

## Privileged access management

For highly privileged actions consider:

- JIT elevation;
- approval;
- session recording;
- short TTL;
- separate admin identity.

Do not make permanent global administrator assignment the default.

---

## Just-in-time access

JIT access should have:

- requested scope;
- reason;
- approver/policy;
- duration;
- automatic expiry.

---

## Break-glass

Break-glass access should:

- bypass only what is necessary;
- be highly visible;
- produce audit;
- expire automatically.

Test it before incidents.

---

## Identity proofing

If the platform establishes real-world identity, document proofing level and
limitations.

Do not claim strong proofing based only on self-registration.

---

## Enrollment

Enrollment should bind a principal to trusted evidence.

For workloads, this may be:

- orchestrator metadata;
- instance identity;
- device attestation.

For humans, enterprise HR/directory processes may be authoritative.

---

## Joiner/mover/leaver

Human identity lifecycle should support:

```text
join
change role/team
leave
```

Access should follow authoritative lifecycle changes.

Avoid manual access accumulation.

---

## Orphan identities

Detect identities with no valid owner.

Examples:

- inactive service accounts;
- abandoned bots;
- deleted workloads;
- former employees.

Remove or quarantine them.

---

## Service-account ownership

Every non-human identity should have:

- owner;
- purpose;
- scope;
- lifecycle.

Do not create anonymous shared credentials.

---

## Credential inventory

Maintain inventory of privileged credentials where practical.

Know:

- issuer;
- owner;
- scope;
- expiration;
- rotation mechanism.

---

## Secret-backed identity

Where static secrets are unavoidable:

- scope narrowly;
- rotate;
- store securely;
- avoid reuse.

Treat them as technical debt if a stronger identity mechanism is available.

---

## Workload attestation

Workload identity should be based on something harder to forge than self-
declared names.

Potential signals:

- scheduler-issued identity;
- node attestation;
- cloud instance identity;
- signed workload metadata.

---

## Node identity

If nodes participate in workload attestation, protect node enrollment and
rotation.

A compromised node may affect all workloads it can attest.

---

## Device attestation

Where hardware-backed attestation is required, define:

- trust root;
- freshness;
- acceptable measurements;
- revocation.

Do not assume hardware attestation proves application-level integrity by
itself.

---

## Credential brokering

A broker can exchange trusted identity for narrow downstream credentials.

Useful flow:

```text
workload identity
    ->
broker
    ->
policy
    ->
short-lived provider credential
```

This reduces static secret distribution.

---

## Cloud federation

Prefer federation into cloud roles/accounts over distributing long-lived cloud
keys where supported.

Map identities to narrow cloud permissions.

---

## Database identity

Where databases support federated/workload identity, use it when appropriate.

Otherwise use short-lived database credentials if available.

Avoid one static database password shared by an application fleet.

---

## Secret-manager access

Use workload identity to access secret management systems.

The secret platform should still enforce resource-level authorization.

---

## SSH access

If SSH is part of the platform, prefer short-lived SSH certificates or
centrally managed identity over static shared keys.

Avoid permanent shared admin keys.

---

## API keys

API keys may remain appropriate for some external integrations.

Treat them as bearer secrets.

Scope, rotate, and audit them.

Do not call an API key "identity" without clarifying its owner and lifecycle.

---

## Machine identity for CI/CD

CI identities should be separate by trust class.

Examples:

```text
untrusted PR
trusted branch build
release publisher
deployment reconciler
```

Do not give PR jobs release/deployment identity.

---

## Source identity

If build/deploy policy depends on source identity, preserve:

- repository;
- revision;
- workflow identity;
- triggering actor.

Compose with `software-supply-chain.md`.

---

## Human-to-workload delegation

If a human initiates automation that later acts asynchronously, preserve a
durable delegated authorization model.

Do not simply copy the user's long-lived token into a queue.

---

## Delegation token design

A delegated token/grant should bind:

- original actor;
- delegate;
- allowed action;
- resource;
- audience;
- expiry.

---

## Chained delegation

If delegation can chain, bound depth.

Preserve provenance.

Avoid unbounded confused-deputy paths.

---

## Confused deputy

A privileged service may be tricked into using its authority for an
unauthorized caller.

Prevent by checking:

- caller;
- target;
- delegated scope;
- resource ownership.

---

## Token forwarding

Avoid forwarding the same bearer token through many services.

Prefer:

- audience-specific exchange;
- downstream delegated credentials;
- workload identity plus user context.

---

## On-behalf-of flows

Use explicit on-behalf-of patterns where needed.

Do not invent ad hoc headers containing usernames and call that delegation.

---

## Identity propagation

If identity context crosses services, preserve it through trusted mechanisms.

Do not trust caller-supplied headers such as:

```text
X-User
X-Role
X-Tenant
```

unless a trusted gateway strips/recreates them and the service validates the
boundary.

---

## Proxy identity

Reverse proxies/gateways may authenticate users.

Backend services should know which headers/assertions are trusted and from whom.

---

## Service mesh identity

A mesh can provide workload identity and mTLS.

It does not automatically solve:

- user identity;
- resource authorization;
- delegation;
- tenant boundaries.

---

## Zero-trust composition

Compose with `zero-trust-service.md`.

Key principles:

- network location is not identity;
- identity is continuously validated according to context;
- access is least-privileged;
- trust is scoped;
- breach is assumed.

---

## Policy platform integration

Identity claims should be inputs to policy, not the entire policy.

A policy decision may combine:

- identity;
- resource;
- environment;
- ownership;
- approval;
- device;
- risk.

---

## Identity and policy versioning

Where decisions are audited, capture relevant:

- identity issuer/version;
- policy version;
- credential ID;
- authorization decision.

---

## Session/token audit

Audit:

- login;
- token issuance;
- privilege elevation;
- delegation;
- revocation;
- administrative changes.

Avoid logging raw token values.

---

## Audit subject identity

Use stable subject identifiers.

Display names can change.

---

## Privacy

Identity data is sensitive.

Minimize:

- unnecessary profile claims;
- broad directory replication;
- long-lived audit payloads;
- copies of personal attributes.

---

## Data minimization

Only propagate claims that downstream systems need.

Do not send full identity profiles to every service.

---

## Pseudonymous identity

For some cross-domain use cases, pseudonymous identifiers may reduce privacy
exposure.

Do not break required auditability or authorization semantics.

---

## Attribute freshness

Claims such as group membership or device state may become stale.

Define freshness and revalidation.

Do not put long-lived dynamic entitlements into long-lived tokens.

---

## Directory synchronization

If synchronizing directories:

- define authoritative source;
- direction;
- conflict behavior;
- deletion/offboarding.

Avoid uncontrolled bidirectional sync.

---

## Identity cache

Caches should have explicit TTL and invalidation.

Do not let stale identity data preserve revoked access indefinitely.

---

## Availability

Identity availability can be highly critical.

Separate:

```text
new authentication
token issuance
token validation
authorization lookup
directory administration
```

Different functions may require different availability.

---

## Offline validation

Signed tokens/certificates can support offline validation.

Tradeoff:

- better availability;
- slower revocation.

Document that tradeoff.

---

## Online validation

Online introspection can support revocation and dynamic policy.

Tradeoff:

- runtime dependency;
- latency;
- potential outage blast radius.

---

## Last-known-good trust

For some machine identity, short-lived cached trust material can allow
temporary operation during control-plane outages.

Define expiry and fail behavior.

---

## Failure modes

Document behavior when:

- issuer unavailable;
- JWKS unavailable;
- policy service unavailable;
- directory unavailable;
- revocation backend unavailable;
- certificate authority unavailable.

---

## Fail-open versus fail-closed

Choose per function.

Examples:

- new privileged login: often fail closed;
- existing short-lived workload certificate validation: may continue offline;
- directory profile enrichment: may degrade.

Do not use one global rule.

---

## Clock dependency

Identity protocols depend on time.

Account for:

- clock skew;
- certificate not-before/not-after;
- token expiry.

Monitor time synchronization.

---

## Replay protection

Protect replay-sensitive flows.

Examples:

- authorization codes;
- SAML assertions;
- signed requests;
- one-time credentials.

Use nonces/state/expiry where appropriate.

---

## Nonce/state

For browser and interactive flows, validate anti-replay and anti-CSRF values.

Do not omit protocol-required state validation.

---

## PKCE

Use PKCE for public OAuth clients where appropriate.

---

## Redirect URI security

Use exact registered redirect URIs where possible.

Avoid wildcard redirect URI patterns.

---

## Native applications

For CLI/native apps, use standard browser/device flows.

Do not embed client secrets in distributed binaries and pretend they remain
secret.

---

## Device authorization

Device-code flows can be useful for headless systems.

Protect against phishing and incorrect audience.

---

## API client credentials

Machine clients using client credentials should receive narrowly scoped tokens.

Do not give one integration account enterprise-wide privilege by default.

---

## Mutual authentication

For high-trust machine interactions, mutual authentication may be useful.

Do not confuse peer authentication with application authorization.

---

## Key storage

Private keys should be stored using appropriate protection:

- HSM;
- KMS;
- TPM;
- OS key store;
- restricted secret store.

Use according to blast radius.

---

## HSM/KMS

Use hardware-backed or managed key protection for high-value signing keys where
risk justifies it.

Do not require HSMs for every low-risk development key.

---

## Signing algorithms

Use established modern algorithms.

Avoid legacy/weak algorithms.

Do not allow algorithm selection to be controlled by untrusted token headers
without policy.

---

## Algorithm confusion

Token validators should explicitly allow accepted algorithms.

Do not accept `none` or arbitrary algorithm substitution.

---

## Key identifiers

`kid` identifies a key candidate.

It is not trust by itself.

Resolve keys only within the configured trusted issuer/key set.

---

## Token size

Bound token size.

Large claim sets create:

- bandwidth;
- header limits;
- parsing cost.

Avoid putting entire authorization graphs into JWTs.

---

## JWT caveats

JWTs are convenient but not automatically secure.

Validate:

- issuer;
- audience;
- expiry;
- signature;
- expected token type.

Do not decode-and-trust.

---

## Opaque tokens

Opaque tokens can support central introspection/revocation.

Use where runtime dependency is acceptable.

---

## Token binding

Where supported and risk warrants, bind tokens to:

- client certificate;
- key proof;
- device.

This reduces bearer-token theft value.

---

## Proof-of-possession

Proof-of-possession mechanisms can strengthen high-value flows.

Do not add complexity without threat-model justification.

---

## Authorization code flow

For browser apps, use established secure flows.

Avoid implicit legacy patterns where stronger modern flows are available.

---

## Session cookies

If identity platform components issue cookies:

- Secure;
- HttpOnly;
- SameSite as appropriate;
- scoped domain/path;
- rotation.

Do not store raw long-lived credentials in browser-readable storage without
need.

---

## CSRF

Protect browser-authenticated state-changing operations against CSRF.

---

## CORS

CORS is browser policy, not authentication.

Configure narrowly where needed.

---

## Admin plane

Identity administration is high privilege.

Separate:

- configuration;
- issuer management;
- key management;
- federation trust;
- global role changes;
- emergency revocation.

Use stronger access controls.

---

## Privilege separation

Consider separate roles for:

- federation admin;
- key admin;
- access admin;
- auditor;
- operator.

Avoid one universal identity-platform superuser.

---

## Four-eyes control

For high-assurance environments, some operations may require dual control.

Examples:

- root-key rotation;
- federation trust addition;
- global policy override.

Use only where risk justifies it.

---

## Root administrator lifecycle

Root admin accounts should be:

- rare;
- protected;
- monitored;
- recoverable;
- excluded from ordinary use.

---

## Emergency revocation

Provide a way to revoke:

- signing keys;
- compromised clients;
- workload identity issuers;
- sessions.

Test emergency procedures.

---

## Kill switch

For severe compromise, the platform may need to disable an issuer/client/trust
domain rapidly.

Understand blast radius before use.

---

## Multi-region identity

If multi-region:

- define issuer topology;
- key replication;
- token validation;
- session state;
- failover.

Avoid two independent issuers accidentally minting conflicting identities
without shared semantics.

---

## Regional trust

Data residency may affect:

- identity attributes;
- audit;
- directory data.

Document where sensitive identity data is replicated.

---

## Disaster recovery

Recover:

- issuer configuration;
- trust anchors;
- signing keys;
- federation mappings;
- critical policy;
- revocation state where necessary.

Do not assume restoring databases is enough if keys/trust roots are lost.

---

## Backup

Back up critical configuration and keys according to security policy.

Root/private key backup requires particularly strong controls.

---

## Rebuildability

Some identity components can be rebuilt from durable configuration.

Prefer rebuildable stateless services.

---

## Upgrade

Identity upgrades must preserve:

- issuer continuity;
- signing-key continuity;
- token validation;
- federation contracts;
- session behavior.

Avoid breaking all clients during upgrade.

---

## Key overlap during upgrade

Support overlapping old/new signing keys during rotation.

Consumers must refresh trusted key sets safely.

---

## Protocol compatibility

Maintain compatibility across:

- OIDC metadata;
- token claims;
- scopes;
- audiences;
- certificate profiles.

Do not silently reinterpret claims.

---

## Client registration lifecycle

Clients should have:

- owner;
- redirect URIs;
- scopes;
- credentials;
- environment;
- lifecycle.

Detect abandoned clients.

---

## Dynamic client registration

If supported, constrain it.

Do not allow arbitrary clients to register privileged redirect URIs/scopes.

---

## Scope governance

New scopes should have:

- clear semantics;
- owner;
- audience;
- privilege level.

Avoid uncontrolled scope proliferation.

---

## Group/role governance

Roles and groups should have lifecycle and ownership.

Remove obsolete entitlements.

---

## Entitlement review

For privileged access, periodic review may be useful.

Automate evidence where practical.

Do not create review theater disconnected from actual access.

---

## Access certification

If required, certify real entitlements from authoritative sources.

Avoid spreadsheet-only access reviews where platform data can provide stronger
evidence.

---

## Least privilege

Default identities to no access.

Grant only necessary permissions.

Review broad permissions.

---

## Separation of duties

Separate conflicting capabilities where risk requires it.

Examples:

- code author vs release approver;
- identity admin vs auditor;
- platform operator vs key custodian.

---

## Human approval

For sensitive delegated access, approval should be:

- scoped;
- time-bound;
- attributable.

Do not grant permanent entitlement to avoid repeated approval.

---

## Policy exceptions

Exceptions should:

- identify subject/resource;
- document rationale;
- expire;
- be reviewed.

---

## External identity provider trust

Treat external IdPs as independent trust domains.

A compromise there can affect every federated relying party.

Limit scope and audience.

---

## Social identity

If consumer identity uses social providers, define the trust and account-
recovery implications.

Do not use social identity for privileged workforce administration unless
explicitly justified.

---

## Identity proof and authorization proof

Identity evidence should not be reused as proof of resource ownership unless
the architecture explicitly binds them.

---

## Audit

Identity audit events may include:

- authentication;
- token issuance;
- token exchange;
- privilege grant;
- privilege revoke;
- admin config change;
- key rotation;
- federation change;
- break-glass.

---

## Audit integrity

For high-assurance systems, protect audit from tampering by identity
administrators.

---

## Audit privacy

Audit logs contain sensitive identity data.

Apply:

- access control;
- retention;
- minimization.

---

## Observability

Track platform health such as:

- authentication success/failure;
- token issuance latency;
- federation failures;
- certificate issuance/renewal;
- revocation latency;
- policy decision latency;
- key-rotation health.

Avoid exposing sensitive claim values in metrics.

---

## Security telemetry

Useful signals may include:

- repeated failed auth;
- invalid issuer;
- audience mismatch;
- expired token;
- replay attempts;
- unexpected federation mapping;
- privileged role changes.

---

## SLOs

Potential identity-platform SLOs:

- token issuance availability;
- authentication availability;
- certificate renewal success;
- authorization decision latency.

Do not invent numbers without requirements.

---

## Rate limiting

Protect:

- login;
- token issuance;
- introspection;
- password recovery;
- device authorization.

Use rate limits appropriate to abuse risk.

---

## Credential stuffing

For password-based auth, defend against credential stuffing.

Prefer stronger factors and breached-password protections where appropriate.

---

## Enumeration

Avoid leaking whether sensitive accounts exist through inconsistent errors.

Balance this against operational usability.

---

## Brute force

Use:

- rate limits;
- lockout/risk controls;
- MFA;
- detection.

Do not create denial-of-service through overly aggressive permanent lockouts.

---

## Password policy

Where passwords remain necessary, prefer modern guidance:

- length;
- breached-password screening;
- avoid arbitrary frequent rotation.

Do not require complex composition rules without evidence.

---

## Passwordless

Use passkeys/WebAuthn where suitable.

Do not call passwordless automatically phishing-proof unless the deployed
authenticator flow actually provides that property.

---

## Secret questions

Avoid knowledge-based recovery questions.

They are weak authentication factors.

---

## API design

Identity platform APIs should expose stable domain semantics.

Examples:

- issue credential;
- exchange token;
- revoke session;
- evaluate access;
- register client;
- inspect identity.

Avoid exposing internal database schema.

---

## Error model

Use machine-usable errors.

Examples:

```text
INVALID_CREDENTIAL
TOKEN_EXPIRED
TOKEN_REVOKED
AUDIENCE_MISMATCH
ISSUER_UNTRUSTED
AUTHENTICATION_REQUIRED
AUTHORIZATION_DENIED
MFA_REQUIRED
CLIENT_DISABLED
RATE_LIMITED
DEPENDENCY_UNAVAILABLE
```

Do not force clients to parse prose.

---

## Sensitive error handling

Errors should help operators without leaking:

- tokens;
- private keys;
- password material;
- excessive directory information.

---

## SDKs

If identity SDKs are provided, compose with `library-sdk.md`.

SDK defaults should:

- validate TLS;
- validate issuer/audience;
- set finite timeouts;
- avoid logging credentials.

---

## CLI

If identity admin/user CLI exists, compose with `cli.md`.

Avoid passing passwords/tokens as process arguments by default.

---

## Developer platform integration

For internal developer platforms, identity should support:

- human login;
- workload identity;
- build identity;
- deploy identity;
- production admin.

Do not reuse one identity for all stages.

---

## Kubernetes integration

For Kubernetes, possible patterns include:

- service accounts;
- projected tokens;
- workload identity federation;
- certificate issuance;
- OIDC auth.

Compose with `kubernetes-workload.md`.

---

## Kubernetes RBAC

Cluster RBAC may use identity-platform assertions.

Do not assume Kubernetes RBAC replaces application authorization.

---

## Operator/controller identity

Controllers should use dedicated service accounts and external credentials.

Compose with `operator-controller.md`.

---

## AI platform identity

AI systems may introduce:

- agent identity;
- tool identity;
- user delegation;
- provider identity.

Do not let an agent inherit unrestricted platform credentials.

Compose with `agentic-system.md` and `ai-security.md`.

---

## Agent delegated authority

An agent acting for a user should receive narrower authority than the hosting
platform.

Preserve user and agent identities separately.

---

## Tool identity

Tools/MCP servers should authenticate the caller and apply resource-aware
authorization.

Tool discovery does not imply permission.

---

## Data access identity

For RAG/data access, authorization should happen before retrieval/disclosure.

Do not rely on the model to enforce document access.

---

## Supply chain

Identity-platform software is privileged.

Compose with `software-supply-chain.md`.

Protect:

- issuer code;
- signing libraries;
- build pipeline;
- dependencies;
- key-management integration;
- policy bundles.

---

## Third-party libraries

Authentication/crypto libraries are security-critical dependencies.

Pin and review them appropriately.

Avoid reimplementing standards from scratch.

---

## Security review triggers

Require stronger review for changes to:

- signing;
- token validation;
- issuer trust;
- federation;
- audience logic;
- admin roles;
- delegation;
- revocation;
- key rotation;
- recovery;
- cross-tenant mapping.

---

## Threat model

At minimum consider:

- stolen bearer token;
- malicious client;
- compromised workload;
- compromised node;
- compromised IdP;
- key theft;
- replay;
- confused deputy;
- cross-tenant mapping error;
- stale entitlement;
- federation misconfiguration;
- admin compromise;
- recovery-flow bypass.

---

## Testing strategy

### Unit tests

Test:

- claim validation;
- audience checks;
- scope mapping;
- policy logic;
- revocation logic;
- error classification.

### Protocol tests

Test OIDC/OAuth/SAML/mTLS behavior as applicable.

### Integration tests

Exercise real issuer/validator interactions.

### Security tests

Test:

- invalid signature;
- wrong issuer;
- wrong audience;
- expired token;
- replay;
- revoked credential;
- cross-tenant access;
- privilege escalation.

### Rotation tests

Test signing/certificate key rollover.

### Recovery tests

Test issuer/key/config restoration.

---

## Token validation tests

Include:

- valid token;
- expired token;
- future not-before;
- wrong issuer;
- wrong audience;
- wrong algorithm;
- unknown key ID;
- malformed token.

---

## Delegation tests

Verify:

- delegated scope cannot exceed original authority;
- target/resource is bound;
- expiry works;
- actor chain is preserved.

---

## Revocation tests

Measure actual revocation behavior.

Do not assert instant revocation if cached/offline validation delays it.

---

## Federation tests

Test:

- valid mapped identity;
- unknown issuer;
- mapping collision;
- removed partner;
- stale metadata/key.

---

## Certificate tests

Test:

- issuance;
- renewal;
- expiry;
- trust chain;
- revoked/untrusted issuer.

---

## Break-glass tests

Verify emergency access path works and generates audit.

Do not wait for an incident to discover it is broken.

---

## Failure tests

Simulate:

- IdP unavailable;
- key discovery unavailable;
- directory unavailable;
- policy engine unavailable;
- CA unavailable.

Confirm documented degraded behavior.

---

## Cross-environment tests

Verify development identities cannot access production unless explicitly
allowed.

---

## Cross-tenant tests

Attempt access across tenants.

Expect deterministic denial.

---

## Fuzzing

Consider fuzzing:

- token parsers;
- certificate parsers;
- SAML/OIDC metadata parsing;
- authorization expression parsing.

---

## Property-based testing

Useful properties include:

- wrong audience is always denied;
- expired credential is never accepted;
- delegation cannot increase authority;
- tenant identity never changes through untrusted claims.

---

## Acceptance path

A useful end-to-end path:

```text
principal authenticates
    ->
identity platform validates evidence
    ->
scoped credential issued
    ->
service validates issuer/audience
    ->
authorization evaluates subject/action/resource
    ->
request allowed
    ->
audit records subject and decision
```

Also test:

```text
valid identity
    +
wrong resource/tenant
    ->
authorization denied
```

And:

```text
credential revoked/expired
    ->
access denied according to documented revocation semantics
```

---

## Documentation

The platform should document:

- principal types;
- trust domains;
- issuers;
- audiences;
- token/certificate profiles;
- federation;
- authorization integration;
- delegation;
- revocation;
- key rotation;
- admin model;
- recovery.

---

## Architecture diagrams

Useful views include:

### Trust domains

```text
Workforce IdP
    |
    v
Identity Platform
    |
    +--> internal services
    +--> cloud providers
    +--> Kubernetes
    +--> SaaS
```

### Workload identity

```text
Workload
   |
   v
Attestation
   |
   v
Credential Issuer
   |
   v
Scoped Credential
   |
   v
Resource
```

### Delegation

```text
Human
   |
   v
Platform
   |
   v
Delegated Token
   |
   v
Downstream Resource
```

---

## Architecture decisions

Use ADRs for consequential decisions such as:

- primary issuer;
- federation model;
- workload identity mechanism;
- token format;
- offline vs online validation;
- authorization architecture;
- key hierarchy;
- revocation strategy;
- admin separation.

---

## Recommended repository shape

Follow existing repository conventions first.

A generic identity-platform repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── architecture/
│   ├── trust-domains.md
│   ├── identity-flows.md
│   ├── authorization.md
│   ├── key-management.md
│   └── decisions/
├── api/
├── src/
│   ├── issuer/
│   ├── federation/
│   ├── validation/
│   ├── delegation/
│   ├── authorization/
│   ├── revocation/
│   ├── admin/
│   └── audit/
├── policies/
├── tests/
│   ├── unit/
│   ├── protocol/
│   ├── integration/
│   ├── security/
│   ├── rotation/
│   └── recovery/
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

An identity-platform repository should expose obvious commands or equivalent
native interfaces for:

```text
check
test
test-protocol
test-integration
test-security
test-rotation
test-revocation
test-recovery
verify
```

These names are illustrative.

Use only commands the repository can actually implement.

---

## Acceptance criteria

An identity platform is not complete because users can log in or workloads can
obtain tokens.

Demonstrate the applicable subset of the following.

### Identity

- principal classes are explicit;
- trust domains are defined;
- stable subject identity is used;
- human/workload/admin identities are separated.

### Authentication

- issuer trust is explicit;
- audience validation works;
- signature validation works;
- MFA/step-up works where required.

### Workload identity

- workload evidence is validated;
- credentials are short-lived where practical;
- static shared credentials are minimized;
- workload identity cannot request arbitrary subject identity.

### Authorization

- valid authentication does not imply blanket access;
- resource/action authorization is enforced;
- cross-tenant access is denied;
- application authorization boundaries are explicit.

### Delegation

- initiating identity is preserved;
- delegated scope is narrower or equal;
- approvals/expiry bind to exact action where required;
- confused-deputy paths are tested.

### Revocation

- token/session/certificate revocation semantics are documented;
- actual revocation latency is measured;
- emergency disable path works.

### Key lifecycle

- signing/certificate keys rotate safely;
- old/new key overlap is supported;
- root trust is recoverable;
- key compromise response is documented.

### Federation

- issuer mapping is explicit;
- identity collisions are prevented;
- partner/customer trust is scoped;
- removed federation no longer grants access.

### Administration

- global admin is separated from ordinary use;
- break-glass is explicit and audited;
- privileged changes are attributable.

### Operations

- identity issuance/validation metrics exist;
- failures are observable;
- dependency outage behavior is documented;
- restore/recovery has been exercised where required.

---

## Optional composition

Common combinations:

```text
platform-architecture + identity-platform
```

For enterprise-wide shared identity capability and trust-domain architecture.

```text
developer-platform + identity-platform
```

For developer, CI, deployment, runtime, and production-admin identity
separation.

```text
control-plane + identity-platform
```

For delegated control-plane authority and workload/operator identity.

```text
identity-platform + zero-trust-service
```

For explicit verification, least privilege, scoped trust, and
resource-aware authorization.

```text
identity-platform + software-supply-chain
```

For trusted build/deploy identities and short-lived release credentials.

```text
identity-platform + high-assurance
```

For stronger key custody, dual control, revocation testing, and recovery
evidence.

```text
identity-platform + ai-security
```

For agent identity, delegated authority, tool access, and model/provider trust
boundaries.

---

## Anti-patterns

Avoid:

- treating authentication as authorization;
- using email as the only durable identity key;
- one issuer/audience accepted everywhere;
- static shared service credentials;
- long-lived production tokens by default;
- one global service account for CI, deployment, and runtime;
- bearer-token forwarding across many services;
- user-supplied tenant headers treated as authoritative;
- audience validation omitted;
- token decode without cryptographic validation;
- `kid` treated as trust;
- arbitrary algorithm acceptance;
- group claims used as the entire authorization model;
- permanent global administrator assignments;
- break-glass that is actually normal access;
- no emergency revocation path;
- revocation claimed to be instant when it depends on token expiry;
- root signing keys copied broadly;
- partner federation with enterprise-wide default trust;
- application-specific business authorization forced into central IAM;
- one broad identity token carrying every entitlement;
- audit logs containing raw tokens or secrets;
- identity platform outage causing unnecessary runtime outage;
- MFA recovery paths weaker than the MFA itself.

Do not mistake successful login for a sound identity architecture.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. identity-platform purpose;
2. principal classes;
3. trust domains and trust anchors;
4. human identity model;
5. workload/automation identity model;
6. authentication protocols;
7. issuer/audience validation model;
8. authorization and resource-scope model;
9. delegation/impersonation model;
10. token/certificate lifecycle;
11. revocation semantics and measured latency;
12. key rotation/root trust model;
13. federation model;
14. admin/break-glass model;
15. privacy/audit model;
16. protocol/security/rotation tests executed;
17. cross-tenant/revocation/recovery tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim an identity platform is "zero trust", "secure", "federated",
"passwordless", "least privilege", or "instant revocation" merely because it
uses OIDC, OAuth, mTLS, passkeys, JWTs, SPIFFE-like identities, or short-lived
certificates.

Describe the actual trust anchors, validation rules, audiences, delegation,
authorization boundaries, revocation behavior, key lifecycle, and evidence.

---

## Guiding principle

A good identity platform should make authority explicit.

A principal should have a stable identity.

An identity should come from a trusted issuer.

A credential should be valid only for the audience and duration intended.

Delegation should narrow authority, not amplify it.

Authentication should never be mistaken for authorization.

Revocation should have known semantics.

Administrative power should be rare and auditable.

And every sensitive access decision should be explainable as:

> This principal, authenticated by this trust domain, using this credential,
> was allowed to perform this action on this resource under this policy and
> this delegated scope at this point in time.
