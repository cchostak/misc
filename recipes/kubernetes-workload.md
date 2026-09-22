# kubernetes-workload.md

> Recipe for scaffolding or elevating an application workload intended to run
> on Kubernetes.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with profiles such as `zero-trust-service.md`,
> `software-supply-chain.md`, `internet-facing.md`, `hostile-input.md`,
> `high-assurance.md`, or `ai-security.md` where appropriate.
>
> This recipe defines workload-level architecture, security, lifecycle,
> observability, configuration, deployment, and verification expectations for
> software that runs on Kubernetes.
>
> It does not require a specific Kubernetes distribution, cloud provider,
> service mesh, ingress controller, GitOps product, Helm, Kustomize, operator,
> or policy engine.

## Purpose

Use this recipe when the repository owns an application, worker, service,
scheduled job, or other runtime workload that is expected to execute on
Kubernetes.

Typical examples:

- HTTP/API services;
- gRPC services;
- workers;
- scheduled jobs;
- event consumers;
- internal platform services;
- AI services;
- security tools deployed as services;
- stateful applications where Kubernetes is a justified runtime;
- sidecars or supporting application components.

Do **not** apply this recipe merely because the organization uses Kubernetes.

Do not use Kubernetes as a default answer for software that:

- is better shipped as a library;
- is a local CLI;
- is a simple one-shot script;
- has no meaningful orchestration need;
- can be operated more simply elsewhere.

The goal is not to produce a large manifest tree.

The goal is to create a workload that behaves predictably inside Kubernetes,
uses the platform intentionally, minimizes privilege, and can be operated
without hidden assumptions.

---

## Core principle

Kubernetes manages desired state.

Your application still owns application correctness.

Kubernetes can:

- restart a crashed process;
- route traffic;
- schedule containers;
- mount configuration;
- constrain resources;
- manage rollout state.

Kubernetes cannot compensate for:

- broken shutdown handling;
- unsafe retries;
- bad authorization;
- corrupt data;
- missing idempotency;
- unbounded memory;
- incorrect health semantics;
- hidden runtime dependencies.

Design the workload as an application first and a Kubernetes object second.

---

## Architectural invariants

A Kubernetes workload SHOULD satisfy these invariants unless the repository has
a documented reason not to.

### 1. Runtime configuration is externalized

Environment-specific configuration should not require rebuilding the artifact.

Use:

- environment variables;
- mounted configuration;
- runtime configuration files;
- platform-native configuration mechanisms.

Do not bake environment secrets or endpoints into the image.

### 2. Secrets are separate from ordinary config

Do not commit secret values.

Use the cluster/platform secret mechanism or an external secret provider.

Do not place secrets in:

- container images;
- ConfigMaps;
- plain-text manifests;
- command-line arguments where they may be exposed;
- logs.

### 3. The container is immutable

Treat the application image as immutable.

Prefer immutable image digests for deployment where the delivery system
supports it.

Do not mutate application code inside the running container.

### 4. The process handles termination correctly

The workload must respond to termination signals.

It should:

- stop accepting new work;
- drain or checkpoint in-flight work where appropriate;
- release resources;
- exit within the configured grace period.

Do not rely on Kubernetes forcibly killing the process as normal shutdown.

### 5. Health probes reflect real semantics

Liveness, readiness, and startup are different signals.

Do not reuse one endpoint blindly for all three.

### 6. Resource limits are intentional

Requests and limits should reflect measured or justified resource needs.

Do not omit resource configuration from production manifests without reason.

Do not copy arbitrary values from tutorials.

### 7. Privilege is minimized

Containers should run with the least privilege necessary.

Prefer:

- non-root;
- dropped capabilities;
- no privilege escalation;
- read-only root filesystem where feasible;
- narrow service-account permissions.

### 8. Scheduling assumptions are explicit

If the workload requires:

- accelerator;
- architecture;
- region;
- zone;
- local disk;
- special kernel capability;
- node label;

declare it.

Do not assume every node is equivalent.

### 9. Stateful assumptions are explicit

