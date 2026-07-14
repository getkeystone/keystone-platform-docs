# keystone-engage

Governed conversational agent for regulated customer interaction.

## What it does

Engage handles customer-facing conversations in regulated environments where
every response must be governed, every decision auditable, and escalation to a
human must happen reliably when the situation requires it. A single customer
turn runs through a fixed, five-phase governed pipeline. Every phase records
what it did before the next phase runs.

## Who it is for

Engage is built for teams that operate customer interaction under a compliance
obligation — financial services, healthcare, insurance, and other regulated
lines of business. It fits the case where a support or advisory conversation is
customer-facing and consequential, where "the model said something we can't
explain" is not an acceptable outcome, and where a human has to be in the loop
the moment a situation crosses a severity threshold.

## Why governed conversation matters

In a regulated setting, a conversational agent is not judged only on the quality
of its answers. It is judged on whether every answer can be explained after the
fact, whether sensitive situations reached a human, and whether the system
stayed inside its cost and authorization bounds. Ungoverned agents fail on the
audit, not on the demo. Engage treats governance as the request path rather than
a wrapper around it: the escalation decision, the retrieval, the budget check,
and the audit entry are steps in the pipeline, not optional middleware.

## Phase pipeline

Each phase is a registered specialist with a function, a tempo classification,
and a cost profile. Tempo sets the latency budget and the dispatch priority for
the phase.

| Phase | Function   | Tempo    | Purpose                                    |
|-------|------------|----------|--------------------------------------------|
| 1     | Empathy    | fast     | Emotional tone assessment                  |
| 2     | Escalation | fast     | Severity-tier classification               |
| 3     | Engagement | medium   | Response composition via governed RAG      |
| 4     | Budget     | medium   | Session cost check against budget          |
| 5     | Monitoring | deferred | Observability and lifecycle signal emission |

Every phase writes an audit entry. The event bus emits task lifecycle events.
Short-circuit conditions halt the pipeline when escalation severity crosses a
threshold or the session budget is exhausted.

## Request flow

```
user turn
    |
    v
[Empathy]      -- fast, emotional tone assessment
    |
    v
[Escalation]   -- fast, severity-tier classification
    |               if tier crosses threshold: short-circuit to human review
    v
[Engagement]   -- medium, governed retrieval + response composition
    |               if retrieval below confidence threshold: fail closed
    v
[Budget]       -- medium, session cost check
    |               if budget exceeded: short-circuit
    v
[Monitoring]   -- deferred, telemetry + event-bus signals
    |
    v
response returned
```

## Governance features

- **Severity-tier human-in-the-loop escalation.** The Escalation phase assigns a
  severity tier. When the tier crosses the configured threshold, the pipeline
  short-circuits and routes the turn to human review before any response is
  returned. Tier thresholds are configurable.
- **Per-step hash-chained audit trail.** Every phase writes an append-only entry
  carrying `prev_hash` and `curr_hash`, plus tokens, cost, model, and latency.
  Breaking the chain requires forging every subsequent hash, so tampering is
  detectable on replay.
- **Budget enforcement and short-circuit.** The Budget phase checks the
  session-rolling cost against the configured budget and short-circuits the
  pipeline when it is crossed. The short-circuit is a recorded event, not a
  silent failure.
- **Fail-closed retrieval.** When governed retrieval does not meet the
  confidence threshold, the Engagement phase refuses to compose a response
  rather than guessing, and the refusal is audited.

## Sample audit trace shape

The field names below are the audit schema. Concrete values — costs, token
counts, severity tier, budget — are shown as placeholders here; a live run
populates them. No real runtime values or registered agent identifiers are
published.

```
entry_<n>  agent=<empathy>     tempo=fast    cost_cents=<n>
           input_tokens=<n> output_tokens=<n> latency_ms=<n>
           prev_hash=<hash> curr_hash=<hash>

entry_<n>  agent=<escalation>  tempo=fast    cost_cents=<n>
           severity_tier=<tier> hitl_required=<bool>
           prev_hash=<hash> curr_hash=<hash>

entry_<n>  agent=<budget>      tempo=medium  cost_cents=<n>
           budget_remaining_cents=<n> short_circuit=<bool>
           prev_hash=<hash> curr_hash=<hash>
```

Each entry chains to the previous one through `prev_hash`/`curr_hash`, so the
sequence of phases for a turn is itself verifiable, not just the individual
entries.

## Published evaluation

Engage ships with a sealed baseline evaluated by the standalone
[keystone-verify](verify.md) harness. The published examples below follow the
`keystone-{component}/{type}-v{n}` naming convention.

| Baseline                  | Result                                             |
|---------------------------|----------------------------------------------------|
| keystone-engage/agent-v1  | 100/100 (regression 70, architecture 25, edge 5)   |
| keystone-core/agent-v0    | Sealed failing: 66 cases, 4 real bugs surfaced     |
| keystone-core/agent-v1    | 186 cases, 558 executions, 0 failures              |

The `agent-v0` run is a failing baseline preserved next to the passing one: it
found four real bugs, which is the evidence that the methodology works. See the
[evaluation model →](../evaluation/index.md) for how baselines are sealed, and
the raw artifacts in
[keystone-ledger →](https://github.com/getkeystone/keystone-ledger).

## Substrate dependencies

Engage consumes the full substrate: the agents registry, the task state machine,
the hash-chained audit ledger, the event bus, MCP-exposed tools with
agent-scoped authorization, and cost-aware dispatch. It is workload logic on top
of shared primitives, not a standalone application.
See the [substrate model →](../architecture/substrate.md).

## Source code

The keystone-engage source code is proprietary. Read-only technical review of
the private repository is available on request for interview-depth evaluation.
[Request access →](../access.md)
