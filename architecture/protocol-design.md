# protocol-design.md

> Architecture recipe for designing or evolving application, service, agent,
> control-plane, event, and machine-to-machine protocols.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with `platform-architecture.md`, `control-plane.md`,
> `api-platform.md`, `eventing-platform.md`, `identity-platform.md`,
> `mcp-tool-server.md`, `agentic-system.md`, `secure-service.md`,
> `library-sdk.md`, and profiles such as `zero-trust-service.md`,
> `software-supply-chain.md`, `high-assurance.md`, `hostile-input.md`,
> or `ai-security.md` where appropriate.
>
> This recipe defines architecture expectations for durable protocol contracts:
> message formats, framing, negotiation, versioning, state machines, ordering,
> idempotency, retries, flow control, authentication, authorization context,
> extensibility, error semantics, and compatibility.
>
> It does not require HTTP, gRPC, Protobuf, JSON, WebSockets, QUIC, TCP,
> MessagePack, CBOR, MCP, or any specific transport or serialization format.

## Purpose

Use this recipe when the system needs a protocol contract rather than merely
an API implementation.

Typical examples:

- service-to-service protocols;
- agent/tool protocols;
- plugin protocols;
- control-plane/data-plane protocols;
- event envelopes;
- device protocols;
- streaming protocols;
- replication protocols;
- management protocols;
- long-lived session protocols;
- multiplexed request/response protocols;
- peer-to-peer coordination;
- state synchronization;
- internal platform protocols.

The goal is not to invent a new wire format.

The goal is to define a contract where both sides can answer:

- Who speaks first?
- How is a peer identified?
- How is version compatibility established?
- How are messages framed?
- Which messages are legal in which states?
- What ordering is guaranteed?
- What happens when a message is duplicated?
- How are retries handled?
- How are errors represented?
- How is cancellation signaled?
- How is backpressure expressed?
- How are capabilities negotiated?
- How are unknown fields handled?
- What happens during partial upgrade?
- Which extensions are safe?
- What prevents downgrade, replay, spoofing, or resource exhaustion?
- How can an implementation recover after reconnect or restart?

---

## Core principle

A protocol is a state machine with compatibility rules.

A useful conceptual model is:

```text
Peer A
  |
  | transport
  v
Framing
  |
  v
Message Decode
  |
  v
Protocol State Machine
  |
  +--> authentication
  +--> capability/version checks
  +--> authorization context
  +--> sequencing
  +--> flow control
  +--> error handling
  |
  v
Application Semantics
  |
  v
Peer B
```

A serialization format is not a protocol.

A transport is not a protocol.

An endpoint list is not a protocol.

The protocol is the complete contract governing valid interaction over time.

---

## Architectural invariants

A protocol SHOULD satisfy these invariants unless there is a documented reason
not to.

### 1. Message boundaries are unambiguous

A receiver must know where one message ends and the next begins.

Do not rely on transport packet boundaries unless the transport guarantees and
exposes message semantics.

### 2. Versioning is explicit

Peers must know what version or capability set they can safely use.

Do not make compatibility depend on synchronized deployment.

### 3. State transitions are defined

For stateful protocols, valid message sequences should be explicit.

Do not let behavior depend on undocumented implementation timing.

### 4. Errors are part of the contract

Errors need stable machine-readable semantics.

Do not require peers to parse free-form text.

### 5. Ordering guarantees are scoped

Define whether ordering is:

- per connection;
- per stream;
- per resource;
- global;
- absent.

Do not imply stronger ordering than the protocol actually provides.

### 6. Idempotency is explicit

A sender must know whether retrying a request can duplicate effects.

### 7. Unknown input behavior is defined

Peers should know how to handle:

- unknown fields;
- unknown message types;
- unknown enum values;
- unknown capabilities.

### 8. Resource use is bounded

The protocol should not allow peers to force unbounded:

- memory;
- CPU;
- streams;
- messages;
- pending requests;
- decompression;
- recursion.

### 9. Authentication does not imply authorization

Peer identity should be separate from permission to perform protocol actions.

### 10. Downgrade behavior is safe

Negotiation must not silently move peers into a weaker security mode.

### 11. Reconnect and resume semantics are explicit

Long-lived protocols need clear behavior after:

- disconnect;
- timeout;
- restart;
- partial delivery.

### 12. Compatibility is tested, not assumed

Version matrices and golden protocol fixtures should prove supported
interoperability.

---

## Protocol scope

Before designing the protocol, define:

- participants;
- transport;
- trust boundary;
- session lifetime;
- connection model;
- message classes;
- request/response behavior;
- streaming;
- server push;
- authentication;
- compatibility window.

Avoid starting with field definitions before interaction semantics are clear.

---

## Participant roles

Identify roles explicitly.

Examples:

```text
client / server
controller / agent
producer / consumer
coordinator / worker
agent / tool server
leader / follower
peer / peer
```

Do not use symmetric terminology for asymmetric authority unless the protocol
is truly symmetric.

---

## Initiator

Define which participant initiates:

- transport;
- handshake;
- authentication;
- capability negotiation;
- session creation.

---

## Connection model

A protocol may use:

- one request per connection;
- persistent connection;
- multiplexed streams;
- connection pool;
- peer-to-peer sessions.

State which model applies.

---

## Session model

Distinguish:

```text
transport connection
```

from:

```text
logical protocol session
```

A session may survive transport reconnect.

Do not tie durable application identity solely to a socket.

---

## Transport selection

Select transport based on actual protocol requirements.

Consider:

- reliability;
- ordering;
- latency;
- multiplexing;
- browser compatibility;
- firewall traversal;
- streaming;
- datagram support;
- operational visibility.

Do not design a custom transport if an established one meets requirements.

---

## Reliable transports

TCP-like transports typically provide ordered byte streams.

They do not preserve application message boundaries.

---

## Datagram transports

Datagram transports preserve message boundaries but may require protocol-level:

- retransmission;
- ordering;
- fragmentation;
- congestion handling.

Use only when required.

---

## HTTP

HTTP can provide:

- request/response;
- intermediaries;
- auth integration;
- caching;
- observability.

Do not invent custom framing over raw TCP when HTTP semantics are sufficient.

---

## HTTP/2 and HTTP/3

Multiplexing can support concurrent streams.

Understand:

- stream lifecycle;
- cancellation;
- connection-level failure;
- flow control.

---

## WebSockets

Useful for long-lived bidirectional messaging.

Define application-level:

- framing;
- message types;
- heartbeat;
- auth lifetime;
- reconnect;
- backpressure.

---

## gRPC

Useful for typed RPC and streaming.

Still define:

- semantic versioning;
- deadlines;
- idempotency;
- errors;
- auth;
- compatibility.

Generated code does not eliminate protocol design.

---

## Custom transport

Use custom transport only for requirements established protocols cannot
reasonably satisfy.