Know which state is:

- ephemeral;
- cached;
- durable;
- externally persisted.

Do not treat pod filesystem state as durable unless the architecture explicitly
provides persistence.

### 10. Rollout and rollback are safe

Deployments should tolerate overlapping old/new versions where rolling updates
are used.

If they cannot, choose a rollout strategy that matches the application.

---

## Workload archetype

Before creating manifests, identify the workload type.

Common types:

- stateless service;
- worker;
- scheduled job;
- long-running consumer;
- stateful service;
- singleton;
- daemon;
- batch job.

The archetype should determine controller choice.

Examples:

```text
stateless replicated service -> Deployment
stateful identity/storage    -> StatefulSet
node-local agent             -> DaemonSet
one-shot task                -> Job
scheduled task               -> CronJob
```

Do not use StatefulSet merely because the application writes files.

---

## Controller selection

Choose the simplest controller that matches lifecycle semantics.

### Deployment

Use for interchangeable replicated workloads.

### StatefulSet

Use when the workload needs stable:

- identity;
- ordinal;
- network name;
- storage association.

### DaemonSet

Use for one-per-node or selected-node agents.

### Job

Use for bounded execution that should complete.

### CronJob

Use for scheduled bounded execution.

Do not use a Deployment to emulate a batch job with custom polling logic unless
there is a reason.

---

## Container image

The image should:

- contain only required runtime content;
- have an explicit entrypoint;
- expose predictable filesystem paths;
- avoid embedded secrets;
- run as non-root where feasible;
- handle signals correctly;
- be versioned immutably.

Prefer minimal images.

Do not optimize image size at the expense of debuggability or security
blindly.

---

## Base image

Use a maintained base appropriate to the runtime.

Consider:

- security update cadence;
- libc/runtime requirements;
- debugging needs;
- package manager presence;
- certificate bundle;
- timezone data.

Pin versions sufficiently for reproducibility.

Do not use `latest`.

---

## Image metadata

Where useful, include labels such as:

- source repository;
- source revision;
- build version;
- artifact version.

Do not embed secrets or sensitive environment details.

---

## Entrypoint

The container entrypoint should invoke the application directly or through a
minimal intentional wrapper.

Avoid shell wrappers that:

- swallow signals;
- hide exit codes;
- perform uncontrolled package installation;
- mutate the filesystem unpredictably.

If a shell wrapper exists, test signal propagation.

---

## PID 1 behavior

The primary process must handle PID 1 semantics correctly.

If the runtime does not reap child processes, use an appropriate init where
needed.

Do not add an init process automatically if the application does not spawn
children.

---

## Signal handling

Handle `SIGTERM` or the platform-relevant termination signal.

Shutdown logic should be idempotent.

A second signal or timeout should not corrupt state.

---

## Termination grace period

Set `terminationGracePeriodSeconds` based on actual shutdown behavior.

Do not use a large value to hide broken termination handling.

Do not set it too small for:

- request drain;
- checkpoint;
- consumer rebalance;
- connection close.

---

## PreStop hooks

Use `preStop` only when needed.

Prefer the application handling shutdown itself.

Do not rely on long sleep hooks as the primary drain mechanism.

---

## Startup probes

Use a startup probe for applications that may legitimately take longer to
initialize.

Examples:

- large model loading;
- migrations before serving;
- runtime compilation;
- cache warmup.

Startup probes prevent liveness from killing slow-starting but healthy
instances.

---

## Readiness probes

Readiness means:

> Should this pod receive new work?

Readiness may depend on:

- application initialization;
- required configuration;
- critical dependencies;
- model loaded;
- listener ready.

Do not make readiness depend on every optional downstream system.

A dependency outage should not necessarily remove every replica from service
if useful degraded behavior exists.

---

## Liveness probes

Liveness means:

> Would restarting this process likely help?

Do not use liveness to report general system health.

A database outage should not usually cause every pod to restart repeatedly.

Avoid restart storms.

---

## Health endpoints

Health responses should be cheap.

Do not expose:

