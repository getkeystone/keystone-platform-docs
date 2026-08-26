# Keystone Applied Intelligence

**Keystone Applied Intelligence** is the independent engineering and R&D
practice of Arnaldo Sepulveda. Keystone's engineering platform (documented on
this site) contains implemented shared runtime capabilities and applied
workloads. **Governed Execution** is the broader runtime-governance research
program and reference platform that this engineering work feeds into. **Track
A Runtime Validity** ([GitHub](https://github.com/getkeystone/track-a-runtime-validity))
is a bounded public research implementation within Governed Execution; it does
not validate the broader platform. This distinction is used consistently
throughout these docs.

Keystone is an independent engineering and R&D practice for building and
evaluating governed AI systems in regulated and high-consequence environments.

The work focuses on the runtime layer between model capability and production
consequence: authorization, task state, evidence, evaluation, auditability,
observability, and fail-closed behavior.

Three extensions run on shared runtime services, with authorization, execution
state, audit evidence, and evaluation designed into the architecture rather than
added only at the application boundary.

The platform is intended for workloads where responses and actions must operate
under explicit authority, evidence, and review constraints: customer
interaction, controlled retrieval and advisory workflows, and the evaluation
infrastructure used to test those claims.

## The three extensions

**keystone-engage.** Governed conversational agent for regulated customer
interaction. The served path is a single governed agent; a multi-agent
coordinator with tempo heterogeneity and per-agent audit trails is implemented
and available behind a config flag, not the default served route. Severity-tier
human-in-the-loop routing and a cost-aware dispatch interface are served.

**keystone-counsel.** Authorization-first retrieval for legal and financial
advisory content. Classification-aware vector-similarity ACL filtering enforced
at the database layer. Fail-closed under insufficient authorization or
insufficient confidence.

**keystone-verify.** Standalone, endpoint-agnostic evaluation harness. Runs
against HTTP endpoints using structured profiles and assertions. Preserves
sealed failing runs alongside passing runs so remediation can be evaluated
against the failure it replaced.
[View on GitHub →](https://github.com/getkeystone/keystone-verify)

## The shared runtime

All three extensions use the same runtime services rather than rebuilding
authorization, state, evidence, coordination, and resource controls
independently.

Six currently implemented services provide that common execution foundation:

- **Agents registry.** A first-class registry of agents, each carrying identity,
  role, tempo classification, and a cost profile. Every audit entry references a
  registered agent.
- **Task state machine.** Explicit ownership, validated transitions, and a
  takeover protocol so long-running tasks stay recoverable rather than silently
  orphaned.
- **Hash-chained audit ledger.** Append-only, SHA-256 hash-chained entries (the
  shared runtime is unkeyed SHA-256; keystone-gov uses a keyed HMAC per record).
  Each entry chains to the previous one, and `verify_chain` walks the full ledger
  on replay, so a break in the chain is detectable.
- **NATS JetStream event bus.** Carries task lifecycle events for observers.
  This is the observability path, distinct from the request path.
- **Query-time authorization.** Access control enforced before retrieval. Engage
  uses a corpus-scope ACL (role to allowed corpora, fail-closed); Counsel enforces
  a classification and client-isolation `WHERE` clause in the retrieval query, so
  unauthorized rows never return. An MCP server entry point is scaffolded but not
  wired to the served path.
- **Cost-aware dispatch.** Every dispatch carries a budget and a tempo target;
  every audit entry records tokens, model, and cost. Dispatch can short-circuit
  when a budget is exhausted, and the short-circuit is a recorded event.

These are six implemented runtime services. They are not the six candidate
substrate dimensions of the research model (see
[Engineering platform and research model](#engineering-platform-and-research-model)),
and should not be read as such.

## Why this is a platform, not a set of demos

The extensions plug into the shared runtime. They do not each rebuild it.

Engage, Counsel, and Verify share common identity, task-state, audit, event,
authorization, and resource services. Adding a workload can therefore reuse
existing runtime contracts rather than rebuilding those mechanisms from scratch.

The architecture is designed to reduce dependence on any particular inference
backend, model provider, or orchestration layer. How well the governance
semantics survive replacement of those components remains an empirical research
question rather than a demonstrated portability property.

The design rationale traces to a specific origin: the operational rigor the
contact-center industry already built for compliance and governance, rebuilt for
LLM-based systems. See [contact-center heritage →](design/heritage.md).

## Engineering platform and research model

Keystone distinguishes the implemented runtime from the research model being
developed around it.

The implemented runtime contains concrete services such as agent registration,
task state, authorization, event coordination, audit evidence, evaluation, and
model dispatch.

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
| track-a-runtime-validity, Governed Execution research track     | Public  |
| Deployment configuration and infrastructure detail              | Private |

The architecture, evaluation outcomes, and design rationale are public, and so is
the source: all five repositories are public. Deployment configuration and
infrastructure detail remain private.
[What is public vs private →](access.md)

## Published evaluation

The evaluation ledger is public:
[keystone-ledger →](https://github.com/getkeystone/keystone-ledger).
The examples below use the naming convention `keystone-{component}/{type}-v{n}`.

| Baseline                       | Result                                          | Status         |
|--------------------------------|-------------------------------------------------|----------------|
| keystone-core/retrieval-v1     | P@1=0.75, MRR=0.79, 8/8 adversarial ACL blocked, fail-closed 5/6 (83%) | mixed          |
| keystone-core/agent-v0         | 186 cases; 9 failing cases, 4 root-cause defects | sealed failing |
| keystone-core/agent-v1         | 186 cases, 558 executions, 0 failures           | passing        |
| keystone-engage/agent-v1       | 100/100 (regression 70, architecture 25, edge 5)| passing        |

The sealed failing run is kept on purpose. Preserving the failing baseline next
to the passing baseline that replaced it provides evidence that the evaluation
process can surface implementation defects. The retrieval-v1 fail-closed figure
(83%, 5 of 6) is from the sealed 2026-04-11 baseline; its single miss (FC-005)
has a demo-grade domain-scope guard merged, with re-verification not yet sealed.

## Learn more

- [System architecture →](architecture/index.md)
- [Extensions overview →](extensions/index.md)
- [Evaluation methodology →](evaluation/index.md)
- [What is public vs private →](access.md)
- [About Keystone →](about.md)
