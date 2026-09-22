# cli.md

> Recipe for scaffolding or elevating a command-line interface.
>
> Apply under `AGENTS.md` and either `SCAFFOLD.md` or `ELEVATE.md`.
> Compose with profiles such as `software-supply-chain.md`,
> `hostile-input.md`, `high-assurance.md`, `zero-trust-service.md`, or
> `ai-security.md` where appropriate.
>
> This recipe defines command structure, UX, compatibility, configuration,
> security, scripting, packaging, observability, testing, and release
> expectations for software primarily consumed through a terminal.
>
> It does not require a specific programming language, CLI framework,
> package manager, shell, operating system, or distribution channel.

## Purpose

Use this recipe when the repository primarily produces a command-line program.

Typical examples:

- developer tools;
- security scanners;
- administration CLIs;
- deployment tools;
- repository tools;
- platform clients;
- data-processing utilities;
- automation commands;
- package/build tools;
- AI-assisted developer CLIs.

The goal is not to create an impressive command tree.

The goal is to build a CLI that is:

- predictable for humans;
- stable for scripts;
- explicit about side effects;
- composable with shell tooling;
- secure with credentials and sensitive output;
- easy to install and upgrade;
- testable without depending on a human terminal;
- difficult to misuse accidentally.

---

## Core principle

A CLI has two audiences:

```text
human operator
    +
automation
```

Design for both.

Humans need:

- discoverability;
- useful errors;
- safe defaults;
- confirmation for dangerous actions.

Automation needs:

- stable exit codes;
- deterministic machine-readable output;
- no interactive surprises;
- predictable stdout/stderr behavior;
- backwards-compatible flags and commands.

Do not optimize one audience by breaking the other.

---

## Architectural invariants

A CLI SHOULD satisfy these invariants unless the repository has a documented
reason not to.

### 1. Exit codes are part of the API

Define exit behavior intentionally.

At minimum, callers should be able to distinguish:

```text
success
user/input error
operational failure
policy/security denial
```

when those distinctions matter.

Do not return `0` after a failed operation.

### 2. Stdout and stderr have distinct purposes

Prefer:

```text
stdout = requested result / machine-consumable output
stderr = diagnostics / progress / warnings
```

This keeps pipelines usable.

Do not mix decorative progress text into JSON output.

### 3. Machine-readable output is first-class

If users are likely to automate the CLI, provide a stable structured format.

Examples:

- JSON;
- NDJSON;
- CSV where appropriate.

Do not force scripts to scrape human prose.

### 4. Dangerous actions are explicit

Destructive or high-impact commands should require stronger intent.

Possible controls:

- explicit verb;
- confirmation;
- `--yes`/`--force` override;
- dry run;
- exact target display.

Do not hide destructive behavior behind generic commands.

### 5. Non-interactive mode is reliable

A CLI running in CI or a script must not unexpectedly block for input.

Detect or expose non-interactive behavior explicitly.

### 6. Configuration precedence is deterministic

Define precedence such as:

```text
flags
    >
environment
    >
config file
    >
defaults
```

unless project requirements specify otherwise.

Do not let multiple config sources silently fight.

### 7. Secrets are never casually printed

Do not expose:

- API keys;
- tokens;
- private keys;
- passwords

in normal output, debug logs, stack traces, or shell command echoes.

### 8. Network behavior is bounded

For networked CLIs:

- set timeouts;
- bound retries;
- validate TLS;
- support cancellation.

Do not hang indefinitely on unreachable services.

### 9. Commands are composable

A CLI should work in:

- terminals;
- pipes;
- CI;
- scripts;
- cron;
- containers.

Do not depend on cursor movement, color, or interactivity for correctness.

### 10. Compatibility matters

Command names, flags, output schemas, and exit codes are public contracts.

Change them deliberately.

---

## CLI boundary definition

Before implementation, identify:

- primary user;
- automation use cases;
- command tree;
- stdin usage;
- stdout contract;
- stderr contract;
- exit-code contract;
- config sources;
- authentication model;
- destructive operations;
- remote dependencies;
- packaging/install path.

A CLI should have a clear conceptual flow:

```text
argv / stdin
    |
    v
parse
    |
    v
validate
    |
    v
resolve config
    |
    v
execute command
    |
    v
format result
    |
    +--> stdout
    |
    +--> stderr
    |
    +--> exit code
```