- credentials;
- internal stack traces;
- detailed topology;
- secret names.

Keep semantics documented.

---

## Resource requests

Requests influence scheduling.

Set them based on:

- measured steady-state use;
- startup requirements;
- expected workload.

Too-low requests create node overcommit and unstable scheduling.

Too-high requests waste cluster capacity.

---

## Resource limits

Limits protect cluster resources but can affect application behavior.

Understand:

- CPU throttling;
- memory OOM kill;
- ephemeral storage pressure.

Do not configure memory limits without testing OOM behavior.

---

## CPU limits

CPU limits may cause throttling.

Use them intentionally.

Some latency-sensitive services may prefer requests without strict CPU limits
depending on organizational policy.

Do not assume every workload needs identical request/limit ratios.

---

## Memory limits

Set memory limits with headroom for:

- runtime overhead;
- caches;
- bursts;
- allocator behavior.

Track OOMKilled events.

Do not treat pod restart as a memory-management strategy.

---

## Ephemeral storage

If the workload writes temporary files:

- bound size;
- define path;
- clean up;
- set ephemeral-storage requests/limits where useful.

Do not assume node disk is unlimited.

---

## Persistent storage

Use persistent volumes only when the application requires local durable state.

Prefer external durable systems for stateless services.

If using PVCs, define:

- access mode;
- capacity;
- storage class;
- backup;
- restore;
- resize behavior;
- lifecycle.

---

## EmptyDir

Use `emptyDir` for pod-lifetime temporary storage.

Know that data disappears when the pod is removed.

Do not use it for durable state.

---

## Read-only root filesystem

Use a read-only root filesystem where feasible.

Provide explicit writable mounts for:

- `/tmp`;
- cache;
- application scratch;
- runtime-generated files.

Do not enable read-only root without testing runtime behavior.

---

## User and group

Run as a non-root user where practical.

Define a stable UID/GID when required by:

- volume permissions;
- security policy;
- runtime expectations.

Avoid UID 0 unless explicitly justified.

---

## Security context

Set workload/container security context intentionally.

Common controls:

- `runAsNonRoot`;
- `allowPrivilegeEscalation: false`;
- dropped capabilities;
- read-only root filesystem;
- seccomp profile.

Do not paste a security context that breaks required behavior and then disable
it wholesale.

---

## Linux capabilities

Drop all capabilities by default where feasible.

Add back only those required.

Document why a capability is necessary.

Avoid privileged mode.

---

## Seccomp

Use the platform/runtime default seccomp profile or a stricter one where
available.

Do not disable seccomp without reason.

---

## AppArmor / SELinux

Use additional mandatory access controls where the platform supports them and
risk justifies the complexity.

Do not require them generically if the target environment does not support
them.

---

## Privileged containers

Privileged mode is a high-risk exception.

Use only when the workload fundamentally requires host-level access.

Examples may include some:

- low-level node agents;
- device plugins;
- infrastructure components.

Document the trust implications.

---

## Host namespaces

Avoid:

- host PID;
- host IPC;
- host network

unless explicitly required.

These expand the trust boundary.

---

## HostPath

Avoid `hostPath` for ordinary application workloads.

If required:

- scope path narrowly;
- prefer read-only;
- document node coupling;
- understand privilege implications.

---

## Service account

Use a dedicated service account when the workload needs Kubernetes API access.

If it does not need API access, avoid unnecessary token mounting where the
platform permits.

Do not use the namespace default service account for privileged applications.

---

## RBAC

Grant only required Kubernetes API permissions.

Scope by:

- verbs;
- resources;
- namespaces.

Avoid wildcards.

Do not grant `cluster-admin` for convenience.

---

## Automount service account token

If the workload does not need Kubernetes API access, disable automatic token
mounting where appropriate.

This reduces credential exposure.

---

## Workload identity

Where the platform supports cloud/workload identity, prefer it over static
cloud credentials.

Bind cloud permissions narrowly.

Do not place long-lived cloud access keys in Kubernetes Secrets if workload
identity is available and appropriate.

---

## Secrets

