
                         AGENTS.md
                 ┌─────────────────────┐
                 │ Engineering policy  │
                 │                     │
                 │ How should we think?│
                 └──────────┬──────────┘
                            │
               ┌────────────┴────────────┐
               │                         │
               ▼                         ▼
        ┌─────────────┐           ┌─────────────┐
        │ SCAFFOLD.md │           │ ELEVATE.md  │
        │             │           │             │
        │ Greenfield  │           │ Brownfield  │
        │             │           │             │
        │ What should │           │ What is     │
        │ exist?      │           │ missing?    │
        └──────┬──────┘           └──────┬──────┘
               │                         │
               └────────────┬────────────┘
                            ▼
                     Working repository
                            │
                            ▼
                    Executable evidence


| Recipe                      | What it establishes                                                                                                                 | Why it belongs                                                               |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `secure-service.md`         | Production API/service: config, health, telemetry, auth boundaries, graceful shutdown, containerization, CI, tests                  | Your generic “boring production service” baseline                            |
| `platform-control-plane.md` | Git → hermetic build → immutable artifact → signing → GitOps → progressive delivery                                                 | The Brazil/Apollo recipe you already have; strong DevSecOps flagship         |
| `software-supply-chain.md`  | SBOM, provenance, signing, verification, dependency closure, artifact promotion, admission policy                                   | Very aligned with your background and reusable across almost everything      |
| `zero-trust-service.md`     | Workload identity, explicit trust boundaries, least privilege, authorization, mTLS/policy where justified                           | Makes “zero trust” operational rather than marketing                         |
| `security-tool.md`          | CLI/service that scans or analyzes things safely: hostile-input handling, deterministic output, SARIF/JSON, exit semantics, fuzzing | Extremely reusable for the kind of tooling security engineers actually write |
| `ai-service.md`             | LLM-backed service with model abstraction, prompt/config separation, structured outputs, telemetry, evals, cost/time limits         | Base AI application recipe                                                   |
| `agentic-system.md`         | Tool-using agent with capability boundaries, approval gates, sandboxing, auditability, budgets and termination conditions           | Probably your most distinctive AI-security recipe                            |
| `rag-system.md`             | Ingestion → indexing → retrieval → generation with provenance, ACL propagation, poisoning defenses and retrieval evals              | RAG has enough unique security failure modes to justify its own recipe       |
| `mcp-tool-server.md`        | MCP/tool interface with schemas, authentication, authorization, side-effect classification, audit logging                           | Very relevant as agent ecosystems become tool-centric                        |
| `ai-evaluation.md`          | Dataset/eval harness for correctness, security, regressions, prompt injection, tool misuse, reliability and cost                    | Important because “tests passed” is inadequate for probabilistic systems     |
| `model-serving.md`          | Self-hosted model inference with model provenance, isolation, resource controls, observability and safe rollout                     | Bridges AI security and platform engineering                                 |
| `secure-data-pipeline.md`   | Batch/stream processing with lineage, schemas, validation, IAM, secrets, replay/idempotency                                         | Useful intersection of platform, data and security                           |
| `kubernetes-workload.md`    | Secure workload conventions without pretending every project requires Kubernetes                                                    | A focused deployment recipe instead of contaminating generic Scaffold        |
| `operator-controller.md`    | Reconciliation loop, idempotency, CRDs, failure recovery, status semantics, RBAC and integration testing                            | Specialized enough to deserve its own architecture                           |
| `library-sdk.md`            | Stable API, semver, compatibility testing, packaging, minimal deps, examples and release automation                                 | Keeps Scaffold from treating everything as a deployable service              |
| `cli.md`                    | Predictable exit codes, stdout/stderr semantics, config precedence, shell completion, packaging, hostile-input behavior             | Another common shape with very different requirements from services          |


| Recipe                      | What it is for                                           | Value                               |
| --------------------------- | -------------------------------------------------------- | ----------------------------------- |
| `platform-architecture.md`  | Generic platform/system-of-systems architecture          | **Foundational**                    |
| `control-plane.md`          | Generic control-plane/data-plane architecture            | **Foundational**                    |
| `developer-platform.md`     | Internal developer platform / paved roads                | **Very high**                       |
| `identity-platform.md`      | Human/workload identity, federation, authorization       | **Very high**                       |
| `api-platform.md`           | API gateway, service exposure, contracts, traffic policy | **Very high**                       |
| `eventing-platform.md`      | Messaging, events, queues, schemas, replay               | **Very high**                       |
| `observability-platform.md` | Logs, metrics, traces, telemetry pipelines               | **High**                            |
| `data-platform.md`          | Enterprise/shared data platform                          | **High**                            |
| `ai-platform.md`            | Shared model/RAG/agent/eval infrastructure               | **Very high for your domain**       |
| `protocol-design.md`        | Designing durable wire/API protocols                     | **Very high for architecture work** |