---

## Command design

Prefer a shallow, coherent command tree.

Examples:

```text
tool scan
tool login
tool config get
tool resource list
tool resource create
```

Avoid deeply nested trees unless the domain genuinely requires them.

Do not encode implementation structure directly into commands.

---

## Command naming

Use consistent verbs and nouns.

Prefer established verbs such as:

```text
get
list
create
update
delete
show
scan
validate
apply
diff
login
logout
```

Avoid multiple synonyms for the same operation.

---

## Positional arguments

Use positional arguments for required, natural primary operands.

Example:

```text
tool show RESOURCE
```

Use flags for optional modifiers.

Avoid long ambiguous positional sequences.

---

## Flags

Flags should:

- have clear names;
- be stable;
- have documented defaults;
- behave consistently across commands.

Prefer long forms for clarity.

Short forms are useful for common options.

Do not reuse the same short flag for incompatible meanings across commands.

---

## Boolean flags

Prefer explicit positive behavior.

Example:

```text
--json
--quiet
--dry-run
```

For dangerous inversions, consider explicit names such as:

```text
--no-verify
```

and make insecure behavior visibly exceptional.

---

## Repeated flags

If repeated flags are supported, define semantics.

Example:

```text
--label a --label b
```

should mean something predictable.

Do not silently use only the last value unless documented.

---

## Flag precedence

If the same setting appears in multiple config sources, precedence must be
deterministic.

Document it.

---

## Unknown flags

Reject unknown flags.

Do not silently ignore typos.

---

## Deprecated flags

For deprecated flags:

- continue supporting them for a defined period where practical;
- emit useful warnings to stderr;
- document replacement.

Do not change meaning silently.

---

## Subcommand compatibility

Treat renamed or removed commands as breaking changes.

If aliases are provided for migration, keep them documented and time-bounded.

---

## Help

Every command should provide useful `--help`.

Help should include:

- summary;
- usage;
- arguments;
- important flags;
- examples where helpful.

Do not require network access for help.

---

## Version

Provide a version command or flag where ecosystem norms support it.

Useful output may include:

- CLI version;
- build revision;
- optional protocol/API version.

Do not dump enormous dependency inventories into ordinary version output.

---

## Shell completion

Provide shell completion when the user base benefits from it.

Do not make completion mandatory for usability.

Generated completion should stay synchronized with the command tree.

---

## Discoverability

A new user should be able to discover the main workflow from:

```text
tool --help
```

Do not require documentation archaeology for common tasks.

---

## Interactive versus non-interactive behavior

Detect TTY use where useful.

Interactive mode may offer:

- prompts;
- colors;
- progress;
- confirmations.

Non-interactive mode should:

- never block unexpectedly;
- avoid ANSI control codes;
- produce stable output.

Do not infer authorization from interactivity.

---

## Confirmation

Use confirmation for destructive or difficult-to-reverse actions where
appropriate.

A confirmation prompt should display:

- exact action;
- exact target;
- environment if relevant.

Prefer:

```text
Delete production database "orders-prod"? [y/N]
```

over:

```text
Continue? [y/N]
```

---

## Non-interactive confirmation

Automation should require an explicit override.

Examples:

```text
--yes
--force
--confirm TARGET
```

Choose semantics appropriate to risk.

Do not auto-confirm merely because stdin is not a TTY.

---

## Dry run

Provide `--dry-run` or equivalent where previewing meaningful side effects is
valuable.

Dry run should show intended actions without performing them.

Do not call a mode dry-run if it still performs irreversible external writes.

---

## Diff / plan

For configuration or infrastructure-like CLIs, show a plan/diff before
high-impact mutation where practical.

The output should distinguish:

- create;
- update;
- delete;
- unchanged.

---

## Exit codes

Define a small stable set.

A generic model may be:

```text
0 = success
1 = requested operation completed with negative/result condition
2 = invalid invocation/configuration
3 = operational/internal failure
4 = authorization/policy denial
```

This is illustrative.

Use ecosystem/domain conventions when established.

Do not expose dozens of undocumented exit codes.

---

## Exit code documentation

Document exit codes intended for automation.

Human-readable errors are not enough.

---

## Partial success

If a command processes multiple items, define partial-success semantics.

Options include:

- nonzero exit if any item fails;
- distinct partial-success exit;
- machine-readable per-item status.

Do not print "success" when half the work failed.