Use Kubernetes Secrets or an external secret mechanism.

Remember that a Kubernetes Secret is not automatically equivalent to a secure
vault.

Protect with:

- RBAC;
- encryption at rest where configured;
- namespace isolation;
- external secret providers where appropriate.

---

## Secret rotation

The application should tolerate credential rotation.

Know whether rotation requires:

- file reload;
- process restart;
- connection refresh.

Do not make every secret rotation require rebuilding the image.

---

## ConfigMaps

Use ConfigMaps for non-secret configuration.

Version configuration changes through source control where practical.

Do not store credentials in ConfigMaps.

---

## Environment variables

Environment variables are convenient for small config.

Be cautious with secrets because:

- process dumps;
- debug output;
- diagnostics

may expose them.

Use mounted secret files where the platform/application model makes that safer.

---

## Configuration validation

Validate configuration at startup.

Fail early for:

- missing required settings;
- invalid enum;
- invalid URL;
- contradictory options.

Do not defer obvious misconfiguration until the first production request.

---

## Service

Use a Service when stable in-cluster addressing or load balancing is needed.

Choose service type intentionally.

Do not expose `LoadBalancer` externally by default.

---

## ClusterIP

Use ClusterIP for internal service discovery.

Do not treat ClusterIP as an authentication boundary.

---

## Headless services

Use headless services when clients need direct pod identity, commonly with
StatefulSets.

Do not use them unless the application requires that behavior.

---

## Ingress and Gateway

Use the platform's established ingress/gateway mechanism.

Do not introduce a second ingress controller without need.

For external exposure, define:

- TLS;
- hostname;
- authentication boundary;
- rate limiting where applicable;
- timeout/body limits.

---

## TLS

Use TLS for traffic crossing relevant trust boundaries.

Terminate TLS at:

- ingress/gateway;
- service;
- mesh

according to architecture.

Do not disable certificate verification for internal convenience.

---

## NetworkPolicy

Use network policy where the cluster supports it and the threat model benefits.

Prefer explicit allowed flows.

Remember:

- network policy is defense in depth;
- it is not application authorization.

Test policy behavior in the target CNI/environment.

---

## Egress policy

Restrict egress for workloads that do not need arbitrary internet access.

Consider allowing only:

- required APIs;
- databases;
- identity provider;
- telemetry endpoints.

Protect metadata/internal infrastructure endpoints.

---

## DNS

Treat DNS as service discovery, not identity proof.

Applications should still authenticate peers when required.

---

## Pod disruption

Plan for voluntary disruptions.

Use PodDisruptionBudget when availability requirements justify it.

Do not set a PDB that prevents cluster maintenance indefinitely.

---

## Replicas

Choose replica count based on availability and load.

Do not set `replicas: 3` by ritual.

Some workloads legitimately require:

- one replica;
- many replicas;
- autoscaling.

---

## Anti-affinity and topology spread

Use topology controls when resilience requires replicas across:

- nodes;
- zones;
- failure domains.

Avoid hard constraints that make scheduling impossible without strong reason.

Prefer topology spread for common availability needs.

---

## Node selectors and affinity

Use selectors/affinity for genuine hardware or placement requirements.

Examples:

- GPU;
- architecture;
- zone;
- compliance node pool.

Do not pin pods to named nodes.

---

## Taints and tolerations

Use tolerations to opt into special node pools.

A toleration permits scheduling; it does not force it.

Use node affinity/selectors as needed.

---

## Priority classes

Use workload priority carefully.

Do not mark ordinary application workloads as critical.

Priority can starve other workloads during pressure.

---

## Horizontal Pod Autoscaler

Use HPA when workload can scale horizontally and a useful scaling signal
exists.

Possible signals:

- CPU;
- memory;
- request rate;
- queue depth;
- custom metrics.

Do not add HPA for a singleton or non-horizontal workload.

---

## Scaling signal

Choose signals related to saturation.

CPU may be poor for:

- queue consumers;
- GPU inference;
- I/O-bound services.

Avoid autoscaling on noisy metrics without stabilization.

