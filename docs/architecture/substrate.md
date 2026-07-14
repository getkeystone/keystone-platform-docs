# The substrate model

The substrate is not a monolith. It is six named surfaces that the three
extensions consume. Extensions call substrate contracts; substrate contracts
talk to infrastructure. Replacing an inference backend or migrating a data
plane is a substrate change, not an extension change.

Each surface is described below three ways: what it is, why it exists, and what
extension behavior it enables. The point of reading them together is to see
that governance, cost, and multi-agent operation are not features bolted onto
the extensions — they are properties of the layer the extensions stand on.

## The six surfaces at a glance

| Surface | Responsibility | Primary contract |
|---|---|---|
| Agents registry | Identity, role, tempo, and cost profile for every agent | Register and resolve agents by id |
| Task state machine | Ownership, lifecycle, and recovery of long-running work | Validated transitions, heartbeat, takeover |
| Hash-chained audit ledger | Tamper-evident record of every governed action | Append entry, verify chain on replay |
| Event bus | Fan-out of task lifecycle events to observers | Publish and subscribe, off the request path |
| MCP tool exposure | Scope-checked tool access for models and agents | Authorize on user + agent identity, then dispatch |
| Cost-aware dispatch | Budget, tempo, and priority on every model call | Dispatch with budget; short-circuit as a recorded event |

## 1. Agents registry

**What it is.** A first-class registry of agents. Each row carries identity, a
role, a tempo classification (`fast`, `medium`, `slow`, `deferred`), and a cost
profile. Every audit entry references an agent by id.

**Why it exists.** Multi-agent operation is a property of the substrate, not of
any one extension. New agents register into the shared registry; an extension
does not maintain its own private roster. Because identity, tempo, and cost are
recorded centrally, the dispatch layer and the audit ledger can reason about
who acted, at what time horizon, and at what cost — without trusting the
extension to report it.

**What it enables.** An extension composes a pipeline out of registered agents.
The Engage conversational flow, for example, runs distinct functions —
Empathy, Escalation, Engagement, Budget, and Monitoring — each a registered
agent with its own tempo and cost profile. Adding a cooperating agent at a new
tempo is a population change against an existing schema, not a migration.

## 2. Task state machine

**What it is.** A task lifecycle expressed as nine states with validated
transitions, heartbeat tracking, and an explicit takeover protocol. Every task
has an explicit owner.

**Why it exists.** Long-running agent work must survive agent restarts, crashes,
and stalls. A task whose owner goes silent should become recoverable, not
silently orphaned. The takeover protocol is heartbeat-driven: a healthy agent
can claim ownership of a task whose previous owner stopped reporting, and the
transition is a validated, recorded event.

**What it enables.** Safe long-running tasks under restart. A stalled agent
times out and its task becomes claimable rather than hanging. Governance
outcomes also live here: a human-in-the-loop escalation moves the task into a
state the operator console can claim, and a budget-exceeded outcome moves it
into a state an operator can review and approve for continuation.

## 3. Hash-chained audit ledger

**What it is.** An append-only ledger of HMAC-verified entries. Each entry
carries a `prev_hash` and a `curr_hash`. Every retrieval, tool call,
authorization decision, and escalation writes an entry. Two implementations
exist: an in-memory / file-backed chain and a PostgreSQL-backed chain.

**Why it exists.** Regulated deployments need an audit record that a reviewer
can trust months later. Because each entry commits to the hash of the one
before it, tampering with any entry breaks the chain and is detectable on
replay — altering the past requires forging every subsequent hash. The audit
trail is therefore evidence, not just logging.

**What it enables.** End-to-end replay of a governed session. A fail-closed
refusal in the Counsel retrieval path is recorded as a first-class entry with
zero content leaked, so a reviewer can confirm that an unauthorized request was
denied and that nothing reached application code. The same discipline is what
lets a sealed evaluation demonstrate structural controls — for example, the
published `keystone-core/retrieval-v1` baseline records all eight adversarial
ACL probes blocked. An illustrative entry, with placeholders for runtime
values, looks like:

```
entry:
  agent:            <agent-id>
  action:           retrieval
  authz:            allow
  chunks_leaked:    0
  input_tokens:     <n>
  output_tokens:    <n>
  model:            <model>
  cost_cents:       <cost>
  session_cost:     <rolling-cost>
  prev_hash:        <hash>
  curr_hash:        <hash>
```