Document why.

---

## Framing

Framing makes message boundaries unambiguous.

Common approaches:

- length prefix;
- delimiter;
- fixed header;
- transport-native message frame.

---

## Length-prefixed framing

A frame may be:

```text
length
type
flags
payload
```

Validate length before allocation.

Bound maximum frame size.

---

## Delimited framing

If delimiters are used:

- define escaping;
- maximum line/frame length;
- encoding.

Avoid unbounded reads waiting for a delimiter.

---

## Partial reads

Receivers must handle partial reads.

Do not assume one read call equals one frame.

---

## Coalesced reads

Receivers may receive multiple messages together.

Parse incrementally.

---

## Frame size

Set hard maximum frame size.

Use separate limits for:

- control frames;
- ordinary data;
- bulk transfer

if needed.

---

## Fragmentation

If the protocol fragments large messages:

- define fragment identity;
- ordering;
- timeout;
- maximum reassembled size.

Prevent fragment-based memory exhaustion.

---

## Compression

If compression is supported:

- negotiate explicitly;
- bound decompressed size;
- protect against decompression bombs.

Do not infer compression from payload content.

---

## Serialization

Choose serialization based on:

- compatibility;
- tooling;
- performance;
- readability;
- language ecosystem.

---

## Text formats

Examples:

- JSON;
- XML;
- text lines.

Advantages:

- debuggable;
- interoperable.

Costs:

- size;
- parsing;
- looser typing.

---

## Binary formats

Examples:

- Protobuf;
- CBOR;
- MessagePack;
- Cap'n Proto.

Advantages may include:

- compactness;
- typed schemas;
- speed.

Do not assume binary automatically means secure or efficient.

---

## Canonical encoding

Canonical serialization may be required for:

- signing;
- hashing;
- cache keys.

Define it precisely.

Do not sign ambiguous serialization.

---

## Character encoding

Use explicit text encoding.

Prefer UTF-8 where appropriate.

Reject invalid encoding deterministically.

---

## Unicode normalization

If identifiers are security-sensitive:

- define normalization;
- compare consistently.

Avoid visually confusable identifiers where risk warrants controls.

---

## Schema

Define message schemas explicitly.

Each message should have:

- type;
- required fields;
- optional fields;
- value ranges;
- extension behavior.

---

## Message type

Use stable identifiers.

Do not reuse an old message type for different semantics.

---

## Required fields

Be conservative about adding required fields.

Required-field additions are often breaking.

---

## Optional fields

Optional fields support evolution.

Define default semantics when absent.

---

## Unknown fields

For extensible protocols, receivers should often ignore unknown additive fields.

But security-sensitive messages may need stricter handling.

Document behavior per message class.

---

## Unknown message type

Possible behaviors:

- reject;
- ignore;
- send unsupported error;
- negotiate capability.

Do not leave behavior implementation-specific.

---

## Unknown enum values

Receivers should tolerate future enum values where forward compatibility
requires it.

Avoid exhaustive assumptions in consumers.

---

## Field reuse

Do not reuse removed field numbers/names where historical decoders might
misinterpret them.

---

## Reserved fields

Reserve removed identifiers where schema technology supports it.

---

## Message classes

Common classes:

```text
handshake
request
response
event
stream-data
ack
error
cancel
heartbeat
control
```

Keep them distinguishable.

---

## Correlation

Request/response protocols need stable correlation.

Use:

- request ID;
- stream ID;
- operation ID.

Do not rely only on message order when concurrency is allowed.

---

## Request ID

A request ID should be unique within its defined scope.

Define scope:

- connection;
- session;
- globally unique.

---

## Operation ID

For long-running side effects, use a durable operation identity separate from
transport request ID.

---

## Sequence number

Use sequence numbers where ordering, loss detection, or replay protection
requires them.

Define:

- scope;
- wrap behavior;
- reset;
- persistence.

---

## Session ID

If sessions survive reconnect, use durable session identity.

Do not trust caller-chosen session IDs without authorization.

---

## State machine

Stateful protocols should define explicit states.

Example:

```text
DISCONNECTED
    ->
CONNECTED
    ->
NEGOTIATING
    ->
AUTHENTICATED
    ->
READY
    ->
DRAINING
    ->
CLOSED
```

---

## Transition table

Document valid transitions.

Example:

| Current | Message | Next | Notes |
|---|---|---|---|
| CONNECTED | HELLO | NEGOTIATING | validates version |
| NEGOTIATING | AUTH | AUTHENTICATED | if successful |
| AUTHENTICATED | READY | READY | capabilities active |
| READY | CLOSE | DRAINING | no new work |
| DRAINING | CLOSE_ACK | CLOSED | session complete |

Invalid transitions should fail predictably.

---

## Unexpected messages

On message invalid for current state:

- reject;
- send protocol error;
- close session if necessary.

Do not silently reinterpret.

---

## Terminal states

Define which states are terminal.

Do not allow accidental resurrection of invalid/closed sessions without new
handshake.

---

## Handshake

A handshake may establish:

- protocol version;
- identity;
- capabilities;
- compression;
- session parameters;
- limits.

Keep handshake bounded.

---

## Version negotiation

Negotiation should identify mutually supported protocol version/capabilities.

Avoid a single mutable `"version": "latest"` value.

---

## Minimum and maximum version

Peers may advertise:

```text
min_supported
max_supported
```

Only if version ordering is meaningful.

---

## Version sets

For non-linear compatibility, advertise explicit version sets/capabilities.

---

## Downgrade attacks

An attacker may try to force peers to choose an older weaker version.

Where security matters:

- authenticate negotiation;
- remember minimum security level;
- reject unsafe downgrade.

---

## Security version floor

Define versions that are no longer acceptable regardless of peer support.

---

## Capability negotiation

Capabilities are often better than protocol-wide version bumps.

Examples:

```text
streaming
compression
batching
tool-cancel
resume
```

---

## Capability identifiers

Use stable identifiers.

Document:

- semantics;
- dependencies;
- conflicts.

---

## Capability dependencies

Some capabilities require others.

Example:

```text
resume-v2 requires session-state-v2
```

Do not negotiate incompatible combinations.

---

## Capability removal

Deprecate capability before removing it.

Track usage where possible.

---

## Authentication

Establish peer identity before privileged protocol actions.

Possible mechanisms:

- mTLS;
- token;
- signed handshake;
- workload identity;
- SASL-like exchange.

Do not invent cryptographic authentication when mature mechanisms exist.

---

## Authentication timing

Authenticate early enough to avoid expensive unauthenticated work.

But some protocols require initial negotiation before auth.

Bound pre-auth resource use tightly.

---

## Mutual authentication

Use where both sides need strong identity.

Do not assume server-authenticated TLS authenticates the client.

---

## Token authentication

Validate:

- issuer;
- audience;
- expiry;
- signature;
- relevant scope.