---

## Vertical scaling

Use vertical recommendations or resizing only where platform/runtime support and
operational model justify it.

Be aware that resource changes may restart workloads.

---

## Scale-to-zero

Scale-to-zero may be useful for event-driven/non-latency-sensitive workloads.

Understand:

- cold start;
- lost warm cache;
- dependency connection startup.

Do not use it where latency requirements cannot tolerate it.

---

## Jobs

Jobs should be idempotent or safe to retry where possible.

Set:

- backoff limit;
- timeout/deadline;
- resource limits.

Do not allow failed Jobs to retry forever.

---

## CronJobs

For CronJobs define:

- schedule;
- concurrency policy;
- starting deadline where relevant;
- history limits;
- timezone if supported/needed.

Avoid overlapping runs when unsafe.

---

## Concurrency policy

For scheduled work choose:

- Allow;
- Forbid;
- Replace

according to semantics.

Do not accept overlapping destructive jobs by default.

---

## Completion deadlines

Long-running jobs should have bounded execution where practical.

Use active deadlines or application timeouts.

---

## Consumers and workers

Message/event consumers need graceful shutdown.

On termination:

- stop fetching new work;
- finish or checkpoint current work;
- release lease/partition;
- exit.

Do not acknowledge messages before durable completion if semantics require
otherwise.

---

## Leader election

Use leader election only for singleton coordination that truly needs it.

Prefer platform-supported leases.

Do not invent ad hoc ConfigMap locks when established coordination primitives
exist.

---

## Stateful workloads

Before using Kubernetes for stateful systems, identify:

- persistence model;
- replication;
- failover;
- backup;
- restore;
- consistency;
- operator burden.

Managed external state may be simpler.

Do not self-host databases on Kubernetes merely for architectural symmetry.

---

## StatefulSet upgrades

For stateful workloads, account for:

- ordered rollout;
- partitioning;
- schema compatibility;
- storage compatibility.

Do not assume rolling back the container also rolls back durable state.

---

## Database migrations

Migration behavior must be explicit.

Possible models:

- separate Job;
- init container;
- application startup;
- external release step.

Prefer a model that avoids multiple replicas racing to migrate.

Migration logic must be compatible with rollout/rollback.

---

## Init containers

Use init containers for bounded setup that must occur before the app starts.

Examples:

- configuration generation;
- waiting for prerequisite condition;
- permissions setup.

Do not hide complex deployment orchestration in long-running init scripts.

---

## Sidecars

Use sidecars when they provide a concrete capability.

Examples:

- proxy;
- log helper;
- local agent.

Every sidecar adds:

- resource cost;
- lifecycle complexity;
- failure modes;
- trust relationships.

Do not use sidecars by default.

---

## Service mesh

A service mesh may provide:

- mTLS;
- traffic policy;
- telemetry;
- identity.

It also adds complexity.

Do not require a mesh for a normal workload unless the environment already
uses one or the requirements justify it.

---

## Logging

Write application logs to stdout/stderr unless the platform convention
requires otherwise.

Use structured logs where useful.

Do not write critical logs only to ephemeral local files.

---

## Log content

Do not log:

- access tokens;
- passwords;
- private keys;
- raw secrets.

Be careful with:

- request bodies;
- headers;
- personal data.

---

## Metrics

Expose application metrics if the environment uses metrics collection.

Prefer stable low-cardinality dimensions.

Do not use:

- user IDs;
- request IDs;
- raw paths with unbounded IDs

as metric labels.

---

## Tracing

Use tracing when useful for distributed requests.

Propagate trace context safely.

Do not include secrets in spans.

---

## Kubernetes events

Do not rely solely on application logs.

Operational diagnosis may also require:

- pod status;
- events;
- restart reason;
- OOM state;
- scheduling state.

---

## Labels

Use labels consistently for selection and operations.

Typical dimensions:

- application;
- component;
- version;
- environment where appropriate.

Do not use mutable labels as security authorization by themselves.

---

## Annotations

Use annotations for non-selector metadata.

