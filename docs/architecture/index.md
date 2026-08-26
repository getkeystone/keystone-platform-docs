# System architecture

Keystone is one platform organized as three implementation layers: **extensions**,
a **shared runtime**, and **infrastructure**. Each layer depends only on the layer
beneath it, and only through a contract. That single rule (extensions call
runtime contracts, the runtime talks to infrastructure) is what makes Keystone a
platform rather than three applications that happen to share a database.

## The layered model

```
┌────────────────────────────────────────────────────────────────┐
│  Extensions                                                      │
│    keystone-engage      keystone-counsel      keystone-verify    │
└────────────────────────────────────────────────────────────────┘
                          │  call runtime contracts
                          ▼
┌────────────────────────────────────────────────────────────────┐
│  Shared runtime                                                  │
│    agents registry          task state machine (9 states)        │
│    hash-chained audit        NATS JetStream event bus            │
│    query-time authz          cost-aware dispatch                 │
└────────────────────────────────────────────────────────────────┘
                          │  contracts talk to infrastructure
                          ▼
┌────────────────────────────────────────────────────────────────┐
│  Infrastructure                                                  │
│    PostgreSQL 16 + pgvector           NATS JetStream            │
│    local LLM serving (Ollama / vLLM)                            │
│    OpenTelemetry + self-hosted trace backend                    │
└────────────────────────────────────────────────────────────────┘
```

Three properties hold this model together:

- **Extensions never touch infrastructure directly.** An extension does not open
  a database connection, publish to the event bus, or call an inference backend
  on its own. It calls a runtime contract, and the shared runtime mediates the
  infrastructure. This keeps governance, authorization, and audit on the request
  path instead of scattered through application code.

- **Extensions do not rebuild shared capabilities.** The agents registry, task
  state machine, audit ledger, event bus, query-time authorization, and
  cost-aware dispatch live in the shared runtime once. Extensions consume them; they do not each carry a
  private roster, a private audit format, or a private budget mechanism. Adding
  an extension is a matter of consuming existing contracts, not re-implementing
  the platform.

- **Infrastructure is replaceable in principle.** Swapping an inference backend or
  migrating a data plane is intended to be a change at the runtime layer rather
  than in the extensions, which never name the backend directly. How completely
  the governance semantics survive such a swap is a research question, not a
  demonstrated portability property.

For the six implemented runtime services and how they compose, see the
[substrate model](substrate.md), which documents both the research abstraction
and its current instantiation. For what each extension does on top of them, see
the [extensions overview](../extensions/index.md).

## Conceptual layers

The implementation diagram above is one view. Conceptually, the platform spans
several layers, some implemented and some research architecture. They are listed
here so implemented mechanisms are not confused with proposed ones:

1. **Capability layer**: models, prompts, tools, memory, planning. External to
   Keystone.
2. **Orchestration layer**: routing, queues, scheduling, delegation, retries,
   recovery.
3. **Shared runtime implementation** (implemented): registry, task state,
   authorization, event coordination, audit, dispatch. The services documented on
   this page.
4. **Candidate runtime substrate model** (research): identity, task state,
   tempo, cost, currency, and fidelity as candidate dimensions of governed
   execution. A candidate representation, not an established or complete set. See
   the [substrate model](substrate.md).
5. **Governance contract** (research): material-change rules, revalidation
   conditions, and consequence policy (PROCEED, HOLD, DENY, ESCALATE).
6. **Action boundary** (research architecture): the progression from generation
   to recommendation to evaluation to authorization to commitment. Generalized
   action binding is a proposed architecture, not a demonstrated guarantee; the
   served path today executes retrieval and generation and does not bind external
   consequences.
7. **Evidence and evaluation** (implemented): hash-chained audit records,
   evaluation artifacts, preserved failure lineage, and the ability to reconstruct
   why an action was handled as it was.

## Design principles

Seven principles carry most of the platform's identity. They are structural
choices, not runtime configuration. They hold whether or not any given model
behaves as expected.

1. **Structural governance over heuristic guardrails.** The primary controls are
   structural: authorization checks that fail closed and run before generation,
   severity-tier human review on high-risk interactions, and integrity-checked
   audit records. Model-based filters exist as defense in depth, but they are
   never the sole control. A prompt that talks its way past a model has not talked
   its way past a database predicate.

2. **Authorization before retrieval, at the database layer.** The authorization
   predicate is part of the query the database executes, not a filter applied to
   results after they return. Unauthorized rows never leave the database, so a bug
   in an orchestrator, a prompt injection, or a hallucinated citation cannot leak
   content the caller was not permitted to see, because the content was never retrieved.

3. **Fail-closed by default.** Insufficient authorization refuses; it does not
   partially answer. Low-confidence retrieval refuses; it does not guess. The safe
   state on ambiguity is refusal with an audited reason, not a best-effort
   response.

4. **Hash-chained audit trail.** Every retrieval, authorization decision, and
   escalation writes an append-only, SHA-256 hash-chained audit entry (the shared
   runtime is unkeyed SHA-256; keystone-gov uses a keyed HMAC per record). Each
   entry carries the hash of the entry before it, and `verify_chain` walks the
   full ledger on replay, so an edit or deletion breaks the chain and is
   detectable. The audit trail is evidence, not a log that can be quietly rewritten.

5. **Sealed failing runs preserved alongside passing runs.** Evaluation runs are
   sealed as durable artifacts in the ledger, and a failing run is not deleted
   when a passing run replaces it. Both are kept. As a published example, the sealed failing
   baseline `keystone-core/agent-v0` (186 cases; 9 failing cases from 4 root-cause
   defects) sits next to the passing `keystone-core/agent-v1` (186 cases, 558
   executions, 0 failures).
   The failing run is the evidence that the methodology finds real bugs. See the
   [evaluation methodology](../evaluation/index.md).

6. **Cost as a first-class signal.** Every dispatch call carries a budget, a tempo
   target, and a priority; every audit entry records tokens, model, per-call cost,
   and session-rolling cost. When a budget is exhausted, dispatch short-circuits,
   and the short-circuit is a recorded event, not a silent failure. Cost is
   measured and governed on the same path as correctness and authorization.

7. **Local-first deployment.** Every runtime component runs on hardware the
   operator controls: the database, the event bus, the trace backend, and the
   model serving. There is no required dependency on an external inference API.
   Regulated operators can run the platform inside their own boundary, which is
   the point.

## Related

- [The substrate model](substrate.md): the research abstraction and its current instantiation
- [Extensions overview](../extensions/index.md): what runs on top of the shared runtime
- [Evaluation methodology](../evaluation/index.md): how runs are sealed and preserved
- [What is public vs private](../access.md): repository access and boundaries
