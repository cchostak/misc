# mcp-tool-server.md

> Recipe for scaffolding or elevating an MCP-compatible tool server.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with profiles such as `ai-security.md`, `zero-trust-service.md`,
> `software-supply-chain.md`, `hostile-input.md`, or `high-assurance.md`
> where appropriate.
>
> This recipe defines architectural, authorization, safety, schema, transport,
> observability, testing, and operational expectations for servers that expose
> tools, resources, prompts, or capabilities to AI clients through MCP or a
> comparable tool protocol.
>
> It does not require a specific language, SDK, deployment platform, transport,
> identity provider, or hosting environment.

## Purpose

Use this recipe when a repository exposes machine-callable capabilities to an
AI client or agent.

Typical examples:

- MCP tool servers;
- internal enterprise tool gateways;
- repository/tool integrations;
- cloud/infrastructure control tools;
- security-analysis servers;
- database/query tools;
- messaging and ticketing integrations;
- document/resource providers;
- local developer-tool bridges;
- SaaS connector servers.

The goal is not merely to expose functions over MCP.

The goal is to expose a **safe, narrow, typed capability boundary** where:

- tools have explicit semantics;
- side effects are classified;
- identities are authenticated where needed;
- authorization is deterministic;
- arguments are validated;
- secrets stay out of model context;
- external responses remain untrusted;
- audit records explain what happened;
- clients cannot silently expand their authority.

---

## Core principle

An MCP server is an authority boundary.

The client may ask:

```text
call tool X with arguments Y
```

but the server must still determine:

```text
is this caller allowed
to invoke this tool
with these arguments
against this resource
in this environment
right now?
```

MCP standardizes tool discovery and invocation.

It does **not** replace authentication, authorization, policy, or safe tool design.

---

## Architectural invariants

An MCP tool server SHOULD satisfy these invariants unless the repository has a
documented reason not to.

### 1. Tool schemas are security boundaries

Every tool must have a clear, machine-readable schema.

Validate:

- required arguments;
- types;
- enums;
- ranges;
- identifiers;
- optional fields;
- unsupported extra fields where strictness is useful.

Schema validation is necessary but not sufficient.

Business and authorization rules must also be enforced.

### 2. Tools are narrow

Prefer a specific tool such as:

```text
create_pull_request(repo, branch, title, body)
```

over a generic tool such as:

```text
github(operation, payload)
```

where the broad interface would allow hidden privilege expansion.

Narrow tools improve:

- authorization;
- auditing;
- model reliability;
- testing;
- blast-radius control.

### 3. Tool possession does not imply tool authorization

A client discovering a tool does not automatically mean the caller may use it.

Authorization should consider:

- caller identity;
- tenant;
- environment;
- target resource;
- action;
- tool;
- side-effect class;
- approval state;
- delegated authority.

### 4. Read and write operations are distinct

Do not combine read, write, delete, and admin behavior into one ambiguous
operation unless there is a strong reason.

Prefer separate capabilities.

### 5. Secrets stay server-side

Do not return raw backend credentials to clients.

The server should use credentials internally to perform authorized operations.

Prefer:

```text
client requests:
    fetch issue

server:
    uses backend credential internally
```

over:

```text
server returns backend token to client
```

### 6. External results are untrusted

Content returned from APIs, repositories, documents, browsers, databases, and
other external systems may contain malicious or misleading instructions.

Do not label tool output as trusted merely because it came through the MCP
server.

### 7. Side effects are explicit

Each tool should have a documented side-effect class.

A useful default classification is:

```text
READ_ONLY
REVERSIBLE_WRITE
HIGH_IMPACT_WRITE
IRREVERSIBLE
```

This classification should inform policy, approval, and logging.

### 8. Authorization is deterministic

Do not delegate authorization to the model.

The MCP server or a dedicated policy layer must enforce access.

### 9. Fail closed for sensitive actions

If identity, authorization, approval, policy, or target validation cannot be
established, sensitive operations should be denied.

### 10. Every high-impact action is auditable

Preserve enough information to reconstruct:

- caller;
- tool;
- validated arguments;
- target resource;
- authorization decision;
- approval if applicable;
- backend result;
- observed effect.

---

## Server role definition

Before implementation, identify what the server exposes.

Possible capability classes:

- tools;
- resources;
- prompts;
- templates;
- subscriptions/events;
- metadata.