Avoid putting secrets or large arbitrary data in annotations.

---

## Naming

Resource names should be stable and predictable.

Do not embed unnecessary environment-specific randomness.

---

## Namespace strategy

Use namespaces according to organizational isolation needs.

Namespaces provide administrative grouping and some policy boundaries.

They are not a complete security boundary by themselves.

---

## Environment separation

Prefer stronger separation for production.

Possible mechanisms:

- distinct clusters;
- namespaces;
- accounts/projects;
- credentials.

Use the architecture that matches risk.

Do not assume namespace separation alone is sufficient for every production
boundary.

---

## Limits and quotas

Namespaces may use:

- ResourceQuota;
- LimitRange.

Workloads should coexist with these controls.

Do not rely on namespace defaults without knowing them.

---

## Image pull policy

Use image pull policy intentionally.

With immutable digests, repeated pulling policy can be simpler to reason about.

Do not use mutable tags plus cached pull behavior and expect deterministic
deployments.

---

## Image registry

Use approved registries according to platform policy.

Authenticate with narrow pull credentials where needed.

Do not embed registry credentials in image references or manifests.

---

## Image signing and verification

If artifact signing is required, verification should occur before or during
admission/deployment.

Compose with `software-supply-chain.md`.

Signing without enforcement is incomplete.

---

## Admission policy

Clusters may enforce policies such as:

- allowed registry;
- signed image;
- non-root;
- no privileged mode;
- required resources;
- approved capabilities.

The workload should satisfy actual policy.

Do not add policy-specific annotations blindly.

---

## Pod Security Standards

Where the cluster uses Kubernetes Pod Security Standards or equivalent, target
the strongest level compatible with the workload.

Ordinary application workloads should generally avoid requiring privileged
behavior.

Do not label a workload "restricted compliant" unless validated against actual
policy.

---

## RuntimeClass

Use RuntimeClass only when the workload requires a specific runtime, such as a
sandboxed runtime.

Do not add it for appearance.

---

## Probes and autoscaling interaction

Probe failures, startup delay, and HPA behavior interact.

Test cold start and scale-out.

Do not let slow startup cause newly created pods to be killed before becoming
ready.

---

## Rolling updates

For Deployments, choose update settings based on capacity and availability.

Understand:

- maxUnavailable;
- maxSurge.

Do not copy percentages blindly.

---

## Backward compatibility

During rolling updates, old and new replicas may run concurrently.

Ensure compatibility for:

- API;
- database schema;
- message schema;
- shared cache;
- external protocol.

If versions cannot coexist, use a different rollout strategy.

---

## Progressive delivery

Use canary or blue/green when release risk justifies it.

Do not add a rollout controller solely because it exists.

If traffic weighting is claimed, distinguish actual request routing from replica
ratio.

---

## Rollback

Rollback should reference a known-good immutable artifact.

Do not roll back to a mutable tag.

Know whether data/schema changes make rollback unsafe.

---

## Deployment history

Preserve enough release metadata to identify:

- artifact;
- configuration;
- rollout;
- outcome.

GitOps or deployment tooling may provide this.

---

## GitOps

If GitOps is used:

- Kubernetes desired state belongs in Git;
- deployment controller reconciles it;
- CI should not secretly bypass GitOps with direct `kubectl`.

Do not require GitOps in every Kubernetes project.

---

## Helm

Use Helm when templating/package semantics provide value.

Avoid overly abstract charts with hundreds of knobs.

Validate rendered output.

Do not use Helm merely to avoid writing a few YAML files.

---

## Kustomize

Use Kustomize when overlay-based environment variation is useful.

Avoid complex patch stacks that are hard to reason about.

Keep environment differences minimal.

---

## Raw manifests

Raw manifests are acceptable for small/simple deployments.

Do not introduce a templating system before there is meaningful variation.

---

## Manifest validation

Validate manifests using appropriate tooling.

Checks may include:

- YAML syntax;
- Kubernetes schema;
- policy;
- rendering;
- object references.

Do not claim deployability from syntax validation alone.

---

## Local Kubernetes

