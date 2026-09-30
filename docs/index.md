# Keystone Applied Intelligence

**Keystone Applied Intelligence** is the independent engineering and R&D
practice of Arnaldo Sepulveda. This site documents Keystone's implemented
runtime mechanisms and applied workloads. **Governed Execution** is the
broader runtime-governance research program that this engineering work feeds
into. **Runtime Validity (Track A)**
([GitHub](https://github.com/getkeystone/runtime-validity)) is the bounded
reference implementation within Governed Execution; it does not validate the
broader research program. This distinction is used consistently throughout
these docs.

Keystone is an independent engineering and R&D practice for building and
evaluating governed AI systems in regulated and high-consequence environments.

The work focuses on the runtime layer between model capability and production
consequence: authorization, task state, evidence, evaluation, auditability,
observability, and fail-closed behavior.

Keystone's engineering work explores a shared-substrate design direction, with
authorization, execution state, audit evidence, and evaluation designed into
the architecture rather than added only at the application boundary.

The platform is intended for workloads where responses and actions must operate
under explicit authority, evidence, and review constraints: customer
interaction, controlled retrieval and advisory workflows, and the evaluation
infrastructure used to test those claims.

## Operational intelligence before implementation

Keystone's engineering work addresses how Applied AI mechanisms are built and
evaluated once an intervention is justified. A related part of the broader
Keystone portfolio asks the upstream question: what should actually change in
the operation, and does that change require AI?

[Support Operations Intelligence](https://github.com/arnaldosepulveda/support-operations-intelligence)
is the separate public research and portfolio project where that operational
analysis is being developed. It is maintained by Arnaldo Sepulveda and is not
an implemented Keystone platform component.

[Operational Intelligence →](operational-intelligence.md)

## The three extensions

**keystone-engage.** Governed conversational agent for regulated customer
interaction. The served path is a single governed agent; a multi-agent
coordinator with tempo heterogeneity and per-agent audit trails is implemented
and available behind a config flag, not the default served route. Severity-tier
human-in-the-loop routing is served. Dispatch schemas carry budget/tempo/cost
fields, but real cost-based model selection and budget enforcement are not yet
exercised end to end.

**keystone-counsel.** Authorization-first retrieval for legal and financial
advisory content. Classification-aware vector-similarity ACL filtering enforced
at the database layer. Fail-closed under insufficient authorization or
insufficient confidence.

**keystone-verify.** Standalone HTTP evaluation harness for compatible endpoints.
Produces structured evaluation results and run metadata using profiles and
deterministic assertions. Passing and failing evaluation lineage is retained
separately in keystone-ledger.
[View on GitHub →](https://github.com/getkeystone/keystone-verify)

## Shared-substrate mechanisms

The platform uses a shared-substrate design. The current public
implementations contain related runtime mechanisms at varying degrees of
maturity; they should not be read as one fully composed runtime service.
Verify is a standalone evaluation harness that evaluates HTTP endpoints and
produces evaluation artifacts; it does not run as a workload on this
substrate.

Six currently implemented mechanisms exist across the public repositories, at
varying degrees of maturity and composition:

- **Agents registry.** The registry implementation carries agent identity, role,
  tempo classification, and cost-profile metadata that can be referenced by
  runtime records.
- **Task state machine.** Explicit ownership, validated transitions, and a
  takeover protocol are implemented for task recovery. Not every lifecycle
  mechanism is exercised by the normal served path today.
- **Hash-chained audit mechanisms.** Engage and Counsel use unkeyed SHA-256
  hash chaining; keystone-gov uses keyed HMAC-SHA256 per record. These are
  implementation-specific audit mechanisms rather than evidence of one composed
  shared audit service.
- **Optional NATS JetStream integration.** Can carry task lifecycle events for
  observers in the optional multi-agent/event path. It is off the normal
  request path and off by default.
- **Query-time authorization.** Access control enforced before retrieval. Engage
  uses a corpus-scope ACL (role to allowed corpora, fail-closed); Counsel enforces
  a classification and client-isolation `WHERE` clause in the retrieval query.
  Records excluded by those predicates are not returned through that query. An
  MCP server entry point is scaffolded but not wired to the served path.
- **Cost/budget fields.** Dispatch and audit schemas carry budget, tempo, and
  cost fields, but real cost-based model selection and budget enforcement are
  not yet exercised end to end.

These are six implemented runtime mechanisms. They are not the six candidate
substrate dimensions of the research model (see
[Engineering platform and research model](#engineering-platform-and-research-model)),
and should not be read as such.

## Relationship among the implementations

The architecture is organized around a shared-substrate design, while the
current public workload implementations remain separately composed.

Across the public workload implementations, related runtime governance
patterns include authorization, audit, task-state mechanisms, and resource
metadata, but those mechanisms are not uniformly present or composed across
workloads. The current public repositories should not be interpreted as
demonstrating a single composed runtime shared by Engage and Counsel. Verify is
external evaluation infrastructure: it can evaluate compatible HTTP endpoints
and produce evaluation artifacts; it is not itself a workload running on this
substrate.

The architecture is designed to reduce dependence on any particular inference
backend, model provider, or orchestration layer. How well the governance
semantics survive replacement of those components remains an empirical research
question rather than a demonstrated portability property.

The design rationale traces to a specific origin: the operational rigor the
contact-center industry already built for compliance and governance informs the
design for LLM-based systems. See [contact-center heritage →](design/heritage.md).

## Engineering platform and research model

Keystone distinguishes the implemented runtime from the research model being
developed around it.

The current Keystone repositories contain concrete mechanisms such as agent
registration, task state, authorization, event coordination, audit evidence,
evaluation, and model dispatch, attributed to their workload above.

The working research architecture, *Governed Execution as a Runtime Contract*,
proposes identity, task state, tempo, cost, currency, and fidelity as candidate
substrate dimensions of governed execution.

These dimensions are a candidate representation of governance-relevant runtime
state, not six implemented services and not a claim of completeness.

Current research asks which runtime changes make a prior governance decision
stale, what should trigger revalidation, and what evidence should allow an
external reviewer to reconstruct why an action proceeded, was held, denied, or
escalated.

Cross-framework portability, generalized action binding, and completeness of the
candidate dimensions remain hypotheses to test.

## What is public vs private

| Surface                                                      | Status  |
|---------------------------------------------------------------|---------|
| Platform documentation (this site)                             | Public  |
| keystone-ledger, published evaluation ledger                   | Public  |
| keystone-verify, evaluation framework                          | Public  |
| keystone-engage, governed conversational agent                 | Public  |
| keystone-gov, governed RAG reference                            | Public  |
| keystone-counsel, authorization-first retrieval                 | Public  |
| runtime-validity, Governed Execution research track (Track A)   | Public  |
| Deployment configuration and infrastructure detail              | Private |

The architecture, evaluation outcomes, and design rationale are public: the
public implementation repositories and documentation/evaluation surfaces are
inspectable. Deployment configuration and internal infrastructure detail
remain private.
[What is public vs private →](access.md)

## Published evaluation

The evaluation ledger is public:
[keystone-ledger →](https://github.com/getkeystone/keystone-ledger).
The examples below use the naming convention `keystone-{component}/{type}-v{n}`.
The `keystone-core/*` identifiers name historical evaluations of a private
repository; their results are retained and published in keystone-ledger.

| Baseline                       | Result                                          | Status         |
|--------------------------------|-------------------------------------------------|----------------|
| keystone-core/retrieval-v1     | P@1=0.75, MRR=0.79, 8/8 adversarial ACL blocked, fail-closed 5/6 (83%) | mixed          |
| keystone-core/agent-v0         | 186 cases; 9 failing cases, 4 root-cause defects | retained failing |
| keystone-core/agent-v1         | A retained internal evaluation of keystone-core/agent-v1 used 186 cases across 12 categories and 558 executions. At keystone-gov commit ff66368 the run produced 144 strict passes, 9 strict failures, and 33 characterization cases; the failures traced to four implementation defects, which were fixed. At commit 6ac192a the same cases produced 153 strict passes and 33 characterization cases. Results apply only to the evaluated commits, configurations, and cases, and are not independent validation. | passing        |
| keystone-engage/agent-v1       | 100/100 (regression 70, architecture 25, edge 5)| passing        |

The retained failing run is preserved alongside the passing baseline. In this
evaluation lineage, the failing run shows that the evaluation configuration
detected specific implementation defects that were subsequently corrected and
re-tested. The retrieval-v1 fail-closed figure (83%, 5 of 6) is from the
retained 2026-04-11 baseline; its single miss (FC-005) has a demo-grade
domain-scope guard merged 2026-05-17, with re-verification not yet recorded.

## Learn more

- [System architecture →](architecture/index.md)
- [Operational Intelligence →](operational-intelligence.md)
- [Extensions overview →](extensions/index.md)
- [Evaluation methodology →](evaluation/index.md)
- [What is public vs private →](access.md)
- [About Keystone →](about.md)