Compose with `identity-platform.md`.

---

## Authentication renewal

Long-lived sessions may outlive credential TTL.

Define:

- reauthentication;
- token refresh;
- session termination.

Do not let a 24-hour connection bypass a 5-minute credential expiry
indefinitely.

---

## Authorization

After authentication, authorize protocol actions.

Authorization may depend on:

- subject;
- message type;
- resource;
- tenant;
- capability;
- environment.

---

## Per-message authorization

Sensitive operations may require authorization on each request.

Do not authorize a session once and assume every future action is permitted.

---

## Session-level authorization

Coarse session grants may be appropriate for narrow protocols.

Keep scope explicit.

---

## Delegation

If one peer acts on behalf of another, preserve delegated authority.

Do not replace user identity with broad server identity silently.

---

## Tenant context

Derive tenant from trusted identity/session state where possible.

Do not accept arbitrary tenant fields as sole authorization evidence.

---

## Authorization changes

Long-lived sessions may outlive role changes.

Define:

- revalidation interval;
- revocation;
- session termination.

---

## Confidentiality

Use encrypted transport across untrusted boundaries.

Do not invent custom encryption if TLS or another established protocol fits.

---

## Integrity

Messages should have integrity protection from transport or protocol.

If messages are stored/forwarded beyond transport, independent signatures may
be needed.

---

## Message signing

If messages are signed:

- canonicalize;
- include context;
- prevent replay;
- identify key.

Do not sign only the payload while leaving security-critical headers mutable.

---

## Replay protection

Replay-sensitive messages may require:

- nonce;
- timestamp;
- sequence;
- session binding;
- operation ID.

---

## Nonce

A nonce should be unique within the threat model.

Bound replay-cache retention.

---

## Timestamp

Timestamp-based replay windows require reliable clocks.

Allow bounded skew.

---

## Binding

Bind authentication proof to:

- session;
- peer;
- channel;
- audience

where needed.

---

## Request semantics

Define whether each operation is:

- read-only;
- idempotent;
- non-idempotent;
- reversible;
- irreversible.

This should drive retry behavior.

---

## Idempotent request

An idempotent request can be repeated without additional intended effect.

Document the identity scope.

---

## Idempotency key

For non-naturally-idempotent operations, use client-generated stable keys where
appropriate.

Define:

- uniqueness scope;
- retention;
- conflict semantics.

---

## Duplicate request

If duplicate request ID arrives:

- return previous result;
- return operation status;
- reject conflict.

Do not execute blindly.

---

## Retry

Define which failures are retryable.

Typical retryable classes:

- temporary unavailable;
- overload;
- timeout before known side effect.

But unknown outcomes require observation, not blind retry.

---

## Retry backoff

Use bounded exponential backoff with jitter.

---

## Retry hints

Protocol errors may include:

- retryable boolean;
- retry-after;
- retry token;
- operation status endpoint.

---

## Unknown outcome

If connection drops after a side-effecting request:

```text
client does not know whether operation succeeded
```

Provide a recovery path:

- query by operation ID;
- reconcile resource state;
- deduplicate on retry.

---

## At-most-once request semantics

Can be approximated with durable deduplication.

Be precise about crash windows.

---

## At-least-once request semantics

Requires idempotent receiver behavior.

---

## Exactly-once caveat

Exactly-once side effects across distributed peers are difficult.

Do not claim it from request IDs alone.

---

## Ordering

Define ordering scope.

Examples:

```text
connection order
stream order
resource-key order
no order
```

---

## Concurrent requests

If concurrency is allowed, responses may arrive out of order.

Require correlation IDs.

---

## Head-of-line blocking

Multiplexing may reduce one stream blocking another.

But connection-level congestion can still exist.

---

## Reordering

Protocols using datagrams/multiple paths may reorder.

Receivers should buffer only within bounded windows.

---

## Causality

Some protocols need causal ordering.

Use explicit dependency/version data.

Do not infer causality solely from arrival time.

---

## Revisions

Resource protocols may use:

- generation;
- revision;
- sequence;
- ETag.

Use to prevent stale updates.

---

## Preconditions

Mutations may include:

```text
apply if revision == X
```

This protects against lost updates.

---

## Optimistic concurrency

On conflict:

- return current version;
- require recompute/retry.

---

## Cancellation

Cancellation should have protocol semantics.

Possible messages:

```text
CANCEL(request_id)
CANCEL_ACK(request_id, state)
```

---

## Cancellation race

The operation may complete while cancellation is in flight.

Define outcomes:

- canceled;
- already complete;
- too late;
- unknown.

---

## Cancellation and side effects

Cancellation does not imply rollback.

Document irreversible boundaries.

---

## Deadlines

Send absolute or relative deadline where useful.

Receivers should stop unnecessary work after expiry.

---

## Timeout

Timeout is local observation.

It does not prove peer did not complete work.

---

## Heartbeats

Heartbeats may detect liveness.

Define:

- interval;
- timeout;
- direction;
- idle semantics.

Do not use excessively aggressive heartbeat intervals.

---

## Keepalive

Transport keepalive and application heartbeat are different.

Use each for its intended layer.

---

## Lease

Some protocols grant time-bounded authority.

Define:

- lease ID;
- expiry;
- renewal;
- fencing.

---

## Fencing token

Use monotonic fencing tokens when stale holders must be prevented from acting.

---

## Split brain

Coordination protocols need explicit split-brain behavior.

Do not rely only on timeout assumptions if stale peers can still mutate shared
resources.

---

## Flow control

Flow control prevents a fast sender overwhelming a slow receiver.

Possible approaches:

- credit/window;
- bounded outstanding requests;
- receiver-advertised capacity;
- transport flow control.

---

## Application-level flow control

Transport flow control may not bound application work queues.

Use explicit application credits where needed.

---

## Window

A receiver may advertise:

```text
max in-flight messages
max bytes
```

---

## Acknowledgements

Define what an ACK means.

Possible meanings:

- frame received;
- message parsed;
- accepted for processing;
- side effect durable.

Do not leave ACK semantics ambiguous.

---

## Negative acknowledgements

NACK may indicate:

- retry later;
- permanent reject;
- invalid message.

Use explicit reason.

---

## Backpressure

Under pressure, sender should:

- pause;
- reduce concurrency;
- retry later;
- shed low-priority work.

Avoid unbounded buffering.

---

## Buffers

Bound all buffers.

Examples:

- inbound frames;
- outbound queue;
- pending requests;
- replay cache;
- fragments.

---

## Priority

If protocol supports priority:

- define classes;
- prevent starvation;
- prevent caller abuse.

Do not let untrusted peers mark all traffic critical.

---

## Rate limits

Bound:

- new sessions;
- messages/sec;
- bytes/sec;
- streams;
- expensive operations.

---

## Quotas

Persistent protocol relationships may need per-tenant quotas.

---

## Connection limits