For local verification, use the simplest suitable disposable cluster if actual
Kubernetes behavior matters.

Examples:

- kind;
- k3d;
- minikube.

Do not require local Kubernetes if unit/integration tests can validate the
change adequately.

---

## Portability

Avoid provider-specific Kubernetes features unless requirements justify them.

Where provider-specific resources are used, isolate and document them.

Do not pretend provider-specific manifests are portable.

---

## CRDs

Do not create a custom resource definition for ordinary application
configuration.

CRDs add:

- API design;
- upgrade;
- compatibility;
- controller requirements.

Use them only when a real Kubernetes-native API is justified.

---

## External dependencies

For every dependency, define:

- address/config;
- timeout;
- retry;
- authentication;
- failure behavior.

Kubernetes DNS/service discovery does not remove ordinary distributed-systems
failure modes.

---

## Dependency startup ordering

Avoid assuming dependent services start before the workload.

Applications should tolerate dependencies becoming available later.

Do not use init containers that wait forever for every downstream service.

---

## Retry storms

When dependencies fail, many replicas may retry simultaneously.

Use bounded exponential backoff and jitter where appropriate.

Do not create cluster-wide retry storms.

---

## Graceful degradation

If optional dependencies fail, continue safely where possible.

Expose degraded state via telemetry.

Do not make every optional integration a readiness dependency.

---

## Pod restarts

Restarts are expected.

Application correctness must tolerate them.

Do not rely on in-memory state for durable workflow progress unless loss is
acceptable.

---

## Rescheduling

Pods may move between nodes.

Avoid node-local assumptions unless explicitly designed.

---

## Eviction

Design for:

- node drain;
- memory pressure;
- spot/preemptible termination where used.

High-value workloads may need topology or capacity controls.

---

## Preemptible/spot nodes

Use for workloads that can tolerate interruption.

Do not schedule critical singleton stateful workloads there without recovery
design.

---

## Backup and restore

If the workload owns persistent data, define backup and restore.

Do not assume PVC persistence equals backup.

Test restore where data matters.

---

## Disaster recovery

For critical workloads, identify:

- artifact recovery;
- config recovery;
- secrets recovery;
- data recovery;
- cluster dependency.

Do not invent RPO/RTO values.

---

## Security testing

Test relevant workload boundaries.

Examples:

- non-root execution;
- denied filesystem paths;
- denied Kubernetes API access;
- network-policy behavior;
- secret exposure;
- oversized requests;
- termination behavior.

---

## Policy tests

Where admission policies exist, validate manifests against them before
deployment where practical.

Do not wait until production admission to discover obvious policy rejection.

---

## Load testing

For scalable services, test representative load.

Observe:

- resource use;
- latency;
- HPA behavior;
- saturation;
- queueing;
- OOM.

Do not size requests/limits solely from idle behavior.

---

## Shutdown testing

Explicitly test termination.

Verify:

- SIGTERM received;
- readiness changes;
- new work stops;
- in-flight work completes or is safely abandoned;
- process exits within grace period.

---

## Restart testing

Kill/restart pods in a safe environment.

Confirm the application tolerates restart.

---

## Dependency failure testing

Simulate loss of:

- database;
- broker;
- upstream API;
- DNS where relevant.

Confirm retries and readiness/liveness behave as intended.

---

## Negative-path tests

At least one meaningful Kubernetes failure path should be demonstrated where
the environment permits.

Examples:

- invalid secret/config prevents startup clearly;
- readiness removes unhealthy pod from service;
- non-root policy blocks forbidden behavior;
- network policy denies unauthorized path;
- OOM or resource saturation fails predictably;
- rollout of bad version is detectable and reversible.

---

## Acceptance criteria

A Kubernetes workload is not complete because `kubectl apply` succeeds.

Demonstrate the applicable subset of the following.

### Image and runtime

- image builds;
- image has immutable version/digest;
- container starts;
- process runs as intended;
- signal handling works.

### Configuration

- config is externalized;
- secrets are separate;
- startup validation works;
- environment-specific config does not require rebuild.

