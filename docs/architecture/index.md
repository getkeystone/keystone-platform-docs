# System architecture

Keystone is one platform organized as three layers: **extensions**, a **shared
substrate**, and **infrastructure**. Each layer depends only on the layer beneath
it, and only through a contract. That single rule — extensions call substrate
contracts, substrate contracts talk to infrastructure — is what makes Keystone a
platform rather than three applications that happen to share a database.

## The layered model

```
┌────────────────────────────────────────────────────────────────┐
│  Extensions                                                      │
│    keystone-engage      keystone-counsel      keystone-verify    │
└────────────────────────────────────────────────────────────────┘
                          │  call substrate contracts
                          ▼
┌────────────────────────────────────────────────────────────────┐
│  Shared substrate                                                │
│    agents registry          task state machine (9 states)        │
│    hash-chained audit        NATS JetStream event bus            │
│    MCP tool exposure         cost-aware dispatch                 │
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
  on its own. It calls a substrate contract, and the substrate mediates the
  infrastructure. This keeps governance, authorization, and audit on the request
  path instead of scattered through application code.

- **Extensions do not rebuild shared capabilities.** The agents registry, task
  state machine, audit ledger, event bus, tool exposure, and cost-aware dispatch
  live in the substrate once. Extensions consume them; they do not each carry a
  private roster, a private audit format, or a private budget mechanism. Adding
  an extension is a matter of consuming existing contracts, not re-implementing
  the platform.

- **Infrastructure is replaceable.** Swapping an inference backend or migrating a
  data plane is a substrate change, not an extension change. The extensions above
  the substrate are unaffected because they never named the backend in the first
  place.

For the six surfaces the substrate exposes and how they compose, see the
[substrate model](substrate.md). For what each extension does on top of them, see
the [extensions overview](../extensions/index.md).

## Design principles

Seven principles carry most of the platform's identity. They are structural
choices, not runtime configuration — they hold whether or not any given model
behaves as expected.

1. **Structural governance over heuristic guardrails.** The primary controls are
   structural: authorization checks that fail closed, tool scopes enforced before
   dispatch, human-approval gates on irreversible actions, and tamper-evident
   audit records. Model-based filters exist as defense in depth, but they are
   never the sole control. A prompt that talks its way past a model has not talked
   its way past a database predicate.

2. **Authorization before retrieval, at the database layer.** The authorization
   predicate is part of the query the database executes, not a filter applied to
   results after they return. Unauthorized rows never leave the database, so a bug
   in an orchestrator, a prompt injection, or a hallucinated citation cannot leak
   content the caller was not permitted to see — the content was never retrieved.

3. **Fail-closed by default.** Insufficient authorization refuses; it does not
   partially answer. Low-confidence retrieval refuses; it does not guess. The safe
   state on ambiguity is refusal with an audited reason, not a best-effort
   response.

4. **Tamper-evident, hash-chained audit trail.** Every retrieval, tool call,
   authorization decision, and escalation writes an append-only, HMAC-verified
   audit entry. Each entry carries the hash of the entry before it, so any edit or
   deletion breaks the chain and is detectable on replay. The audit trail is
   evidence, not a log that can be quietly rewritten.

5. **Sealed failing runs preserved alongside passing runs.** Evaluation runs are
   sealed as immutable artifacts, and a failing run is not deleted when a passing
   run replaces it. Both are kept. As a published example, the sealed failing
   baseline `keystone-core/agent-v0` (66 cases, 4 real bugs surfaced) sits next to
   the passing `keystone-core/agent-v1` (186 cases, 558 executions, 0 failures).
   The failing run is the evidence that the methodology finds real bugs. See the
   [evaluation methodology](../evaluation/index.md).

6. **Cost as a first-class signal.** Every dispatch call carries a budget, a tempo
   target, and a priority; every audit entry records tokens, model, per-call cost,
   and session-rolling cost. When a budget is exhausted, dispatch short-circuits —
   and the short-circuit is a recorded event, not a silent failure. Cost is
   measured and governed on the same path as correctness and authorization.

7. **Local-first deployment.** Every substrate component runs on hardware the
   operator controls: the database, the event bus, the trace backend, and the
   model serving. There is no required dependency on an external inference API.
   Regulated operators can run the platform inside their own boundary, which is
   the point.

## Related

- [The substrate model](substrate.md) — the six surfaces extensions consume
- [Extensions overview](../extensions/index.md) — what runs on top of the substrate
- [Evaluation methodology](../evaluation/index.md) — how runs are sealed and preserved
- [What is public vs private](../access.md) — repository access and boundaries