---

## Stdout

Stdout should contain the requested result.

For structured modes, output only the structure.

Example:

```bash
tool list --output json | jq .
```

must not fail because banners or progress were added.

---

## Stderr

Use stderr for:

- warnings;
- progress;
- diagnostics;
- human error details.

Structured automation can redirect it separately.

---

## Quiet mode

A quiet mode may suppress nonessential diagnostics.

Do not suppress errors that scripts need to diagnose unless the exit code still
clearly signals failure.

---

## Verbose/debug mode

Debug mode may include:

- request IDs;
- endpoint;
- timing;
- retry decisions.

It must not expose secrets.

Do not dump raw authorization headers.

---

## Color

Use color only for human-facing output.

Support:

- no-color conventions;
- non-TTY suppression;
- explicit override where useful.

Correctness must not depend on color.

---

## Progress indicators

Use progress bars/spinners only in interactive output.

Do not emit cursor-control characters in CI logs or structured output.

---

## Tables

Tables are useful for humans.

Machine automation should have structured output.

Do not make scripts parse columns whose spacing may change.

---

## JSON output

If JSON is supported:

- define stable field names;
- use valid JSON;
- avoid comments/banners;
- preserve types;
- document breaking changes.

Do not serialize all values as strings for convenience.

---

## NDJSON

Use newline-delimited JSON for streaming or large result sets where useful.

Each line should be independently valid JSON.

---

## CSV

Use CSV only where flat tabular data is natural.

Do not force nested domain objects into lossy CSV output.

---

## Raw output

A `--raw` mode may be useful for single scalar or payload results.

Keep its semantics precise.

---

## Output schema

Treat machine-readable output as an API.

Version it if long-lived consumers depend on it.

Do not silently rename fields in a patch release if scripts rely on them.

---

## Stable ordering

Where output ordering is not semantically meaningful, choose deterministic
ordering when practical.

This improves:

- scripting;
- testing;
- diffs.

---

## Stdin

Use stdin for:

- piped data;
- secrets where ecosystem conventions permit;
- large payloads.

Do not read stdin unexpectedly during non-interactive execution.

---

## Sensitive stdin

If accepting secrets from stdin, avoid echoing them.

Document whether trailing newline is stripped.

---

## File input

For file arguments:

- validate existence;
- validate type where relevant;
- bound size for hostile input;
- handle `-` as stdin only if documented.

Do not follow arbitrary symlinks in privileged contexts without considering
trust.

---

## File output

Avoid overwriting existing files silently.

Use:

- `--force`;
- explicit destination;
- atomic write where practical.

For sensitive output, use restrictive permissions.

---

## Atomic writes

For configuration or generated files, prefer:

```text
write temporary
    ->
fsync if required
    ->
rename
```

when partial files would be dangerous.

---

## Temporary files

Use secure platform primitives.

Do not create predictable temp filenames in shared directories.

---

## Configuration

Configuration should be discoverable and explicit.

Typical sources:

- flags;
- environment;
- config file;
- defaults.

Document precedence.

---

## Config file location

Follow platform/ecosystem conventions.

Avoid writing config into the repository working directory by default unless
the CLI is repository-scoped.

---

## Config format

Use a common format appropriate to the ecosystem.

Validate configuration before performing side effects.

Do not accept arbitrary executable configuration unless intentionally designed.

---

## Unknown config keys

Choose strict or permissive behavior deliberately.

Strict validation helps catch typos.

Forward compatibility may justify warnings rather than errors.

---

## Profiles/contexts

For CLIs managing multiple environments/accounts, use explicit profiles or
contexts.

Examples:

```text
dev
staging
prod
```

The active context should be inspectable.

Do not let a hidden stale context cause accidental production changes.

---

## Environment indication

For high-impact commands, show the target environment clearly.

Consider stronger prompts/guards for production.

Do not rely only on terminal color.

---

## Credentials

Credentials must not appear in:

- command history where avoidable;
- process listings;
- logs;
- error output;
- config committed to source.

Prefer:

- secure credential store;
- environment mechanisms where conventional;
- stdin;
- workload/device login flows;
- OS keychain.

---

## Command-line secrets

Avoid flags such as:

```text
--password secret
--token secret
```

when safer mechanisms exist, because process arguments may be observable.

If supported for compatibility, warn/document the risk.

---

## Login