### Health

- startup semantics are correct;
- readiness controls traffic/work;
- liveness does not cause restart loops;
- health endpoints expose no sensitive data.

### Security

- non-root where feasible;
- privilege escalation disabled where feasible;
- capabilities minimized;
- service-account permissions scoped;
- unnecessary token mount disabled where applicable;
- secret values absent from manifests/logs.

### Resources

- requests/limits are defined or consciously omitted;
- OOM/throttling behavior is understood;
- temp storage is bounded where needed.

### Networking

- Service/Ingress exposure is intentional;
- TLS/auth boundaries are defined;
- egress is deliberate where risk requires it;
- network policy is tested where used.

### Lifecycle

- graceful shutdown works;
- restarts are tolerated;
- rollout compatibility is understood;
- rollback target is immutable.

### Operations

- logs are available;
- metrics exist where appropriate;
- alertable failure modes are observable;
- deployment and rollback procedures are documented.

---

## Optional composition

Common combinations:

```text
secure-service + kubernetes-workload
```

For a production API/service deployed to Kubernetes.

```text
kubernetes-workload + zero-trust-service
```

For workload identity, service-to-service authorization, and reduced implicit
network trust.

```text
kubernetes-workload + software-supply-chain
```

For signed images, provenance, admission verification, and immutable promotion.

```text
kubernetes-workload + internet-facing
```

For hardened public exposure, abuse controls, TLS, ingress, and edge policy.

```text
kubernetes-workload + high-assurance
```

For stronger fault injection, recovery testing, stricter policy, and deeper
operational verification.

```text
ai-service + kubernetes-workload
```

For model-backed application services running under Kubernetes lifecycle and
resource controls.

```text
model-serving + kubernetes-workload
```

For accelerator-aware model serving with Kubernetes scheduling and workload
security.

---

## Anti-patterns

Avoid:

- Kubernetes solely because "production uses Kubernetes";
- giant manifest trees for trivial workloads;
- `latest` image tags;
- secrets in ConfigMaps;
- secrets committed in manifests;
- running as root by default;
- privileged containers for convenience;
- wildcard RBAC;
- using the default service account for privileged API access;
- hostPath for ordinary application storage;
- one health endpoint blindly used for startup/readiness/liveness;
- liveness tied to every external dependency;
- sleep-based shutdown;
- arbitrary CPU/memory values copied from examples;
- no resource requests in heavily shared clusters without reason;
- one replica count used by ritual;
- HPA based on irrelevant metrics;
- readiness before model/cache/runtime initialization completes;
- mutable config inside container images;
- direct CI `kubectl` when GitOps is the intended deployment authority;
- service mesh adoption without a concrete need;
- CRDs for ordinary app configuration;
- PVCs treated as backups;
- namespace membership treated as authorization;
- relying on pod-local files for durable state;
- rolling update of incompatible versions without a migration strategy.

Do not mistake Kubernetes manifest volume for production maturity.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. workload archetype;
2. Kubernetes controller used;
3. image identity strategy;
4. runtime user/security context;
5. configuration/secrets model;
6. service-account/RBAC model;
7. health probe semantics;
8. resource request/limit strategy;
9. networking/exposure model;
10. persistent/ephemeral state model;
11. scaling model;
12. shutdown/restart behavior;
13. rollout/rollback model;
14. observability signals;
15. policy/admission assumptions;
16. tests executed;
17. Kubernetes integration checks executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a workload is "production-ready", "secure", "highly available", or
"cloud native" merely because manifests exist or deploy successfully.

Describe the runtime, privilege, lifecycle, resource, networking, and
verification controls that were actually exercised.

---

## Guiding principle

A good Kubernetes workload should remain understandable even after the YAML is
removed.

The application should know how to start.

It should know how to stop.

It should know when it is ready.

It should know what state is durable.

It should use only the privilege it needs.

It should fit inside explicit resource bounds.

It should tolerate rescheduling and restart.

And Kubernetes should provide orchestration around a sound application design,
not compensate for the absence of one.