Document:

- intended client types;
- trust level;
- backend systems;
- required credentials;
- side effects;
- tenant boundaries;
- deployment model.

Do not expose capabilities "just in case".

---

## Tool inventory

Maintain an explicit tool inventory.

For each tool record:

- name;
- purpose;
- input schema;
- output schema;
- side-effect class;
- required permission;
- target resource type;
- retry/idempotency semantics;
- approval requirements;
- backend dependency.

This may live in code, metadata, or generated documentation.

The important requirement is discoverability and testability.

---

## Tool naming

Tool names should be:

- stable;
- specific;
- verb-oriented where appropriate;
- unambiguous.

Prefer:

```text
read_issue
create_issue
close_issue
```

over:

```text
issue_action
do_issue
run
```

Names influence both humans and models.

---

## Tool descriptions

Descriptions are security-relevant because clients use them to decide when to
invoke a tool.

Descriptions should state:

- what the tool does;
- what it does not do;
- whether it mutates state;
- important preconditions;
- dangerous effects;
- expected arguments.

Do not make destructive operations sound harmless.

---

## Argument validation

Validate tool arguments before backend access.

Examples:

- repository name belongs to allowed scope;
- environment is one of allowed values;
- file path remains inside approved root;
- URL uses approved scheme;
- SQL identifier matches allowed resource;
- deployment target exists;
- amount/value is within configured bounds.

Do not rely solely on schema type validation.

---

## Resource scoping

Tools should be scoped to explicit resources.

Possible scope dimensions:

- repository;
- organization;
- tenant;
- account;
- project;
- cluster;
- namespace;
- database;
- table;
- directory;
- region;
- environment.

Avoid broad global authority when a narrow resource scope is sufficient.

---

## Caller identity

If the server is shared, remote, or privileged, establish caller identity.

Possible mechanisms include:

- mTLS;
- OAuth/OIDC;
- signed tokens;
- workload identity;
- local OS identity for local-only servers;
- another deployment-appropriate mechanism.

Do not trust a caller-supplied username field as authentication.

---

## Delegated authority

If a client acts on behalf of a human user, preserve the distinction between:

- user identity;
- client/agent identity;
- MCP server identity;
- backend service identity.

Do not let the server's broad backend credential silently grant every caller
the server's full authority.

---

## Authorization

Authorization should be explicit.

A useful policy question is:

```text
May caller C,
through client A,
invoke tool T,
with arguments X,
against resource R,
in environment E?
```

Possible models:

- RBAC;
- ABAC;
- resource ownership;
- policy-as-code;
- capability tokens;
- delegated scopes.

Use the simplest model that correctly represents the boundary.

---

## Policy enforcement point

The MCP server is usually an enforcement point.

The order should be:

```text
request
  ->
authenticate
  ->
validate tool exists
  ->
validate arguments
  ->
authorize
  ->
check approval if needed
  ->
invoke backend
  ->
verify result
  ->
audit
```

Do not invoke the backend before authorization.

---

## Approval gates

High-impact tools may require explicit approval.

Examples:

- production deployment;
- destructive delete;
- credential rotation;
- external message send;
- merge to protected branch;
- infrastructure mutation.

Approval should bind to:

- tool;
- target;
- material arguments;
- caller;
- expiration.

Do not reuse approval after material arguments change.

---

## Read/write separation

Where practical, separate:

```text
get_resource
update_resource
delete_resource
```

rather than:

```text
resource(action=...)
```

This simplifies:

- policy;
- permissions;
- observability;
- model behavior;
- approval.

---

## Dangerous generic tools

Treat the following as high-risk:

- arbitrary shell;
- arbitrary HTTP;
- arbitrary SQL;
- unrestricted filesystem;
- cloud-admin passthrough;
- browser automation with authenticated session.

If such a tool is required:

- scope aggressively;
- isolate execution;
- add policy;
- add limits;
- log usage;
- require approval for dangerous operations.

Prefer typed alternatives whenever possible.

---

## Filesystem tools

For file access:

- define allowed root(s);
- normalize paths;
- prevent path traversal;
- handle symlinks;
- separate read/write roots;
- avoid exposing secret directories.

Do not allow arbitrary absolute paths by default.

---

## Shell tools

If shell execution exists:

- use ephemeral sandboxing where possible;
- restrict working directory;
- restrict environment variables;
- restrict network access;
- set time/resource limits;
- avoid privileged execution;
- remove production credentials.

Generated commands must be treated as untrusted.

---

## SQL/data tools

Prefer typed data operations over arbitrary SQL.

If SQL is exposed:

- parameterize values;
- restrict statements;
- scope database identity;
- use read-only credentials for read tools;
- bound result sizes;
- protect against cross-tenant queries.

Do not give a model unrestricted DBA authority.

---

## HTTP tools

If arbitrary or semi-arbitrary HTTP fetching exists:

- restrict schemes;
- validate destinations;
- protect private/internal networks where needed;
- protect metadata endpoints;
- enforce timeouts;
- bound response size;
- handle redirects deliberately.

An HTTP tool can become an SSRF and exfiltration primitive.

---

## Cloud tools

For cloud APIs:

- use narrowly scoped identities;
- separate read/write/admin tools;
- scope account/project/region;
- avoid organization-wide admin by default;
- require approval for destructive operations.

Do not expose a cloud CLI passthrough when typed operations suffice.

---

## Messaging tools

Separate:

```text
draft_message
send_message
```

where practical.

Validate:

- recipient;
- channel;
- content;
- attachments.

External communication is a side effect.

---

## Repository tools

For source-control tools:

- scope repository/org;
- distinguish read, branch, PR, merge, admin;
- avoid protected-branch direct write where PR flow is appropriate;
- validate branch/revision;
- respect repository instructions.

Do not let arbitrary repository content change server authorization policy.

---

## MCP resources

If exposing resources:

- define URI/identifier semantics;
- authorize resource reads;
- bound resource size;
- sanitize metadata;
- preserve provenance.

Do not expose sensitive resources solely because they are discoverable.

---

## Resource discovery

Discovery may itself leak sensitive metadata.

Consider whether callers are allowed to know:

- resource names;
- repository existence;
- tenant IDs;
- secret names;
- internal endpoints.

Apply authorization to discovery where needed.

---

## MCP prompts

If exposing reusable prompts/templates:

- version them;
- review them;
- avoid secrets;
- treat them as behavior-affecting artifacts;
- document intended use.

Do not let remote/untrusted systems modify privileged prompt templates without review.

---

## Prompt injection from tool results

Assume tool output may contain adversarial natural language.

Examples:

- issue comments;
- web pages;
- README files;
- emails;
- tickets;
- document text.

Mitigate with:

- deterministic policy;
- narrow tool capabilities;
- content provenance;
- read/write separation;
- approval gates.

Do not rely solely on prompt instructions to neutralize hostile tool output.

---

## Output schema

Where practical, return typed outputs.

Prefer:

```json
{
  "id": "123",
  "state": "open",
  "title": "..."
}
```

over long unstructured prose.

Typed outputs reduce ambiguity and injection surface.

---

## Output minimization

Return only data required for the client task.

Do not expose:

- backend credentials;
- internal auth tokens;
- unnecessary headers;
- raw secrets;
- unrestricted environment details.

Minimize sensitive metadata.

---

## Error model

Define stable error categories.

Examples:

```text
INVALID_ARGUMENT
NOT_FOUND
AUTHENTICATION_REQUIRED
AUTHORIZATION_DENIED
APPROVAL_REQUIRED
RATE_LIMITED
BACKEND_TIMEOUT
BACKEND_ERROR
UNKNOWN_OUTCOME
INTERNAL_ERROR
```

Do not return raw backend exceptions or stack traces to clients.

---

## Unknown outcome

For side-effecting backend calls, timeout does not necessarily mean failure.

Represent unknown outcome explicitly.

Before retry:

- reconcile backend state;
- determine whether the action already succeeded;
- use idempotency keys where supported.

Do not blindly repeat a potentially destructive write.

---

## Idempotency

For retry-prone writes, define idempotency.

Possible mechanisms:

- backend idempotency key;
- operation ID;
- unique resource constraint;
- precondition;
- deduplication.

Document whether a tool is safe to retry.

---

## Timeouts

Every external/backend call should have an intentional timeout where applicable.

Avoid indefinite hangs.

Use per-operation timeout budgets.

---

## Retry policy

Retry only retryable failures.

Do not retry:

- authorization denial;
- invalid arguments;
- policy denial;
- explicit approval requirement;
- deterministic backend rejection.

