# Keystone Applied Intelligence

Keystone is a platform for building governed AI systems that regulated
enterprises can deploy. It is three extensions running on one shared substrate,
with governance, authorization, and evaluation designed in from the first commit
rather than bolted on afterward.

The platform is built for environments where every response must be governed,
every decision auditable, and every retrieval authorized before it happens:
customer interaction, legal and financial advisory, and the evaluation that
proves the other two behave.

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

**keystone-verify.** Standalone evaluation harness, endpoint-agnostic. Runs
against any HTTP endpoint over a structured profile and assertion vocabulary.
It writes structured, reproducible runs; failing runs are preserved alongside
passing runs, and the ledger seals and versions them.
[View on GitHub →](https://github.com/getkeystone/keystone-verify)

## The shared substrate

All three extensions sit on one shared substrate. Six named surfaces do the
governance work so the extensions do not have to reimplement it:

- **Agents registry.** A first-class registry of agents, each carrying identity,
  role, tempo classification, and a cost profile. Every audit entry references a
  registered agent.
- **Task state machine.** Explicit ownership, validated transitions, and a
  takeover protocol so long-running tasks stay recoverable rather than silently
  orphaned.
- **Hash-chained audit ledger.** Append-only, SHA-256 hash-chained entries (the
  shared substrate is unkeyed SHA-256; keystone-gov uses a keyed HMAC per record).
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

## Why this is a platform, not a set of demos

The extensions plug into the substrate. They do not each rebuild it.

Engage, counsel, and verify share one agents registry, one task state machine,
one audit chain, one event bus, one authorization model, and one cost model.
Adding a capability is a population change against existing contracts, not a new
stack. Swapping an inference backend or migrating a data plane is a substrate
change, not an extension change. That shared foundation is what separates a
platform from three disconnected proofs of concept.

The design rationale traces to a specific origin: the operational rigor the
contact-center industry already built for compliance and governance, rebuilt for
the LLM substrate. See [contact-center heritage →](design/heritage.md).

## What is public vs private

| Surface                                           | Status  |
|---------------------------------------------------|---------|
| Platform documentation (this site)                | Public  |
| keystone-ledger — published evaluation ledger     | Public  |
| keystone-verify — evaluation framework            | Public  |
| keystone-engage — governed conversational agent   | Public  |
| keystone-gov — governed RAG reference             | Public  |
| keystone-counsel — authorization-first retrieval  | Public  |
| Deployment configuration and infrastructure detail| Private |

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
| keystone-core/retrieval-v1     | P@1=0.75, MRR=0.79, 8/8 adversarial ACL blocked, fail-closed 83% | passing        |
| keystone-core/agent-v0         | 66 cases, 4 real bugs surfaced                  | sealed failing |
| keystone-core/agent-v1         | 186 cases, 558 executions, 0 failures           | passing        |
| keystone-engage/agent-v1       | 100/100 (regression 70, architecture 25, edge 5)| passing        |

The sealed failing run is kept on purpose. A failing baseline preserved next to
the passing one that replaced it is evidence that the evaluation methodology
finds real bugs.

## Learn more

- [System architecture →](architecture/index.md)
- [Extensions overview →](extensions/index.md)
- [Evaluation methodology →](evaluation/index.md)
- [What is public vs private →](access.md)
- [About Keystone →](about.md)