## 4. Event bus

**What it is.** An asynchronous event bus that carries task lifecycle events —
`created`, `claimed`, `heartbeat`, `completed`, `failed` — and fans them out to
observers. The current implementation is built on NATS JetStream. This is the
observability path, not the request path; the request path is direct dispatch
through the coordinator.

**Why it exists.** Observers need to know that work is happening without sitting
on the synchronous request path. The request path is direct and returns a
value; the event bus is fan-out and returns nothing. An audit entry is written
on the request path; an event on the bus signals that the entry now exists.
Separating the two keeps the request path fast and lets observers subscribe
freely.

**What it enables.** Live observability. The operator console stays current by
subscribing to lifecycle events rather than polling. A Monitoring function can
emit signals to the bus and to the tracing pipeline without participating in
the request path. New observers — additional consoles, external monitors,
future replicas — attach by subscribing, with no change to the extensions
producing the events.

## 5. MCP tool exposure

**What it is.** A Model Context Protocol server entry point. Tool authorization
takes both user identity and agent identity as input. Different agents carry
different scopes.

**Why it exists.** Tool access is a structural control, not a model behavior.
The tool contract enforces scope before dispatch, so a model that requests a
tool it lacks scope for is refused at the substrate — not by the model, and not
by a prompt. A prompt injection that convinces a model to call an out-of-scope
tool still fails, because the scope check does not consult the model.

**What it enables.** Agent-scoped tool use. Within a single extension,
different agents hold different tool scopes, so a lower-privilege phase cannot
invoke a higher-privilege action. Tool-backed flows — such as a governed
payment-arrangement interaction in Engage — run through the same scope check
before any tool implementation is reached, and each invocation writes an audit
entry.

## 6. Cost-aware dispatch

**What it is.** Every dispatch call carries a budget, a tempo target, and a
priority. Every audit entry records input tokens, output tokens, the model
used, cost in cents, and session-rolling cost.

**Why it exists.** Cost is a first-class operational signal, not an
afterthought discovered on an invoice. Because budget travels with the dispatch
call and rolling cost is recorded on every entry, dispatch can short-circuit
when a session's budget is exhausted — and the short-circuit is a recorded
event, not a silent failure. Tempo travels the same way, so the dispatch layer
can honor a call's time horizon rather than treating every model call
identically.

**What it enables.** Budget enforcement as a defined control. A Budget function
checks session-rolling cost and halts the pipeline when it is crossed, moving
the task into a state an operator can review. Tempo targets let `fast` per-turn
work and `deferred` batch signaling coexist under one dispatch contract. And
because cost is recorded alongside every action, evaluation can report cost next
to accuracy rather than treating them as separate concerns.

## Why it is a substrate, not a library

The agents registry, task state machine, tempo dimension, and cost fields exist
before they are fully populated. The schema is designed for multiple
cooperating agents at different tempos on day one, even when a first release
runs one primary coordinator across a small number of specialist functions.
Adding cooperating agents is a population change, not a schema migration.

A library would hand each extension a set of helpers and let each extension
decide how to hold identity, lifecycle, audit, and cost. The substrate does the
opposite: it owns those surfaces, and the extensions consume the contracts. That
is why replacing an inference backend or migrating a data plane is a substrate
change with a stable extension-facing contract, and why governance and cost are
uniform across every extension rather than reimplemented per extension.

## Infrastructure

The substrate runs on local-first infrastructure:

- **PostgreSQL 16 with pgvector** — corpus, embeddings, task state, and the
  PostgreSQL-backed audit chain.
- **Local LLM serving (Ollama / vLLM)** — inference on hardware the operator
  controls; no dependency on a hosted model provider for core operation.
- **NATS JetStream** — the event bus for task lifecycle fan-out.
- **OpenTelemetry with a self-hosted trace backend** — traces and per-agent
  latency, kept in-house.

Internal node identifiers, network topology, and addresses are not published.

## Related

- [System architecture →](index.md)
- [Extensions: keystone-engage →](../extensions/engage.md)
- [Extensions: keystone-counsel →](../extensions/counsel.md)
- [Evaluation methodology →](../evaluation/index.md)
- [Design heritage →](../design/heritage.md)
- [What is public vs private →](../access.md)