Use bounded backoff/jitter where appropriate.

---

## Rate limiting

Rate-limit at the appropriate level.

Possible keys:

- caller;
- tenant;
- tool;
- backend;
- side-effect class.

Expensive or destructive tools may need stricter limits.

---

## Quotas

Where backend cost matters, define quotas.

Examples:

- API spend;
- cloud operations;
- expensive searches;
- large exports.

The client should not be able to disable quotas through tool arguments.

---

## Concurrency

Bound concurrency to protect backends and the server.

Avoid unbounded parallel tool execution.

For writes against the same resource, consider serialization or optimistic concurrency controls.

---

## Precondition checks

For high-impact actions, verify current state.

Examples:

- branch still points to expected commit;
- deployment still references expected digest;
- resource still exists;
- target environment matches policy.

This reduces stale-plan execution.

---

## Postcondition verification

After a high-impact action, verify the effect where practical.

Examples:

```text
update resource
    ->
backend says success
    ->
read resource
    ->
confirm state
```

Do not treat a transport-level 200 as proof of intended effect if stronger verification is available.

---

## Secrets

Secrets should remain server-side.

Requirements:

- do not expose raw secrets to clients;
- do not log secrets;
- do not include them in tool outputs;
- scope backend credentials;
- prefer short-lived credentials;
- rotate;
- separate environments.

If the backend supports workload identity, prefer it over static keys.

---

## Tenant isolation

For multi-tenant servers:

- derive tenant from trusted identity;
- scope tools;
- scope backend credentials where possible;
- scope caches;
- scope resources;
- test cross-tenant denial.

Do not trust a `tenant_id` tool argument as the sole tenant boundary.

---

## Environment isolation

Separate production and non-production access.

Environment should be part of policy.

A client connected to development capability should not automatically reach production resources.

---

## Transport security

For remote servers:

- use authenticated transport;
- use TLS where appropriate;
- validate certificates;
- protect tokens;
- avoid plaintext credentials.

For local servers:

- restrict local socket/process access where needed;
- do not assume localhost equals trusted.

---

## Session state

Keep server-side session state minimal.

If sessions exist:

- bind them to caller identity;
- expire them;
- isolate tenants;
- do not use session ID as sole authorization.

---

## Connection lifetime

Long-lived sessions should handle:

- credential expiry;
- policy change;
- server restart;
- revocation.

Do not assume authorization remains valid forever because a session was established once.

---

## Dynamic tool registration

Dynamic tools increase risk.

If supported:

- authenticate registrants;
- review schema;
- classify side effects;
- apply policy;
- avoid automatic privileged enablement.

Do not let a client register arbitrary code and then invoke it with server authority.

---

## Third-party tool servers

If federating or proxying other MCP/tool servers:

- authenticate upstream servers;
- scope permissions;
- validate schemas;
- preserve provenance;
- classify trust;
- isolate failures.

Do not treat federation as transitive trust.

---

## Versioning

Version:

- tool schemas;
- output schemas;
- server API behavior;
- policy-relevant semantics.

Avoid breaking existing clients casually.

If a tool changes side-effect semantics, treat that as a security-relevant breaking change.

---

## Tool deprecation

Deprecate tools explicitly.

Provide:

- replacement;
- timeline;
- migration guidance.

Do not silently change a read tool into a write-capable tool under the same name.

---

## Discovery metadata integrity

Tool metadata should come from trusted server configuration.

Do not let untrusted backend content rewrite tool descriptions or side-effect labels.

---

## Audit logging

For meaningful actions, record:

- request/run ID;
- caller identity;
- client identity where available;
- tool;
- validated arguments;
- resource scope;
- policy result;
- approval ID;
- backend target;
- result;
- duration.

Redact sensitive values.

---

## Audit integrity

For higher-assurance deployments, prevent ordinary clients/tools from deleting or rewriting their own audit records.

---

## Observability

Track:

- tool invocation rate;
- success/failure;
- latency;
- authn failures;
- authz denials;
- approval-required counts;
- backend errors;
- unknown outcomes;
- rate-limit events;
- per-tool saturation.

Avoid high-cardinality labels containing raw user data.

---

## Health

Expose health appropriate to the runtime.

Distinguish:

- process liveness;
- readiness;
- critical backend availability.

Do not make liveness fail merely because one optional backend is unavailable.