Limit:

- connections per identity;
- connections per source;
- total concurrent sessions.

---

## Multiplexing

If multiple logical streams share one connection:

- identify stream;
- bound streams;
- isolate errors;
- define stream close/reset.

---

## Stream lifecycle

States may include:

```text
IDLE
OPEN
HALF_CLOSED
CLOSED
RESET
```

---

## Stream reset

A stream-level failure should not always kill the entire connection.

Define error scope.

---

## Connection-level errors

Reserve for conditions such as:

- framing corruption;
- auth failure;
- incompatible version;
- global state corruption.

---

## Error model

Use stable machine-readable error codes.

Example categories:

```text
INVALID_MESSAGE
UNSUPPORTED_VERSION
UNSUPPORTED_CAPABILITY
AUTHENTICATION_FAILED
AUTHORIZATION_DENIED
RESOURCE_NOT_FOUND
CONFLICT
RATE_LIMITED
OVERLOADED
TIMEOUT
CANCELED
RETRYABLE_ERROR
PERMANENT_ERROR
PROTOCOL_ERROR
INTERNAL_ERROR
```

---

## Error scope

An error should identify whether it applies to:

- message;
- request;
- stream;
- session;
- connection.

---

## Error details

Include structured details where useful.

Do not expose secrets or internal stack traces.

---

## Error text

Human-readable text may change.

Do not make clients depend on it.

---

## Fatal errors

Define which errors close the connection/session.

---

## Recoverable errors

A recoverable message error should not necessarily terminate healthy streams.

---

## Error retry semantics

Error codes should state whether retry may succeed.

Avoid generic `"ERROR"`.

---

## Redirect

If protocol supports redirect:

- authenticate destination;
- constrain schemes/hosts;
- prevent downgrade.

Do not follow arbitrary peer-provided destinations blindly.

---

## Discovery

Peers may discover endpoints through:

- DNS;
- registry;
- config;
- control plane.

Discovery does not establish identity.

---

## Endpoint identity

Authenticate the endpoint after discovery.

Do not trust DNS name resolution alone.

---

## Endpoint migration

If endpoints change, long-lived sessions should reconnect predictably.

---

## Resumption

Session resumption may reduce reconnect cost.

Define what state survives.

---

## Resume token

A resume token should be:

- authenticated;
- scoped;
- expiring;
- replay-safe.

---

## Resume position

For streams, specify:

- last acknowledged sequence;
- offset;
- revision.

---

## Duplicate on resume

Resume may cause duplicates.

Consumers must understand deduplication semantics.

---

## Gaps

If resume point is no longer available:

- fail explicitly;
- request snapshot/resync.

Do not silently skip data.

---

## Full resynchronization

Protocols that maintain state should have a full-resync path.

Example:

```text
snapshot
    +
incremental updates
```

---

## Snapshot consistency

Define whether snapshot represents a coherent point in time.

---

## Snapshot version

Bind subsequent deltas to snapshot revision.

---

## Delta protocol

Deltas should include:

- base revision;
- new revision;
- changed fields/items.

Reject deltas applied to wrong base unless conflict semantics exist.

---

## State convergence

For synchronization protocols, define convergence rules.

Do not assume eventual consistency without conflict resolution.

---

## Conflict resolution

Possible strategies:

- leader wins;
- revision compare;
- merge;
- CRDT;
- explicit conflict.

Choose according to domain.

---

## Versioning strategy

Separate:

```text
protocol version
message schema version
capability version
application resource version
```

They are not necessarily the same.

---

## Major protocol version

Use for incompatible interaction changes.

Do not bump major version for additive fields if compatibility remains.

---

## Minor capability evolution

Capabilities may evolve independently.

---

## Version identifiers

Use stable identifiers.

Avoid parsing arbitrary semantic ordering if not defined.

---

## Compatibility matrix

Document supported peer combinations.

Example:

| Client | Server | Status |
|---|---|---|
| v1 | v1 | supported |
| v1 | v2 | supported |
| v2 | v1 | limited |
| v2 | v2 | supported |

Actual matrix depends on protocol.

---

## Rolling upgrades

Design for old/new peer coexistence if deployment is not atomic.

---

## Upgrade sequencing

Define whether safe order is:

```text
servers first
clients second
```

or another sequence.

---

## Feature activation

A new capability may require all relevant peers upgraded before enabling it.

---

## Feature flags

Use temporary feature gates where migration requires.

Retire them.

---

## Downgrade

If downgrade is supported, define state compatibility.

New state may make old implementations unsafe.

---

## Data migration

Protocol-version changes may require persistent-state migration.

Treat it separately from wire compatibility.

---

## Deprecation

Deprecate:

- version;
- message;
- field;
- capability.

Provide:

- replacement;
- timeline;
- compatibility window.

---

## Retirement telemetry

Track usage of deprecated protocol features where possible.

---

## Extensibility

Design extension points intentionally.

Possible approaches:

- optional fields;
- extension namespace;
- capability IDs;
- vendor/private ranges.

---

## Extension namespace

Prevent collisions.

Examples:

```text
org.example.feature
vendor:1234
```

---

## Critical versus ignorable extensions

Some extensions must be understood for safe behavior.

Use an explicit critical flag/negotiation mechanism.

Do not silently ignore security-critical extensions.

---

## Vendor extensions

Vendor extensions may be useful.

Keep them from becoming accidental core contract.

---

## Reserved ranges

Reserve identifiers for:

- future standard use;
- experimental;
- vendor/private.

---

## Experimental features

Clearly mark unstable features.

Do not let experimental message types become permanent undocumented dependencies.

---

## Backward compatibility

Older receivers should tolerate supported newer senders according to declared
rules.

---

## Forward compatibility

New receivers should tolerate supported older peers.

---

## Unknown field preservation

Some proxy/forwarding protocols may need to preserve unknown fields.

If unknown fields are dropped, document it.

---

## Proxying

Protocol-aware proxies should define:

- which fields they inspect;
- which fields they modify;
- identity propagation;
- unknown extension behavior.

---

## Intermediaries

If intermediaries exist, determine whether end-to-end properties survive them.

Examples:

- encryption termination;
- identity;
- signatures;
- ordering.

---

## Gateways

A gateway can translate protocol versions.

Do not let translation hide incompatible semantics.

---

## Protocol translation

For translation, define:

- lossless mappings;
- lossy fields;
- unsupported operations.

---

## Security boundaries

Document:

- peer trust;
- transport trust;
- intermediaries;
- administrative channels.

---

## Management protocol

Keep management/admin protocol separate from ordinary data operations where
risk warrants.

---

## Admin operations

Examples:

- reconfigure;
- reset;
- dump state;
- change trust;
- rotate keys.

Require stronger authorization.

---

## Debug commands

Do not expose debug/introspection commands in production without authorization.

---

## Diagnostic protocol