For networked CLIs, login should establish credentials through an appropriate
standard flow.

Avoid implementing bespoke password handling when provider-standard auth is
available.

---

## Logout

Logout should remove or invalidate local credentials where possible.

Do not claim server-side revocation if logout only deletes local state.

---

## Token storage

If storing tokens locally:

- use OS-appropriate secure storage where practical;
- restrict file permissions if file-based;
- never print token value during normal status commands.

---

## Credential precedence

Define whether explicit flags, environment, profiles, or workload identity take
precedence.

Ambiguous credential selection is dangerous.

---

## Authentication diagnostics

Errors should distinguish:

- no credentials;
- expired credentials;
- invalid credentials;
- authorization denial.

Do not dump credential material when debugging auth.

---

## Network behavior

For networked CLIs, define:

- endpoint;
- TLS;
- timeout;
- retry;
- proxy;
- user agent;
- cancellation.

---

## TLS

Verify certificates by default.

If insecure TLS is allowed for development, make it explicit and visibly
dangerous.

Prefer:

```text
--insecure-skip-tls-verify
```

over vague flags such as:

```text
--unsafe
```

Document scope.

---

## Endpoint override

Allow custom endpoints only when needed.

Validate scheme and format.

An endpoint override can become an SSRF or credential-exfiltration risk in
privileged automation.

---

## Proxies

Follow standard proxy environment conventions where appropriate.

Do not invent incompatible proxy variables without need.

---

## Timeouts

Network operations must have finite timeouts.

Expose overrides for legitimate long-running operations.

Do not default to infinite waits.

---

## Cancellation

Ctrl-C should cancel active work promptly and safely.

For multi-step operations:

- stop starting new work;
- cancel in-flight work where safe;
- preserve useful error state.

Do not ignore SIGINT.

---

## Signals

Handle relevant signals predictably.

Do not install global signal behavior unrelated to CLI lifecycle.

---

## Retries

Retry transient failures only.

Use bounded backoff/jitter.

Do not retry:

- invalid input;
- authentication failure;
- authorization denial;
- destructive writes with unknown outcome

without idempotency or reconciliation.

---

## Unknown outcome

A timeout during a remote write may mean the operation succeeded.

Represent this explicitly.

Provide follow-up guidance or reconciliation.

Do not blindly repeat the command.

---

## Idempotency

Where remote APIs support idempotency keys, use them for retryable mutation.

Expose request/operation IDs when useful for troubleshooting.

---

## Pagination

For list commands:

- stream or paginate;
- bound memory;
- support `--limit` where useful.

Do not silently fetch millions of records into memory.

---

## Large output

For large results:

- support streaming;
- avoid rendering giant tables;
- allow structured output;
- allow filters.

---

## Filtering

Prefer server-side filtering where supported.

Do not make a CLI download all production data just to filter locally.

---

## Concurrency

If the CLI performs parallel operations:

- bound concurrency;
- preserve deterministic result reporting;
- handle partial failures.

Do not spawn unbounded workers based on input size.

---

## Batch commands

For batch operations, support:

- per-item status;
- summary;
- deterministic exit behavior.

If side effects occur, consider `--dry-run`.

---

## Destructive commands

Use explicit verbs such as:

```text
delete
destroy
revoke
purge
```

Avoid destructive behavior under vague verbs such as `apply` unless domain
semantics clearly establish it.

---

## Force flags

A `--force` flag should have narrow semantics.

Do not use one generic force flag to bypass:

- validation;
- authorization;
- TLS;
- confirmations;
- policy

all at once.

---

## Security boundaries

The CLI should not confuse local user intent with remote authorization.

A command being executable locally does not mean the remote operation is
authorized.

Remote systems must still enforce policy.

---

## Local privilege

Avoid requiring root/admin privileges unless necessary.

If elevated privilege is needed:

- scope it narrowly;
- explain why;
- separate privileged subcommands where useful.

Do not ask users to run the entire CLI with `sudo` for one filesystem action.

---

## Shell execution

If the CLI invokes subprocesses:

- avoid shell interpolation when direct exec is sufficient;
- validate arguments;
- propagate exit status;
- handle signals;
- avoid secret leakage.

Prefer argument arrays over string-built shell commands.

---

## Command injection

Never concatenate untrusted input into shell commands.

If a shell is genuinely required, escape according to the actual shell and
minimize untrusted interpolation.

