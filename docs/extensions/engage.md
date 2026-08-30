# keystone-engage

## What it does

Keystone Engage is a governed conversational-agent reference implementation for
regulated customer interaction. The normal served `/engage` path uses one
`EngageOrchestrator` to perform corpus-scope authorization, retrieval,
confidence handling, local generation, severity classification, and audit
recording.

The repository also contains optional multi-agent, task-state, event, budget,
and tool-authorization mechanisms. Their presence in code is not evidence that
each mechanism is exercised by the default served path.

## Default served path

The standard FastAPI application and the default configuration of the alternate
application create an `EngageOrchestrator`. The orchestrator:

- resolves a session and caller role;
- authorizes retrieval against the role's allowed corpus scopes;
- retrieves evidence and handles insufficient confidence;
- generates a response with local inference;
- classifies severity and marks defined escalations; and
- records request and response events in an audit chain.

This is the default behavior described by the public API. Human escalation is a
structured response outcome; human presence alone should not be read as proof
of meaningful oversight, correct authority, or a completed external handoff.

## Optional multi-agent Coordinator

When `KEYSTONE_MULTI_AGENT` is enabled in the alternate application,
`/engage` uses the `Coordinator` instead. That optional path composes empathy,
escalation, engagement, budget, and monitoring functions around dispatch.
Budget and monitoring logic run as Coordinator methods; they are not separate
standalone agent implementations.

The Coordinator exists in code and has unit tests, but it is not the default
served route and is not covered by the published Engage evaluation. Claims
about its behavior must therefore remain separate from claims about the normal
served path.

## Retrieval authorization

Engage implements corpus-scope authorization: a caller role maps to allowed
corpora, and retrieval is limited to those scopes. This differs from Counsel's
role, classification, and client-relationship model and from Gov's role,
domain, and jurisdiction constraints.

The repository also defines and unit-tests `authorize_tool_call`, which accepts
an agent identity. It is not called by a served request path. The MCP directory
contains a stdio server and placeholder catalog tool, but MCP is scaffolded
rather than a served tool-authorization boundary.

## Task state

The task-state model defines validated transitions across created, claimed,
in-progress, stuck, rescheduled, completed, verified, and failure states.
Heartbeat, stuck detection, takeover, and rescheduling mechanisms are present,
with unit coverage for state transitions.

Not all of these mechanisms are exercised by the normal served path. The
implementation does not demonstrate distributed lease or fencing guarantees,
and the public deployment repositories do not establish a distributed recovery
service.

## Optional event integration

NATS JetStream integration belongs to the optional multi-agent and lifecycle
event path. The default served path does not require it. Integration tests
require a separately running NATS service, and the public deployment
repositories do not establish NATS as a deployed service shared across Keystone
workloads.

OpenTelemetry instrumentation is implemented independently of that deployment
claim and records application spans and selected agent, model, token, latency,
cost, and budget attributes when values are supplied.

## Cost and budget boundary

Dispatch, task, audit, and telemetry schemas include fields for budget, tempo,
model, tokens, cost, and session rolling cost. The optional Coordinator contains
an in-memory session-cost check and tests that exercise its short-circuit.

These mechanisms do not demonstrate real cost-based model selection or complete
budget enforcement end to end. Local dispatch currently reports effectively
zero inference cost, and the default served path does not run the Coordinator's
budget phase. Engage evaluation metadata also records that cost fields were not
auditable when the ledger was not local to the evaluation host.

Schema fields and unit-tested optional logic should therefore be treated as
interfaces and bounded implementation mechanisms, not platform-wide active cost
governance.

## Audit boundary

Engage uses an unkeyed SHA-256 hash chain. Audit entries include previous and
current hash values, while the PostgreSQL backend also stores selected substrate
metadata. The exact entries produced depend on the served path.

A chain can support detection of changes relative to trusted prior state. An
actor able to rewrite an unanchored store may potentially recompute later
hashes. The audit mechanism does not provide immutable history, independent
witnessing, semantic correctness, or proof that an authorization or escalation
decision was justified.

## Evaluation status

Engage has its own retained internal evaluation lineage in
[keystone-ledger](https://github.com/getkeystone/keystone-ledger):

| Retained artifact | Result at the evaluated commit |
|---|---|
| `keystone-engage/agent-v0` | 96 of 100 cases passed |
| `keystone-engage/agent-v1` | 100 of 100 cases passed |

These results are separate from the historical `keystone-core/agent-v0` and
`keystone-core/agent-v1` artifacts, which evaluated a different system under
test. The Engage v1 result applies to its evaluated commit, cases, configuration,
and served path. It does not cover the optional Coordinator and does not
establish independent validation, production suitability, or general behavior
under untested inputs.

## Relationship to the broader platform

Engage is a separately composed implementation within the Keystone engineering
platform. Its mechanisms are relevant engineering examples for the broader
[system architecture](../architecture/index.md) and
[substrate research abstraction](../architecture/substrate.md). They do not
establish that Engage consumes one shared platform registry, audit service,
event service, authorization service, or cost-aware dispatcher.

## Source code

The implementation is public at
[github.com/getkeystone/keystone-engage](https://github.com/getkeystone/keystone-engage).