If diagnostics are available, redact sensitive state.

---

## Resource exhaustion

Threat-model:

- oversized frame;
- too many streams;
- nested structures;
- compression bombs;
- auth handshake floods;
- slowloris behavior;
- pending request explosion.

---

## Parser hardening

Treat wire input as hostile.

Use:

- length bounds;
- recursion bounds;
- field-count bounds;
- integer-overflow checks.

Compose with `hostile-input.md`.

---

## Integer handling

Validate:

- signed/unsigned conversion;
- overflow;
- length arithmetic.

---

## Recursive structures

Bound nesting depth.

---

## Repeated fields

Bound element count.

---

## Strings

Bound length.

Validate encoding.

---

## Maps

Bound entries.

Avoid attacker-controlled hash collision pathologies where runtime is
susceptible.

---

## Decompression

Bound expanded size before allocation where possible.

---

## Authentication floods

Pre-auth handshake should be cheap.

Rate-limit expensive cryptographic work where necessary.

---

## Slow peer

Set:

- read timeout;
- write timeout;
- idle timeout.

Do not hold resources forever for peers sending one byte at a time.

---

## Slow consumer

Bound outgoing queue.

Disconnect or shed when peer cannot keep up.

---

## Resource ownership

Every protocol-created resource should have owner/session/tenant semantics.

---

## Orphaned resources

If client disconnects, define whether created resources:

- persist;
- expire;
- roll back.

---

## Lease-based resources

Use leases for temporary resources where appropriate.

---

## Garbage collection

Protocol-created temporary state should be reclaimable.

---

## Keepalive abuse

Bound heartbeat rates.

Reject peers flooding control frames.

---

## Priority inversion

If high-priority control frames share queues with data, ensure data floods
cannot block required control traffic.

---

## Control frames

Consider separate limits/queues for:

- cancel;
- heartbeat;
- flow-control updates.

---

## Security properties

State required properties explicitly.

Examples:

- server authentication;
- mutual authentication;
- confidentiality;
- integrity;
- replay resistance;
- forward secrecy;
- authorization.

Do not infer them from product names.

---

## Cryptographic agility

If crypto negotiation exists, avoid allowing unsafe algorithm downgrade.

Prefer small approved algorithm sets.

---

## Cipher negotiation

Use established transport libraries.

Do not implement custom cipher negotiation.

---

## Key rotation

Long-lived protocols need key/session rotation behavior.

---

## Session keys

If sessions derive keys, bind them to handshake identity and transcript.

---

## Channel binding

Where needed, bind higher-layer auth to transport channel.

---

## Mutual trust changes

If trust roots change while connected, define whether sessions remain valid.

---

## Revocation

Long-lived sessions should define response to revoked identity.

Options:

- terminate immediately;
- reauth on interval;
- expire naturally.

---

## Reauthorization

Authorization may need re-evaluation for long sessions.

---

## Confidential metadata

Even encrypted payloads may expose:

- message size;
- timing;
- frequency.

Do not claim total confidentiality if metadata remains visible.

---

## Padding

Use padding only if traffic analysis is in threat model.

Avoid unnecessary complexity.

---

## Error side channels

Authentication errors should not leak excessive account or key existence
information.

Balance diagnostics with security.

---

## Capability side channels

Capability negotiation may reveal software/version inventory.

Restrict if sensitive.

---

## Privacy

Protocols may expose identity and metadata.

Minimize what peers need.

---

## Logging

Log protocol events with:

- peer identity;
- version;
- message type;
- request/operation ID;
- error category.

Do not log full sensitive payloads by default.

---

## Wire dumps

Packet/frame dumps are powerful diagnostics and dangerous data copies.

Restrict and redact them.

---

## Tracing

For multiplexed or distributed protocols, trace:

- request;
- stream;
- operation;
- retries;
- peer transitions.

---

## Metrics

Useful metrics:

- connections;
- handshake failures;
- active streams;
- message rate;
- bytes;
- protocol errors;
- retry;
- flow-control stalls;
- version/capability usage.

---

## Cardinality

Do not put request IDs or peer-generated arbitrary values into metric labels.

---

## Audit

Audit privileged protocol actions separately from ordinary traffic logs.

---

## Protocol observability

A connection being open does not mean protocol is healthy.

Track:

- successful handshakes;
- useful message progress;
- queue/backpressure;
- heartbeat state.

---

## Liveness

Define liveness at protocol level.

Do not confuse TCP connection existence with peer health.

---

## Readiness

A peer may be connected but not ready.

State machine should represent that.

---

## Protocol SLOs

Potential SLOs:

- handshake success;
- request success;
- message delivery latency;
- session stability.

Use consumer requirements.

---

## Reconnect storms

After outage, peers may reconnect simultaneously.

Use:

- jitter;
- exponential backoff;
- connection limits.

---

## Thundering herd

Avoid synchronizing heartbeat/retry intervals across fleets.

---

## Load shedding

Reject or defer low-priority new sessions under overload.

---

## Connection draining

Before shutdown:

- stop accepting new work;
- signal drain if protocol supports;
- complete/cancel in-flight work;
- close predictably.

---

## Graceful shutdown

Define close handshake where needed.

---

## Abrupt disconnect

Peers must handle abrupt transport loss.

Never require perfect close for correctness.

---

## Half-open connection

Use keepalive/heartbeat/timeouts to detect dead peers where necessary.

---

## Network partition

Protocols coordinating authority need explicit partition behavior.

Do not assume connectivity failure means the remote side stopped acting.

---

## Offline operation

Some clients may operate offline.

Define conflict/resync behavior.

---

## Mobile/intermittent clients

If clients frequently disconnect:

- durable session state;
- resume;
- deduplication;
- backoff

may be important.

---

## Clock skew

If protocol uses time:

- define accepted skew;
- avoid using wall clock for ordering where monotonic sequence is better.

---

## Monotonic clocks

Use for local timeout measurement.

---

## Cross-system time

Do not assume timestamps from independent systems are strictly ordered.

---

## Capability discovery

A client may ask:

```text
what can you do?
```

Discovery should not imply authorization.

---

## Tool/protocol discovery

For agent/tool protocols, a tool may be discoverable but unavailable to a given
caller.

Compose with `mcp-tool-server.md`.

---

## Agent protocols

Agent protocols need particular attention to:

- instruction/data separation;
- tool schemas;
- capability negotiation;
- side-effect classification;
- approval;
- result provenance.

---

## MCP-like protocols

Protocol standardization does not replace security policy.

Authentication, authorization, audit, and safe tool design remain required.

---

## Control-plane protocols

Compose with `control-plane.md`.

Important properties include:

- desired-state revision;
- fencing;
- idempotent actions;
- reconnect/resync;
- durable operation identity.

---

## Event protocols

Compose with `eventing-platform.md`.

Important properties include:

- event identity;
- schema;
- ordering;
- replay;
- delivery semantics.