---

## Environment inheritance

Subprocesses inherit environment by default in many runtimes.

Remove sensitive variables when child processes do not need them.

---

## PATH resolution

For security-sensitive CLIs, understand how subprocess executables are resolved.

Avoid accidentally executing attacker-controlled binaries earlier in `PATH`.

---

## Plugins

Plugin systems increase attack surface and compatibility burden.

Use only when real extension needs exist.

If supported:

- define discovery;
- trust model;
- versioning;
- isolation;
- signing/verification where justified.

Do not execute arbitrary plugins from writable current-directory locations by
default.

---

## Update mechanism

If the CLI self-updates, treat update as supply-chain-sensitive.

Prefer package-manager updates where appropriate.

Do not download and execute unsigned arbitrary binaries.

---

## Telemetry

Do not emit usage telemetry unexpectedly.

If telemetry exists:

- document it;
- minimize data;
- avoid command arguments that may contain secrets;
- provide control consistent with product policy.

---

## Crash reporting

Crash reports may include sensitive command arguments or environment data.

Redact aggressively.

Do not upload full environment dumps by default.

---

## Logging

CLIs usually need diagnostics more than persistent local logs.

If persistent logs exist:

- document location;
- bound retention;
- protect sensitive data.

Do not leave secrets in world-readable log files.

---

## Audit

For local-only CLIs, local audit may not be meaningful.

For privileged administrative CLIs, rely on the authoritative remote system
for durable audit where possible.

Do not present local history as authoritative proof of remote action.

---

## Repository awareness

Repository-scoped CLIs should discover project context carefully.

Examples:

- config file;
- VCS root;
- workspace metadata.

Do not traverse arbitrary parent directories indefinitely without clear rules.

---

## Current working directory

Define whether commands operate relative to:

- CWD;
- repository root;
- explicit `--directory`.

Avoid surprising path behavior.

---

## Symlinks

When manipulating filesystem content in security-sensitive contexts, account
for symlink traversal.

Do not assume lexical path normalization prevents filesystem escape.

---

## Cross-platform behavior

If supporting multiple operating systems, test:

- path handling;
- line endings;
- terminal behavior;
- executable discovery;
- permissions;
- signals where relevant.

Do not claim portability based only on compilation.

---

## Shell quoting

Documentation examples should use safe quoting.

Avoid examples that break on spaces or special characters.

---

## Locale

Machine-readable output should not depend on locale.

Human output may localize if the product supports it.

Do not emit locale-formatted numbers into JSON.

---

## Time and dates

For structured output, use unambiguous timestamps such as ISO 8601.

Clearly define timezone.

Do not emit ambiguous local timestamps in automation output.

---

## Determinism

Commands should produce stable results for the same inputs where the domain
allows it.

Avoid:

- random ordering;
- nondeterministic temp output;
- hidden timestamp fields in generated config

unless required.

---

## Reproducibility

For build/generation CLIs, record relevant:

- tool version;
- config;
- input revision;
- output identity.

---

## Generated files

Generated output should be reproducible where practical.

Provide:

```text
generate
verify-generated
```

or ecosystem equivalents when generation is part of normal development.

---

## Formatting

If the CLI emits source/configuration files, use canonical formatting.

Do not generate unstable whitespace that creates noisy diffs.

---

## Streaming input/output

For Unix-style tools, support pipes where natural.

Do not require temporary files for data that can stream safely.

---

## Backpressure

When streaming between stdin/network/stdout, avoid unbounded buffering.

---

## Broken pipe

Handle downstream pipe closure gracefully.

Example:

```bash
tool list | head
```

should not emit a scary stack trace if the downstream consumer exits early.

---

## Terminal width

Human table formatting may adapt to terminal width.

Structured output must not.

---

## Accessibility

Do not encode critical meaning only in:

- color;
- animation;
- emoji.

Plain text should remain usable.

---

## Help examples

Examples should be copy-pasteable and safe.

Do not put destructive production commands in the first quickstart without
clear warning.

---

## Default target

Avoid dangerous implicit defaults.

A CLI should not default to production if multiple environments exist unless
that is an established domain convention with strong justification.

---

## Configuration discovery

If searching for config in parent directories, define the stop condition.

Do not unexpectedly pick up unrelated config from a user's home or filesystem
root.

---

## Security-sensitive config

Validate ownership/permissions of credential files where platform conventions
support it.

