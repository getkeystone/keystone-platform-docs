# The substrate model

This page describes a **research abstraction**, not a concrete platform service
or a single shared runtime. The substrate is a candidate semantic representation
of governance-relevant runtime state about a unit of work that should remain
meaningful even when models, frameworks, and providers change. Current Keystone
mechanisms are engineering examples that inform this abstraction. They do not
form one composed runtime and do not define the abstraction itself.

## Candidate substrate dimensions

The working research architecture proposes six candidate dimensions. They are
research hypotheses, not an established or complete ontology:

- **Identity**: who or what is acting, and under whose authority.
- **Task state**: where the unit of work is in its lifecycle.
- **Tempo**: the time horizon the work is expected to run under.
- **Cost**: what the work may consume, and what it consumed.
- **Currency**: whether the original justification still legitimately authorizes
  the intended consequence at the point of execution.
- **Fidelity**: the bar the result must clear for its context, and the evidence
  that it did.

The research question is whether these facts can support decisions that remain
meaningful across changes in execution technology. Their completeness and
portability have not been demonstrated.

## Relationship to Governed Execution

The dimensions above describe candidate governance-relevant state. They are not
the planes of the controlling *Governed Execution as a Runtime Contract*
architecture, which separates:

- a **control plane** for authority, policy, admissibility, placement, budget,
  and release;
- an **execution plane** for models, retrieval, tools, delegation, and
  workflows;
- an **evidence plane** for decisions, authorizations, actions, evaluations,
  failures, and outcomes; and
- a separate **action boundary** that determines whether output may create an
  external consequence.

Orchestration determines how work proceeds. Governance determines whether the
intended consequence remains justified to proceed.

A governance contract would decide which substrate facts matter to an intended
consequence, which changes are material, what must be revalidated, what evidence
must be recorded, and whether the result is **PROCEED**, **HOLD**, **DENY**, or
**ESCALATE**. The material-change taxonomy and the conditions under which a
prior justification becomes stale remain research questions.

## Current engineering examples

The public Keystone repositories contain separate mechanisms relevant to the
abstraction. This section maps those mechanisms without claiming common runtime
ownership, universal coverage, or deployment as a shared stack.

| Research concern | Current engineering example | Boundary |
|---|---|---|
| Identity and authorization | Retrieval constraints in Gov, Counsel, and Engage | The mechanisms differ by workload; there is no demonstrated central agent registry shared by every repository |
| Task state and tempo | Engage state transitions, heartbeat, stuck detection, takeover, and rescheduling | Some mechanisms are not exercised by the default served path; distributed lease or fencing guarantees are not established |
| Cost and budget | Engage dispatch and audit schemas carry budget, tempo, cost, model, and token fields | Real cost-based selection and complete budget enforcement are not demonstrated end to end |
| Evidence | Workload-specific audit records and retained evaluation artifacts | Audit implementations and coverage differ; internal evaluation is not independent validation |
| Currency | Runtime Validity revalidates a controlled process-local authority change | This is a bounded research implementation, not external revocation or production authorization infrastructure |
| Fidelity | Gov evidence thresholds and HHEM scoring; Verify profile assertions; retained Ledger results | Results apply to evaluated configurations and commits, not general production suitability or portability |

### Authorization mechanisms

Authorization is applied differently in each implemented workload:

- **keystone-gov** constrains retrieval using role, domain, and jurisdiction
  information, alongside its served Governed RAG checks.
- **keystone-counsel** applies classification and client-relationship constraints
  in retrieval. Client isolation is implemented and regression-tested, while
  the current published corpus is global and does not establish a production
  multi-client deployment.
- **keystone-engage** implements corpus-scope authorization. Its
  `authorize_tool_call` function is not wired into the served path, and its MCP
  support is scaffolded rather than a served authorization control.

These examples support the principle that authorization should precede access to
protected evidence. They do not establish one universal authorization service.

### Task-state mechanisms

Engage implements validated state transitions and includes heartbeat,
stuck-detection, takeover, and rescheduling mechanisms. The default served path
uses a single `EngageOrchestrator`; the multi-agent Coordinator is optional and
configuration-dependent. Not every task-state mechanism is exercised by that
default path. The implementation therefore does not establish distributed
lease, fencing, or general recovery guarantees.

### Audit and evaluation evidence

Audit mechanisms must be described separately:

- **keystone-engage** and **keystone-counsel** use unkeyed SHA-256 hash chains.
  A chain can make changes detectable relative to a trusted reference state,
  but an actor able to rewrite an unanchored store may also recompute subsequent
  hashes.
- **keystone-gov** uses a keyed HMAC-SHA256 value per record, with different and
  limited field coverage. That integrity check does not establish semantic
  correctness, authorization validity, independent witnessing, or immutable
  history.
- **keystone-ledger** retains internal evaluation artifacts and lineage, with
  recorded checksums and Git history rather than a general cryptographic
  sealing mechanism.

The Ledger results are evidence about the evaluated commits and configurations.
They do not establish generality, portability, production suitability, or
third-party validation. In particular, Counsel has no dedicated evaluation
baseline in the Ledger. Verify produces structured results and run metadata for
compatible HTTP endpoints through profiles, but it does not own historical
retention and should not be attributed as the producer of historical
`keystone-core` artifacts.

### Optional event integration

NATS JetStream integration belongs to Engage's optional multi-agent and event
path. It is off the default served path. The public deployment repositories do
not demonstrate NATS as a deployed service shared across Keystone workloads, so
it is not evidence of a common platform event bus.

### Cost and budget metadata

Engage schemas expose budget, tempo, priority, model, token, and cost fields.
Those interfaces make cost and budget available for future governance controls,
but real cost-based model selection and complete budget enforcement are not
demonstrated end to end. `LocalDispatcher` currently records effectively zero
cost, so the fields should not be read as evidence that active cost governance
is operating across the platform.

## Runtime Validity

Runtime Validity is the bounded Track A reference implementation within the
Governed Execution research program. Its current demo performs a controlled,
process-local authority change and retains transition evidence. Under full
revalidation, the implemented result can change from **PROCEED/MATCH** to
**HOLD/MISMATCH**; with `revalidation_mode="none"`, the state is
**NOT_EVALUATED**, not justified.

This result is useful evidence for a narrow research mechanism. It does not
demonstrate the complete substrate, authentic external revocation, production
authentication or authorization, durable persistence, independent witness
evidence, structural evidence binding, cryptographic evidence integrity, or
real external-consequence enforcement. **HOLD** is an implementation design
choice, not a universal conclusion of the research architecture.

## Infrastructure choices are not substrate requirements

PostgreSQL with pgvector, local model serving, NATS JetStream, and
OpenTelemetry appear in particular Keystone implementations. They are
engineering choices, not requirements of the substrate abstraction. Their
presence in one repository does not show that every workload uses the same
stack or that governance semantics will survive replacement of those
technologies. That portability is a hypothesis to evaluate.

## Related

- [System architecture](index.md)
- [Extensions: keystone-engage](../extensions/engage.md)
- [Extensions: keystone-counsel](../extensions/counsel.md)
- [Evaluation methodology](../evaluation/index.md)
- [Design heritage](../design/heritage.md)
- [What is public vs private](../access.md)