---

## API protocols

Compose with `api-platform.md`.

HTTP/gRPC APIs are protocols and should document:

- idempotency;
- timeout;
- errors;
- versioning;
- authorization.

---

## Plugin protocols

Plugin protocols should define:

- lifecycle;
- capabilities;
- compatibility;
- sandbox boundary;
- failure isolation.

Do not give plugins unrestricted host access because the wire protocol is
typed.

---

## Device protocols

Device protocols may require:

- constrained bandwidth;
- intermittent connectivity;
- key rotation;
- firmware version skew;
- physical compromise assumptions.

---

## Firmware/protocol compatibility

Do not require simultaneous fleet upgrade.

Maintain compatibility windows.

---

## Peer identity rotation

Long-lived devices/services should support identity/key rotation without full
reprovisioning where possible.

---

## Protocol formalization

For complex stateful protocols, formal modeling may be justified.

Possible approaches:

- state transition tables;
- property tests;
- TLA+/PlusCal-like models;
- model checking.

Use where concurrency/safety risk warrants it.

---

## Safety properties

Examples:

- unauthorized peer never reaches READY;
- stale leader never commits after newer epoch;
- canceled request never starts after cancellation acknowledged;
- duplicate operation ID never creates duplicate resource.

---

## Liveness properties

Examples:

- valid request eventually completes or returns terminal error;
- healthy peers eventually reconnect;
- flow-controlled sender eventually resumes when credits return.

---

## Invariants

Write protocol invariants in plain language even if formal tools are not used.

---

## Implementation guidance

Separate:

- frame codec;
- message schema;
- state machine;
- transport adapter;
- application handler.

This improves testing and portability.

---

## Parser isolation

Keep parsing separate from side effects.

Do not execute application logic while message is only partially validated.

---

## Validation order

A useful order:

```text
frame bounds
    ->
decode
    ->
schema
    ->
state-machine legality
    ->
authentication/authorization
    ->
application semantics
```

Exact order may vary.

---

## Code generation

Generated codecs can reduce parser bugs.

Still test unknown/malformed input.

---

## Hand-written parsers

Use only where necessary.

Prefer memory-safe languages/libraries for hostile protocols where feasible.

---

## Zero-copy parsing

Use only when performance requires it.

Do not sacrifice validation clarity prematurely.

---

## Allocation limits

Preflight declared lengths before allocating.

---

## Memory ownership

For high-performance protocols, define buffer lifetime carefully.

Avoid use-after-free and aliasing bugs.

---

## Concurrency model

Define whether handlers run:

- serially;
- per stream;
- parallel;
- actor/event-loop.

Protocol semantics should not depend accidentally on scheduler timing.

---

## Per-resource serialization

Serialize operations where resource-level ordering matters.

---

## Locking

Avoid holding global locks across network I/O.

---

## Reentrancy

If callbacks may invoke protocol methods recursively, define supported behavior.

---

## Thread safety

SDK/client libraries should document concurrency guarantees.

Compose with `library-sdk.md`.

---

## Client library

A protocol client library should expose:

- connection lifecycle;
- timeout;
- cancellation;
- retries;
- errors;
- streaming.

Do not hide important protocol semantics behind magical defaults.

---

## Server implementation

Server should:

- bound work;
- validate messages;
- isolate sessions;
- expose metrics;
- fail safely.

---

## Proxy implementation

A proxy must preserve protocol semantics.

Do not silently downgrade auth, versions, or error behavior.

---

## Protocol gateway

Gateways translating legacy/new protocol should expose translation loss.

---

## Backward compatibility testing

Run old client against new server.

---

## Forward compatibility testing

Run new client against old server within supported matrix.

---

## Golden wire fixtures

Keep versioned fixtures of:

- handshake;
- messages;
- errors;
- edge cases.

Use them to detect accidental encoding changes.

---

## Round-trip tests

Encode → decode should preserve semantics.

---

## Differential tests

If multiple implementations exist, compare behavior on same corpus.

---

## Parser fuzzing

Fuzz:

- frame parser;
- message decoder;
- compression;
- extension parsing.

This is strongly recommended for exposed binary protocols.

---

## State-machine fuzzing

Generate random legal/illegal message sequences.

Assert:

- no crash;
- no unauthorized transition;
- bounded resources.

---

## Property-based testing

Useful properties:

- decode(encode(x)) preserves supported values;
- duplicate idempotent request does not duplicate effect;
- unsupported capability never activates;
- invalid transition never invokes handler;
- frame length above max is rejected before large allocation.

---

## Mutation testing

For high-assurance protocol validators, mutation testing may reveal weak tests.

---

## Load testing

Test:

- connections;
- streams;
- messages/sec;
- large valid frames;
- slow peers.

---

## Soak testing

Long-running sessions can reveal:

- leaks;
- request-ID wrap bugs;
- keepalive issues;
- stale authorization;
- queue growth.

---

## Network fault testing

Inject:

- packet loss;
- delay;
- duplication;
- reordering if transport allows;
- disconnect;
- half-open connections.

---

## Reconnect tests

Drop connection mid-request.

Verify documented resume/unknown-outcome semantics.

---

## Upgrade tests

Exercise supported mixed-version peers.

---

## Downgrade tests

Attempt negotiation to deprecated/insecure versions.

Expect rejection where policy requires.

---

## Replay tests

Replay captured authentication/control messages.

Ensure replay-sensitive operations reject them.

---

## Resource exhaustion tests

Test:

- oversized declared length;
- many tiny frames;
- deep nesting;
- many streams;
- slow sender;
- compression bomb.

---

## Authentication tests

Test:

- valid identity;
- expired credential;
- wrong audience;
- revoked credential;
- unauthenticated pre-auth behavior.

---

## Authorization tests

Test:

- allowed message;
- forbidden message;
- wrong resource;
- wrong tenant;
- admin-only operation.

---

## Error compatibility tests

Old clients should handle new error detail according to contract.

---

## Cancellation tests

Test races:

- cancel before start;
- cancel during work;
- completion before cancel;
- disconnect after cancel.

---

## Flow-control tests

Verify sender respects advertised limits.

---

## Backpressure tests

Slow receiver intentionally.

Ensure bounded memory.

---

## Fragment tests

Test missing, duplicate, overlapping, oversized fragments if fragmentation is
supported.

---

## Conformance suite

For protocols with multiple implementations, build a conformance suite.

It should test:

- handshake;
- versions;
- messages;
- errors;
- state transitions;
- security;
- limits.

---

## Interoperability matrix

Record implementation/version combinations actually tested.

Do not claim protocol portability from one implementation.

---

## Reference implementation

A reference implementation can clarify ambiguous behavior.

But written protocol semantics remain authoritative.

---

## Specification

The protocol specification should define:

- terminology;
- transport assumptions;
- framing;
- schemas;
- state machine;
- negotiation;
- errors;
- security;
- compatibility;
- limits.

---

## Normative language

Use:

- MUST;
- MUST NOT;
- SHOULD;
- SHOULD NOT;
- MAY

consistently if formal specification style is useful.

---

## Examples

Include example exchanges.

Examples are illustrative, not normative unless explicitly stated.

---

## ABNF / formal grammar

Use grammar notation where text syntax needs precision.

---

## Binary layout diagrams

For binary protocols, document byte order and field widths.

Example:

```text
0               1               2               3
+---------------+---------------+---------------+---------------+
| Version       | Type          | Flags         | Reserved      |
+---------------+---------------+---------------+---------------+
|                         Payload Length                        |
+---------------------------------------------------------------+
```

Only if relevant.

---

## Byte order

Specify endianness.

Do not leave it platform-native.

---

## Alignment

If wire layout has padding/alignment, specify exactly.

Prefer encoding independent of compiler struct layout.

---

## Reserved bits

Receivers should validate reserved bits according to compatibility policy.

---

## Checksums

Use checksums for accidental corruption where transport/storage does not
already provide sufficient integrity.

Do not use non-cryptographic checksum as authenticity proof.

---

## Magic values

A magic/version prefix can help detect misrouted traffic.

It is not authentication.

---

## Content type

For extensible payloads, identify content encoding/type explicitly.

---

## Negotiated limits

Peers may negotiate:

- frame size;
- concurrent streams;
- compression.

Use min of safe local/remote limits.

Do not let peer increase local hard maximum.

---

## Hard limits

Local safety limits override negotiation.

---

## Soft limits

Soft limits may be dynamically adjusted.

Document behavior.

---

## Protocol governance

For shared enterprise/platform protocols, define who owns:

- specification;
- extension registry;
- versioning;
- conformance tests;
- deprecation.

---

## Change process

Classify changes:

```text
non-wire implementation
compatible wire change
capability addition
behavioral change
breaking protocol change
security emergency
```

---

## Extension review

Review extensions for:

- collision;
- security;
- compatibility;
- implementation burden.

Avoid central review of trivial local behavior.

---

## Assigned identifiers

Maintain registry for:

- message types;
- error codes;
- capabilities;
- extensions.

---

## Identifier reuse

Never reuse retired security-sensitive identifiers.

---

## Security response

If a protocol version is found insecure:

- define minimum safe version;
- update negotiation policy;
- notify consumers;
- retire vulnerable path.

---

## Deprecation telemetry

Observe remaining old-version usage.

---

## Migration strategy

A protocol migration may use:

```text
add new version/capability
    ->
deploy compatible receivers
    ->
deploy senders
    ->
enable feature
    ->
observe
    ->
retire old
```

---

## Dual-stack protocol

Running old/new protocol in parallel may simplify migration.

Bound duration.

---

## Translation bridge

A bridge can support legacy peers.

Treat it as temporary architecture where possible.

---

## Protocol ossification

Too many intermediaries depending on accidental behavior can make protocol
evolution hard.

Keep contracts explicit.

---

## Unknown-extension tolerance

Forward compatibility often depends on receivers tolerating what they do not
understand safely.

---

## Greasing

For widely deployed extensible protocols, periodically exercising reserved/
unknown values can reduce ossification.

Use only where ecosystem scale justifies it.

---

## Observed behavior versus specified behavior

Do not standardize accidental implementation quirks without deliberate choice.

---

## Security review triggers

Require stronger review for changes to:

- handshake;
- authentication;
- version negotiation;
- message signing;
- replay protection;
- admin messages;
- parser/framing;
- compression;
- capability downgrade.

---

## Threat model

At minimum consider:

- unauthenticated peer;
- malicious authenticated peer;
- replay;
- downgrade;
- resource exhaustion;
- parser confusion;
- state-machine confusion;
- spoofed identity;
- stale authorization;
- extension abuse;
- compromised intermediary;
- protocol smuggling.

---

## Protocol smuggling

If protocol is tunneled through intermediaries, inconsistent parsing can create
security gaps.

Keep framing and normalization consistent across layers.

---

## Cross-protocol confusion

Prevent credentials/messages from one protocol context being accepted in
another.

Bind:

- audience;
- protocol ID;
- channel.

---

## Reflection/amplification

Datagram/public protocols should consider reflection attacks.

Avoid unauthenticated requests causing much larger responses.

---

## Oracle behavior

Authentication/crypto errors can become side channels.

Keep sensitive validation errors appropriately uniform.

---

## Parser differential

Different implementations may parse malformed input differently.

Use conformance/fuzzing to reduce ambiguity.

---

## Downgrade confusion

Do not negotiate security-critical behavior based only on unauthenticated
client preference.

---

## Extension confusion

Unknown critical extensions must not be silently ignored.

---

## Request confusion

Bind response to request/operation identity.

Do not accept mismatched correlation.

---

## Credential forwarding

Do not forward end-user bearer credentials between peers unless explicitly
designed.

Prefer audience-specific delegation.

---

## Protocol secrets

Never place long-term secrets in ordinary protocol messages unless encrypted
and required.

Prefer proof/derived credentials.

---

## Supply chain

Compose with `software-supply-chain.md`.

Protect:

- codec libraries;
- protocol generators;
- cryptographic libraries;
- reference implementations;
- schema compilers.

---

## Protocol dependency review

Parsing/crypto dependencies are high-risk.

Keep versions intentional.

---

## Generated code

Pin code generators where wire compatibility depends on output.

---

## Reproducible fixtures

Keep canonical generated fixtures under version control if useful.

---

## Operational tooling

Provide tools for:

- decode frame;
- inspect handshake;
- verify version;
- simulate peer;
- validate capture.

Keep them safe.

---

## Protocol CLI

Compose with `cli.md` if a diagnostic client exists.

Do not require packet captures for ordinary troubleshooting.

---

## Support bundle

A support bundle may include:

- negotiated version;
- peer identity;
- capability set;
- recent error codes.

Redact secrets.

---

## Runbooks

Create runbooks for:

- handshake failure;
- version mismatch;
- reconnect storm;
- stuck stream;
- auth failure spike;
- protocol error spike;
- upgrade incompatibility.

---

## Documentation views

Useful diagrams include:

### Session lifecycle

```text
CONNECT
  ->
NEGOTIATE
  ->
AUTHENTICATE
  ->
READY
  ->
DRAIN
  ->
CLOSE
```

### Request lifecycle

```text
REQUEST
  ->
VALIDATE
  ->
AUTHORIZE
  ->
PROCESS
  ->
RESPONSE
```

### Resume

```text
DISCONNECT
  ->
RECONNECT
  ->
AUTH
  ->
RESUME(position)
  ->
DELTA/SNAPSHOT
```

### Version negotiation

```text
Client versions/caps
      |
      v
Intersection + policy floor
      |
      v
Negotiated version/caps
```