---

## Backpressure

If backends degrade:

- bound queues;
- reject or shed load;
- surface rate limiting;
- avoid unbounded retries.

Do not let client demand exhaust server resources.

---

## Caching

Cache only safe data.

Consider cache keys that include:

- tenant;
- identity scope;
- resource;
- tool version;
- authorization context.

Do not cache protected results globally if callers have different permissions.

---

## Backend API drift

External APIs may change.

Use contract tests where practical.

Do not assume a backend response will always match yesterday's shape.

---

## Sandbox

If a tool runs untrusted code or parses hostile content:

- isolate execution;
- restrict filesystem;
- restrict network;
- bound CPU/memory/time;
- destroy the environment after use.

Apply `security-tool.md` or `hostile-input.md` where appropriate.

---

## Data retention

Do not retain full tool inputs/outputs indefinitely by default.

Define retention for:

- audit;
- debug logs;
- cached results;
- uploaded files;
- generated artifacts.

Minimize sensitive content.

---

## Privacy

Tool servers often bridge privileged enterprise systems.

Document data flows.

Do not forward data to third-party services unless that is intentional and authorized.

---

## Supply chain

The tool server itself may become highly privileged.

Compose with `software-supply-chain.md`.

At minimum consider:

- pinned dependencies;
- signed releases;
- SBOM;
- provenance;
- dependency scanning;
- reproducible builds where practical.

A compromised MCP server can become a privilege escalation path.

---

## Testing strategy

### Unit tests

Test:

- schema validation;
- argument normalization;
- policy;
- side-effect classification;
- error mapping;
- idempotency logic;
- output shaping.

### Authorization tests

Test:

- allowed caller;
- denied caller;
- wrong tenant;
- wrong environment;
- insufficient scope;
- approval required;
- expired approval.

### Backend contract tests

Test real or representative backend behavior.

Validate:

- request mapping;
- response parsing;
- retry behavior;
- timeout behavior.

### Security tests

Test:

- path traversal;
- SSRF where relevant;
- injection into backend queries;
- secret redaction;
- tool-result prompt injection handling;
- cross-tenant access.

### Negative-path tests

At least one important denial should be demonstrated end to end.

Examples:

- unauthorized caller denied;
- destructive tool denied without approval;
- wrong environment denied;
- malformed arguments rejected before backend call;
- unknown-outcome write reconciled before retry.

---

## Client simulation

Where possible, include integration tests that simulate a real MCP client.

Verify:

- discovery;
- tool schema exposure;
- invocation;
- errors;
- authn/authz;
- result format.

Do not test only internal handler functions.

---

## Fuzzing

Consider fuzzing:

- request parsing;
- schema validation;
- URI/resource parsing;
- transport framing;
- file/path tools;
- custom protocol adapters.

Crashes should become regression tests.

---

## Threat model

Document threats such as:

- malicious client;
- compromised client;
- prompt-injected client;
- compromised backend;
- malicious backend content;
- credential theft;
- cross-tenant access;
- tool confusion;
- overly broad schemas;
- unsafe retries;
- arbitrary shell/HTTP abuse.

A privileged tool server should have an explicit threat model.

---

## Security review triggers

Require stronger review when changes affect:

- new tool registration;
- side-effect class;
- backend credentials;
- production scope;
- authorization;
- approval logic;
- shell/browser/network capability;
- tenant boundaries;
- destructive behavior.

These are security-boundary changes.

---

## Documentation

The repository should document:

- available tools;
- side-effect classification;
- input schemas;
- output schemas;
- authn model;
- authz model;
- approval requirements;
- backend systems;
- environment scope;
- retry/idempotency semantics;
- known limitations.

Users should know exactly what authority the server exposes.

---

## Architecture decisions

Use ADRs for consequential choices such as:

- transport;
- identity model;
- authorization model;
- generic versus typed tool design;
- tool federation;
- approval mechanism;
- sandbox architecture;
- multi-tenant isolation.

Do not create ADRs for trivial tool additions.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic MCP tool server may resemble:

```text
.
├── AGENTS.md
├── README.md
├── src/
│   ├── server/
│   ├── tools/
│   ├── resources/
│   ├── auth/
│   ├── policy/
│   ├── approvals/
│   ├── backends/
│   └── audit/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── security/
│   └── contract/
├── schemas/
├── docs/
│   ├── tools/
│   ├── architecture/
│   └── adr/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

An MCP tool server repository should expose obvious commands or equivalent native interfaces for:

```text
check
test
test-authz
test-security
test-contract
test-integration
run
```

These names are illustrative.

Use the ecosystem's native tooling where appropriate.

---

## Acceptance criteria

An MCP server is not complete because a client can list tools and invoke one.

Demonstrate the applicable subset of the following.

### Tool design

- tools have explicit schemas;
- side-effect classes are defined;
- tool descriptions are accurate;
- broad generic tools are constrained or avoided.

### Authentication

- caller identity is established where required;
- unauthenticated access is rejected where required;
- credentials are not exposed to clients.

### Authorization

- allowed tool call succeeds;
- unauthorized tool call fails;
- resource scope is enforced;
- cross-tenant/environment access is denied.

### Approval

- high-impact operation requires approval where policy says so;
- approval is bound to exact operation parameters;
- stale/changed approval is rejected.

### Argument safety

- malformed arguments are rejected before backend access;
- path/URL/query arguments are constrained;
- unknown fields are handled intentionally.

### Backend safety

- timeouts exist;
- retry policy is bounded;
- unknown outcomes are represented correctly;
- idempotent retries are safe.

### Output safety

- secrets are redacted;
- outputs are typed where practical;
- external/tool result text is treated as untrusted;
- errors do not leak backend internals.

### Operations

- audit records exist;
- tool latency/errors are observable;
- rate/backpressure controls exist where needed;
- health signals work.

### Security

- at least one important deny-path test passes;
- at least one malicious-input test passes;
- prompt-injected external content cannot directly authorize a privileged write.

---

## Optional composition

Common combinations:

```text
mcp-tool-server + zero-trust-service
```

For shared or remote servers with explicit workload/user identity and fine-grained authorization.

```text
mcp-tool-server + software-supply-chain
```

For signed, verifiable privileged tool servers and versioned schemas.

```text
mcp-tool-server + security-tool
```

For scanners, analyzers, and security automation exposed through MCP.

```text
mcp-tool-server + hostile-input
```

For servers that ingest arbitrary files, URLs, repositories, or attacker-controlled content.

```text
mcp-tool-server + high-assurance
```

For stricter audit integrity, independent policy checks, and stronger approval boundaries.

```text
agentic-system + mcp-tool-server
```

For end-to-end agent architectures where the agent client and tool server are governed as separate trust domains.

---

## Anti-patterns

Avoid:

- one `run_anything` tool;
- arbitrary shell as the default interface;
- broad backend admin credentials;
- returning raw secrets;
- treating discovery as authorization;
- trusting user-supplied tenant IDs;
- combining read/write/delete into one opaque tool;
- approving vague actions;
- retrying timed-out writes blindly;
- exposing all tools to all clients;
- trusting tool output because it came through MCP;
- tool descriptions that hide side effects;
- unauthenticated remote servers with privileged backends;
- using localhost as the only trust boundary for privileged tools;
- allowing dynamic tool registration without policy;
- auto-trusting federated/third-party tool servers;
- leaking raw backend exceptions;
- global caches for protected results;
- treating MCP transport security as sufficient authorization.

Do not mistake protocol compliance for security.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. server purpose;
2. exposed tool/resource inventory;
3. side-effect classification;
4. caller identity model;
5. authorization model;
6. tenant/environment scoping;
7. approval model;
8. backend credential model;
9. argument-validation strategy;
10. retry/idempotency/unknown-outcome model;
11. audit model;
12. observability model;
13. secret-handling model;
14. integration/contract tests executed;
15. authorization/deny-path tests executed;
16. malicious-input/security tests executed;
17. commands actually run;
18. observed results;
19. unverified assumptions;
20. controls deliberately not implemented and why.

Never claim an MCP server is "safe" or "secure" merely because the protocol is
implemented correctly.

Describe the concrete authorization and enforcement boundaries.

---

## Guiding principle

An MCP server should expose capability, not ambient authority.

The client may discover a tool.

That does not mean the caller may use it.

The model may request an action.

That does not mean the server should perform it.

The server should validate.

Policy should authorize.

Credentials should stay behind the boundary.

Side effects should be explicit.

High-impact actions should be attributable.

And every tool should expose the smallest authority required to accomplish its intended task.