Warn or reject obviously unsafe permissions when risk warrants.

---

## Offline mode

If offline operation is meaningful, support it explicitly.

Do not silently fall back to stale cached data when the command claims current
remote state.

---

## Caching

If caching remote data:

- bound it;
- version it;
- scope by account/profile;
- define freshness;
- protect sensitive values.

Do not allow one account's cached data to appear under another account.

---

## Cache invalidation

Provide clear behavior for:

- TTL;
- manual clear;
- config/account change.

---

## Config migration

If config format changes:

- migrate safely;
- preserve backup where useful;
- avoid destructive rewrite before validation.

---

## Backward compatibility

Treat these as compatibility-sensitive:

- command names;
- flags;
- defaults;
- exit codes;
- JSON field names;
- config format;
- credential locations;
- environment variables.

---

## Machine-readable compatibility

Human output can evolve more freely than structured output.

If scripts are expected to consume JSON, document stability expectations.

---

## Semantic versioning

Use semantic versioning where appropriate.

Breaking CLI automation contracts generally belong in major versions unless the
project's versioning policy says otherwise.

---

## Deprecation

For deprecated commands/flags:

- warn on stderr;
- provide replacement;
- give migration path;
- remove only according to versioning policy.

---

## Experimental commands

Mark experimental commands clearly.

Avoid presenting unstable features as part of the normal stable interface.

---

## Completion and aliases

Aliases should not create ambiguous behavior.

Keep canonical command names in docs and machine interfaces.

---

## Plugin compatibility

If plugins exist, version the plugin protocol.

Do not make plugins depend on private internal packages accidentally.

---

## Packaging

The CLI artifact should install cleanly.

Potential distribution formats:

- ecosystem package;
- standalone binary;
- archive;
- container image;
- OS package.

Use the simplest channels appropriate to users.

---

## Standalone binaries

If publishing binaries:

- name predictably;
- include version;
- publish checksums;
- sign where required;
- test on target platforms.

---

## Archives

Archives should contain only expected files.

Avoid nested unpredictable top-level directories unless conventional.

---

## Installer scripts

If offering `curl | sh` style installation, treat it as a high-trust path.

Prefer transparent, reviewable installers and authenticated artifacts.

Do not make it the only installation path if package-manager options exist.

---

## Package managers

Follow ecosystem conventions.

Do not maintain five package-manager channels unless users need them and the
project can sustain them.

---

## PATH installation

Document how users obtain the executable on `PATH`.

Avoid modifying shell profiles silently.

---

## Self-contained runtime

If the CLI is expected to be a single binary, do not require hidden runtime
downloads after installation unless documented.

---

## Supply chain

Compose with `software-supply-chain.md`.

At minimum consider:

- immutable release version;
- reproducible build;
- dependency pinning;
- SBOM;
- provenance;
- signatures/checksums;
- verified publishing.

A CLI often executes with user credentials and deserves strong artifact
integrity.

---

## Update trust

If the CLI downloads:

- plugins;
- schemas;
- policies;
- toolchains;
- binaries

verify integrity and source where required.

Do not treat HTTPS alone as complete artifact provenance.

---

## Testing strategy

A CLI requires tests at the interface boundary.

### Unit tests

Test:

- parsing helpers;
- config merge;
- validation;
- business logic;
- output formatting.

### CLI integration tests

Execute the built CLI as a process.

Verify:

- stdout;
- stderr;
- exit code;
- files;
- side effects.

### Golden tests

Useful for stable human output.

Review updates intentionally.

Do not use giant snapshots that hide meaningful changes.

### Structured-output tests

Validate JSON/NDJSON against expected schema.

### Packaging tests

Install the released/built artifact in a clean environment and execute it.

---

## Test the binary, not only functions

A CLI can fail through:

- entrypoint;
- packaging;
- argument parser;
- signal handling;
- stdout routing.

At least some tests should run the actual executable.

---

## Exit-code tests

Test exit codes explicitly.

Do not assume the framework maps exceptions correctly.

---

## Stdout/stderr tests

Test channel separation.

A regression that sends warnings to stdout can break automation even though the
human output still looks correct.

---

## Interactive tests

Where prompts matter, test:

- yes;
- no;
- EOF;
- non-TTY;
- `--yes`.

---

## Signal tests

For long-running commands, test Ctrl-C/cancellation where practical.