---

## Architecture decisions

Use ADRs for consequential decisions such as:

- transport;
- framing;
- serialization;
- handshake;
- version negotiation;
- capability model;
- idempotency;
- ordering;
- replay protection;
- resume semantics.

---

## Recommended repository shape

Follow existing repository conventions first.

A protocol-focused repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── spec/
│   ├── protocol.md
│   ├── state-machine.md
│   ├── security.md
│   ├── compatibility.md
│   └── registries/
├── schemas/
├── src/
│   ├── framing/
│   ├── codec/
│   ├── state/
│   ├── auth/
│   └── transport/
├── fixtures/
│   ├── v1/
│   └── malformed/
├── conformance/
├── tests/
│   ├── unit/
│   ├── compatibility/
│   ├── conformance/
│   ├── security/
│   ├── fuzz/
│   ├── failure/
│   └── interoperability/
├── docs/
│   ├── migration/
│   ├── operations/
│   └── runbooks/
└── <ecosystem build/dependency files>
```

Only create directories that contain meaningful content.

---

## Verification interface

A protocol project should expose obvious commands or equivalent native
interfaces for:

```text
check
test
test-wire
test-state-machine
test-compatibility
test-conformance
test-security
test-interoperability
fuzz
test-failure
verify
```

These names are illustrative.

Use only commands the repository can actually implement.

---

## Acceptance criteria

A protocol is not complete because two current implementations can exchange one
happy-path message.

Demonstrate the applicable subset of the following.

### Framing and schema

- message boundaries are unambiguous;
- frame/message size is bounded;
- unknown fields/messages have defined behavior;
- malformed input is rejected safely.

### State machine

- valid states/transitions are documented;
- illegal messages do not invoke application behavior;
- close/reconnect behavior is defined;
- long-lived sessions handle credential expiry.

### Compatibility

- version negotiation exists where required;
- downgrade rules are explicit;
- mixed-version interoperability is tested;
- deprecation/migration process exists.

### Reliability

- retry semantics are documented;
- idempotency is tested;
- unknown outcomes have recovery path;
- cancellation and deadlines have defined races;
- reconnect/resume does not silently lose or duplicate state beyond contract.

### Ordering and flow control

- ordering scope is explicit;
- correlation IDs exist for concurrent requests;
- buffers are bounded;
- slow peers trigger backpressure rather than unbounded memory growth.

### Security

- authentication is established before privileged actions;
- authorization is action/resource-aware where needed;
- replay-sensitive messages are protected;
- unsafe downgrade is rejected;
- parser/resource-exhaustion cases are tested.

### Observability

- negotiated version/capabilities are visible;
- protocol errors are machine-classified;
- connection/stream health is measurable;
- sensitive payloads are not logged by default.

### Evolution

- extension namespace exists where needed;
- critical/ignorable extensions are distinguishable;
- identifier registries prevent collisions;
- removed fields/IDs are not dangerously reused.

---

## Optional composition

Common combinations:

```text
platform-architecture + protocol-design
```

For defining durable platform contracts between independently evolving
components.

```text
control-plane + protocol-design
```

For controller/agent, desired-state, fencing, reconnect, and reconciliation
protocols.

```text
api-platform + protocol-design
```

For HTTP/gRPC contracts with explicit retries, idempotency, versioning, and
state semantics.

```text
eventing-platform + protocol-design
```

For message envelopes, delivery semantics, ordering, replay, and schema
evolution.

```text
mcp-tool-server + protocol-design
```

For tool discovery/invocation protocols with explicit capability and security
boundaries.

```text
agentic-system + protocol-design
```

For agent-to-agent or agent-to-tool protocols with identity, approval, and
bounded authority.

```text
protocol-design + zero-trust-service
```

For authenticated peers, scoped authorization, replay resistance, and explicit
trust boundaries.

```text
protocol-design + high-assurance
```

For formal invariants, conformance testing, fuzzing, fault injection, and
mixed-version verification.

---

## Anti-patterns

Avoid:

- calling a serialization format a protocol;
- assuming TCP packet boundaries are message boundaries;
- no maximum frame size;
- one mutable `"latest"` protocol version;
- synchronized upgrade required for every change;
- authentication performed only after expensive parsing/work;
- valid identity treated as blanket authorization;
- free-form string errors as the client contract;
- retry semantics left undocumented;
- non-idempotent requests automatically retried;
- timeout treated as proof no side effect occurred;
- request correlation based only on message order when concurrency exists;
- unbounded pending-request maps;
- unbounded stream count;
- compression without decompressed-size limits;
- old insecure protocol version negotiated silently;
- long-lived sessions that ignore revoked credentials forever;
- unknown critical extensions silently ignored;
- implementation-native struct layout used directly on wire;
- global ordering assumed from per-stream ordering;
- reconnect that silently skips data;
- session resumption tokens with no scope/expiry;
- one malformed stream killing every multiplexed connection without need;
- only current-client/current-server interoperability tested;
- protocol behavior inferred from code with no written contract.

Do not mistake "it works between these two binaries today" for a durable
protocol.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. protocol purpose and participant roles;
2. transport and connection/session model;
3. framing and serialization model;
4. message/schema model;
5. state machine;
6. authentication model;
7. authorization/delegation model;
8. version/capability negotiation model;
9. ordering/correlation model;
10. idempotency/retry/unknown-outcome model;
11. cancellation/deadline/resume model;
12. flow-control/backpressure/resource-limit model;
13. replay/downgrade/security model;
14. extension/deprecation/migration model;
15. observability/error model;
16. wire/state/compatibility tests executed;
17. conformance/security/fuzz/failure tests executed;
18. commands actually run;
19. observed interoperability results;
20. unverified assumptions and deliberate omissions.

Never claim a protocol is "backward compatible", "secure", "reliable",
"exactly once", "self-healing", or "extensible" merely because it uses TLS,
Protobuf, gRPC, HTTP, sequence numbers, retries, or generated clients.

Describe the actual wire contract, state machine, negotiation, resource bounds,
security properties, error semantics, mixed-version behavior, and evidence.

---

## Guiding principle

A good protocol should make interaction boring even when peers are different
versions, networks fail, messages arrive twice, credentials expire, and one
side behaves badly.

Framing should be unambiguous.

States should be explicit.

Identity should be established.

Authority should be scoped.

Versions should negotiate safely.

Retries should not duplicate effects.

Timeouts should not lie about outcomes.

Unknown fields should have defined meaning.

Buffers should be bounded.

Downgrades should not weaken security.

Reconnect should recover deliberately.

And the protocol should be explainable as:

> These peers, speaking these compatible capabilities over this authenticated
> channel, may exchange these messages in these states under these ordering,
> retry, flow-control, and error semantics—and when either peer, the network,
> or the version boundary fails, the resulting behavior is still defined.