---

## Filesystem tests

Test:

- missing files;
- permission denied;
- existing output;
- symlinks;
- temp cleanup.

---

## Network failure tests

For networked CLIs, test:

- DNS failure;
- timeout;
- connection reset;
- TLS failure;
- 401;
- 403;
- 429;
- 5xx;
- malformed response.

---

## Retry tests

Verify:

- retry count;
- backoff policy;
- no retry on permanent errors;
- unknown-outcome handling.

---

## Fuzzing

Consider fuzzing:

- config parsers;
- file parsers;
- command argument parsing where custom;
- protocol decoders.

---

## Property-based testing

Useful properties may include:

- config round-trip;
- normalization idempotency;
- parser never crashes;
- `--output json` always emits valid JSON on success.

---

## Cross-platform tests

If multi-platform support is promised, run representative tests on those
platforms.

---

## Documentation tests

Where practical, execute shell snippets or examples.

Stale CLI docs are particularly harmful.

---

## Help snapshot

A lightweight help snapshot can catch accidental public command changes.

Use intentionally.

---

## Performance

Performance matters when commands operate on large repos/data.

Benchmark:

- startup time;
- scan throughput;
- memory;
- network round trips.

Do not optimize cold-start milliseconds if the command is dominated by a
30-second remote operation.

---

## Startup time

Developer CLIs benefit from fast startup.

Avoid:

- unnecessary network calls;
- loading huge config;
- eagerly importing heavy modules

before command dispatch.

---

## Lazy initialization

Initialize expensive dependencies only for commands that need them.

`tool --help` should not initialize cloud SDKs or connect to databases.

---

## Concurrency performance

Parallelism should be configurable or bounded.

Do not assume maximum CPU utilization is always desirable on developer
machines.

---

## Observability

For remote/high-value operations, expose request/operation IDs.

This helps correlate CLI errors with server logs.

Do not embed a full observability stack inside a simple local CLI.

---

## Diagnostics command

A `doctor`, `diagnose`, or equivalent command may be useful for complex tools.

It can check:

- config;
- credentials;
- endpoint reachability;
- dependency versions.

Do not include secrets in diagnostic bundles.

---

## Diagnostic bundles

If generating support bundles:

- redact;
- inventory included files;
- get explicit user action before upload.

---

## Crash behavior

Unexpected failures should:

- return nonzero;
- show concise user-facing message;
- optionally show detailed stack trace in debug mode.

Do not dump a raw stack trace on every normal user error.

---

## User errors versus internal errors

Distinguish mistakes such as:

```text
unknown flag
invalid path
missing required value
```

from internal bugs.

Use different messages and possibly exit codes.

---

## Error messages

Good errors should answer:

- what failed;
- why;
- what the user can do next.

Avoid vague:

```text
Something went wrong.
```

---

## Suggestions

For typos, command suggestions can help.

Do not auto-execute a guessed destructive command.

---

## Localization

Only localize if the project actually supports it.

Machine-readable output and exit codes should remain stable.

---

## Documentation

The repository should document:

- installation;
- command overview;
- configuration precedence;
- credential handling;
- output modes;
- exit codes;
- destructive command behavior;
- examples;
- upgrade/migration behavior.

---

## README

The README should quickly show:

```text
install
    ->
configure/authenticate
    ->
run common command
```

Do not make the quickstart require reading every advanced option.

---

## Man pages

Generate man pages where the audience/distribution benefits.

Keep them synchronized with command definitions.

---

## Shell completion generation

Generate completion from the same command metadata where possible.

Avoid hand-maintained drift.

---

## Architecture decisions

Use ADRs for consequential choices such as:

- config precedence;
- credential storage;
- machine-output stability;
- plugin architecture;
- self-update mechanism;
- destructive-action policy.

Do not create ADRs for every flag.

---

## Recommended repository shape

Follow ecosystem conventions first.

A generic CLI repository may resemble:

```text
.
├── AGENTS.md
├── README.md
├── src/
│   ├── cli/
│   ├── commands/
│   ├── config/
│   ├── output/
│   └── client/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── cli/
│   ├── packaging/
│   └── security/
├── docs/
│   ├── commands/
│   └── adr/
└── <ecosystem build/package files>
```

Only create directories that contain meaningful content.

---

## Verification interface

A CLI repository should expose obvious commands or equivalent native interfaces
for:

```text
check
test
test-cli
test-integration
test-packaging
build
package
verify
```

These names are illustrative.

Use the ecosystem's native tooling where appropriate.

---

## Acceptance criteria

A CLI is not complete because `tool --help` renders.

Demonstrate the applicable subset of the following.

### Command interface

- command tree is coherent;
- required arguments/flags validate;
- help is useful;
- unknown flags fail clearly;
- version output works.

### Automation contract

- exit codes are intentional;
- stdout/stderr are separated;
- machine-readable output is valid;
- non-interactive mode does not prompt unexpectedly;
- stable output schema is documented where applicable.

### Safety

- destructive actions require explicit intent;
- dry-run/plan works where applicable;
- production/high-impact targets are visible;
- `--force` semantics are narrow;
- unknown-outcome writes are not blindly retried.

### Security

- secrets are absent from logs/output;
- TLS verification is enabled by default;
- credentials are stored/loaded safely;
- command-line secret exposure is avoided where practical;
- subprocess arguments are injection-safe;
- local privilege is minimized.

### Reliability

- timeouts are finite;
- cancellation works;
- retries are bounded;
- batch partial failures are represented correctly.

### Packaging

- artifact builds;
- clean install works;
- installed artifact executes;
- target platform support is verified;
- version metadata is correct.

### Testing

- actual executable is exercised;
- exit codes are tested;
- stdout/stderr are tested;
- network failure paths are tested where relevant;
- interactive confirmation is tested where relevant.

---

## Optional composition

Common combinations:

```text
cli + software-supply-chain
```

For signed/verifiable binaries, provenance, checksums, and trusted publishing.

```text
cli + hostile-input
```

For scanners, parsers, archive tools, and CLIs that process attacker-controlled files.

```text
cli + high-assurance
```

For stronger compatibility, fuzzing, signal tests, fault injection, and release evidence.

```text
cli + zero-trust-service
```

For administrative or platform CLIs using workload/user identity and sensitive remote APIs.

```text
cli + security-tool
```

For scanners and security automation where exit codes, deterministic findings,
and machine-readable output are central.

```text
cli + ai-security
```

For AI-assisted CLIs that process untrusted model output or expose tool/action capabilities.

---

## Anti-patterns

Avoid:

- printing diagnostics into JSON stdout;
- always returning exit code `0`;
- prompts in CI with no override;
- `--force` that disables unrelated safety controls;
- secrets passed on argv by default;
- TLS verification disabled by default;
- infinite network timeouts;
- blind retries of side-effecting operations;
- destructive defaults;
- hidden production context;
- unstable JSON fields;
- scripts required to scrape tables;
- giant deeply nested command trees;
- network calls during `--help`;
- package that only works from repository checkout;
- global mutable configuration;
- surprise telemetry;
- plugins loaded from untrusted directories;
- running the entire CLI as root for one privileged action;
- direct shell interpolation of user input;
- machine output dependent on terminal width/color.

Do not mistake a friendly prompt for a safe CLI.

---

## Completion evidence

When this recipe is applied, the final report should state:

1. CLI purpose and primary users;
2. command tree;
3. argument/flag conventions;
4. stdout/stderr contract;
5. exit-code model;
6. machine-readable output formats;
7. config precedence;
8. authentication/credential model;
9. destructive-action safeguards;
10. timeout/retry/cancellation model;
11. subprocess/filesystem safety model;
12. packaging/install model;
13. compatibility/deprecation policy;
14. interactive versus non-interactive behavior;
15. CLI/integration tests executed;
16. packaging tests executed;
17. security/failure-path tests executed;
18. commands actually run;
19. observed results;
20. unverified assumptions and deliberate omissions.

Never claim a CLI is "automation-safe", "secure", "scriptable", or
"production-ready" merely because its happy-path command works.

Describe the exit-code, stream, credential, failure, packaging, and destructive
action behavior that was actually exercised.

---

## Guiding principle

A good CLI should be boring in the best possible way.

Humans should know what it is about to do.

Scripts should know exactly what happened.

Stdout should be usable.

Stderr should be useful.

Exit codes should mean something.

Credentials should stay hidden.

Dangerous actions should require intent.

Remote failures should terminate predictably.

And the same command should behave sensibly whether it is run by a human at a
terminal, by a CI job at 03:00, or inside a shell pipeline nobody will inspect
for six months.
